# Multi-Tenant Risk Assessment — Interview Delivery Script

## Format notes: what makes a script "Staff-level" rather than "Senior-level"

Before the script itself, four things distinguish how a Staff/Principal candidate *delivers* a design from how a Senior candidate delivers the same content — the design docs already cover the *what*; this covers the *how you say it out loud*.

**Proactive disclosure, not reactive answering.** A Senior candidate answers the question asked. A Staff candidate surfaces the hard parts of the problem before being asked about them — "before I draw anything, I want to flag two things this prompt is quietly testing for" is a sentence a Senior candidate rarely says unprompted. The script below is built around specific moments to say this.

**Numbers before diagrams.** Every section that has a number in it (scale, latency, retention) gets said out loud *before* the corresponding box gets drawn, never after. Saying "so given ~5,800 events/sec, that's why I'm reaching for a stream instead of synchronous calls" lands very differently from drawing a stream first and justifying it when asked.

**Name the tradeoff, don't just state the choice.** "I'm using Postgres for Findings" is a Senior sentence. "I'm using Postgres for Findings, not the NoSQL store everything else uses, because the read pattern here needs multi-attribute filtering that key-value doesn't do well" is the Staff version of the same sentence. Every architectural noun in this script should be followed by a "because."

**Time-box out loud.** Say "I want to spend most of our remaining time on the duplicate/ordering problem since that's clearly the point of this prompt" — this signals judgment about where the value is, and it also gives the interviewer a chance to redirect you if they wanted something else, which reads as collaborative rather than presumptuous.

Don't memorize this word for word — internalize the beats and the "because" habit, and let your own phrasing carry it. A script recited flatly reads worse than rougher language delivered with real command of the material.

---

## The Script

### Opening (0:00–0:30)

"Before I start drawing, let me make sure I have the requirements right, and I'll flag a couple of things up front that I think this prompt is testing for beyond the obvious checks."

### 1. Clarify requirements (0:30–3:00)

"So functionally: we're ingesting observations from identity providers and security systems — role assignments, auth methods, sign-in activity, risk signals — multi-tenant, and running them against a small set of deterministic checks. When a check fails we open a finding, when newer evidence shows it passing we resolve it. Admins need to list risky users, see the evidence behind a finding, and check how fresh the underlying data is. That's the functional side, and I don't think I need to ask much more there.

What I do want to call out explicitly — because it's easy to read past it — is that the prompt states two non-functional requirements as if they were minor details: this has to work correctly across multiple tenants without data leaking between them, and it has to handle duplicate and delayed observations correctly. I'm going to treat both of those as first-class design constraints, not edge cases I bolt on at the end, and I'll come back to the second one in depth once the high-level architecture is up."

*[Expect: the interviewer may just nod and let you continue, or may ask "what does correctness under delay mean to you specifically" — if so, give the one-line preview: "an observation can arrive after a newer one already changed our understanding of the user, and we need to not let it regress that," then say you'll go deep on it later.]*

### 2. Capacity estimation (3:00–7:00)

"Let me size this before drawing anything, since it drives a few choices downstream. I'll assume 10,000 tenants, averaging 5,000 users each with a long tail — so around 50 million users total. Observations are event-driven, fired on change rather than polled, so I'll estimate roughly 10 per user per day across all the observation types combined. That's about 500 million observations a day, which averages to roughly 5,800 a second — but it won't be flat; it'll spike around business-hours sign-in waves across time zones.

That number is why I'm reaching for an async, stream-backed ingestion path rather than processing observations synchronously in the request path — at this rate, and with the bursty shape, a queue in front of processing is what keeps ingestion latency flat regardless of downstream load."

### 3. API design (7:00–11:00)

"On the API side, ingestion is a single endpoint: `POST /v1/tenants/{tenantId}/observations`, batch-capable, body is a list of `{source, sourceEventId, userId, observationType, eventTime, payload}`. Two fields I want to call out specifically: `sourceEventId` is the provider's own idempotency key — without it we'd be stuck deduping on content hashing, which is weaker — and `eventTime` is mandatory, because the entire delayed-observation story I'll get to later depends on having a real timestamp for when the fact was true, not just when we received it. This returns `202 Accepted` since it's async.

On the read side, admins need three things the prompt calls out explicitly: `GET /v1/tenants/{tenantId}/users?risky=true&checkId=&severity=&cursor=` for the list, `GET .../users/{userId}/findings` and `GET .../findings/{findingId}` for evidence, and `GET .../users/{userId}/freshness` returning the last event time and ingestion time per attribute plus a staleness flag. I'm designing the freshness endpoint per-attribute rather than per-user, because different observation types can go stale independently — MFA status might be fresh while sign-in activity hasn't synced in hours."

### 4. Data model (11:00–16:00)

"This is the part I want to slow down on, because getting the layering right here is what makes the rest of the design fall out naturally instead of needing special-casing later. Three layers, kept deliberately distinct.

First, the Observation itself — immutable, append-only, one row per event as it arrived: `observation_id, tenant_id, user_id, source, source_event_id, observation_type, event_time, ingested_at, payload`. I'm keeping `event_time` and `ingested_at` as two separate fields on purpose — one is when the fact was true, the other is when we found out about it, and conflating them is exactly what breaks delayed-observation handling later.

Second, User Current State — this is the materialized view the rule engine actually reads from, one row per tenant and user, but versioned *per attribute*, not per record. So `is_privileged`, `mfa_registered`, `last_sign_in_at` are each their own little object with a value, an event_time, and the observation_id that set it. And `open_risk_signals` is a set keyed by `risk_id` rather than a single scalar, because resolving one signal shouldn't touch the others. That per-attribute versioning, rather than per-row, is the single most important modeling decision in this whole design — it's what lets one stale field not block an update to a fresher one, which is the mechanism the whole duplicate/delay story rests on.

Third, Finding — unique on tenant, user, and check id, with status, severity, timestamps, and an evidence snapshot. I'm snapshotting the evidence at evaluation time rather than keeping a live pointer back to current state, so a finding's evidence stays accurate to why it was raised even after the user's state moves on."

*[Draw the three-layer separation as you say this, if a whiteboard is available — this is the moment to have something on the board.]*

### 5. High-level architecture (16:00–21:00)

"Let me put the flow up. An identity provider posts an observation, it hits the Ingestion API, which validates the tenant and does a conditional write to the Observation Store keyed on `source_event_id` — that's where exact duplicate retries die, before they ever reach processing. It's published to a stream partitioned by `tenant_id` plus `user_id`, so one consumer processes a given user's events in the order they arrived.

That consumer is one box, deliberately — I'm not splitting 'apply the update' and 'evaluate the rules' into two services. The handler does three things in one function call per message: a conditional write to User Current State that only applies if the incoming event_time is newer than what's stored for that specific attribute; if that write actually took effect, it looks up which checks depend on the changed attribute through a small in-memory map and evaluates just those; and it upserts the resulting finding. All synchronous, same process, because these checks are cheap local boolean logic against a row already in hand — there's no reason to pay for a network hop between 'update' and 'evaluate' here. I'd only split that into two services if check logic later needed to call out to something external and I wanted to isolate that latency from ingestion throughput.

Admin reads go through a tenant-scoped API hitting Findings for the list and evidence, and User Current State for freshness."

### 6. Deep dive — duplicates and ordering (21:00–30:00, the section to spend the most time in)

"This is the part of the prompt I think is actually being tested, so I want to go deep here rather than skim it.

Duplicates are handled at the edge: the `source_event_id` uniqueness constraint on the Observation Store write means a redelivered webhook never reaches the consumer a second time. And even if it did — say a consumer crashes and redelivers from the stream — the whole update-and-evaluate sequence is a pure function of the observation and current state, so replaying it produces the identical result. That's what lets me use at-least-once delivery without needing exactly-once stream semantics, which is operationally a lot simpler to run.

The harder problem is ordering. Stream partitioning by user guarantees *arrival* order, not *event-time* order — a slow connector can deliver an old observation after a newer one already landed and changed our picture of the user. The per-attribute event-time comparison handles this directly: if the incoming event_time is older than what's stored for that attribute, the write is rejected, current state is untouched, but the observation is still durably logged to the audit trail, and I bump a late-arrival metric.

Here's the harder case I want to raise myself rather than wait to be asked: strict newest-wins can miss a transient risk window. Say a late 'MFA deregistered' event has an event_time that falls *between* two observations we already processed — by the time it arrives, it's stale relative to current state, so it gets rejected, and we never reflect that the user was briefly non-compliant. My answer to that is to split the correctness guarantee in two on purpose: real-time evaluation optimizes for 'is this user risky right now' using latest-known state, which is cheap and sufficient for these checks, while the immutable observation log preserves everything needed to reconstruct history and re-run evaluation retroactively if we're ever asked 'was this user compliant on this specific date' for an audit. I want to be clear that's a deliberate tradeoff I'm choosing, not a gap I'm hoping you don't notice."

*[If asked "walk me through exactly how the trigger works" — this is where the onObservation mechanics go: the conditional write's own return value is the signal, an in-memory attribute-to-check map does the routing, no separate rule engine service. Have this ready but don't volunteer the full code-level detail unless asked.]*

### 7. Deep dive — why one box, and why versioned checks (30:00–34:00)

"Two smaller points worth stating explicitly if there's time. First, checks are triggered per changed attribute through that dependency map, not by re-running all checks on every observation — at this event rate, scanning every check every time is wasted work for no benefit, since most checks don't depend on most attributes. Second, check definitions are versioned config rather than hardcoded logic, specifically because the prompt says 'a small set' of checks today, which reads to me as a set that's going to grow — versioned config plus the ability to replay the immutable log means I can backfill findings under a new check without touching the ingestion path at all."

### 8. Failure modes and tradeoffs (34:00–40:00)

"If the consumer falls behind — say a downstream dependency slows down — findings go stale, but nothing is lost, because observations are still landing safely in the immutable log the whole time. I'd rather expose that lag explicitly through the freshness API than hide it behind an SLA nobody can see being missed. I'd put a number on it: something like 'findings reflect observations within 60 seconds at p99' as the target, with freshness as the metric that tells you when you're violating it.

On consistency: this is an eventually-consistent system overall, but because each user's events are processed by a single consumer in order, most of the races you'd otherwise worry about — two updates to the same user interleaving badly — don't actually happen. The one race that can happen is if a multi-attribute check's dependencies are updated from what would be different partitions in a type-partitioned topic design, which I'm happy to go into if that's an area you want to probe."

### 9. Close (40:00–42:00)

"To summarize the three decisions I'd defend hardest here: per-attribute versioning instead of per-record, because it's what makes one stale field not block a fresher one; treating real-time state and the audit log as two separate correctness models instead of trying to make one store serve both; and idempotent upserts as the reason I don't need exactly-once delivery anywhere in this pipeline. Happy to go deeper on any of those, or on the parts I moved quickly through."

---

## Anticipated follow-ups and one-line answers to have ready

- **"What if two attributes for the same check update concurrently from different sources?"** → Each observation type maps to a distinct attribute, so there's no write-write race on any single field; the multi-attribute *check* itself is eventually consistent and self-corrects on the next triggering event.
- **"Why not a database trigger to fire evaluation?"** → There isn't one — it's application code in the consumer; the conditional write's return value (applied or rejected) is the trigger signal, checked in the same function, not a separate DB-level mechanism.
- **"How would you shard the Kafka topics?"** → Partition by `tenant_id + user_id` by default; only split into separate topics by observation type if a high-volume, low-priority type is measurably starving a rarer, higher-priority one — not by source, since correctness here doesn't depend on cross-source ordering.
- **"What if the Findings Store can't keep up with the query patterns at scale?"** → It's already Postgres specifically because of the filtering need; the next lever is partitioning by tenant_id, then Citus/CockroachDB, before reaching for a secondary search index fed by CDC.
