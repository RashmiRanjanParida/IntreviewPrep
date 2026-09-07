# System Design Interview Prep

Each subfolder is one system design problem worked through end-to-end: requirements, capacity estimation, API, data model, major components, hard-problem deep dives, an architecture diagram, and an interview run-through with a staff/principal signal checklist.

| Problem | Folder |
|---|---|
| TinyURL | [`tinyurl/`](./tinyurl) |
| Multi-Tenant Risk Assessment | [`multitenant-risk-assessment/`](./multitenant-risk-assessment) |
| OAuth App Risk | [`oauth-app-risk/`](./oauth-app-risk) |
| Sign-In Risk Scoring | [`signin-risk-scoring/`](./signin-risk-scoring) |
| Multi-Tenant Policy Evaluation Engine | [`policy-engine/`](./policy-engine) |
| Multi-Tenant Secrets & Certificate Lifecycle (PKI) | [`pki-lifecycle/`](./pki-lifecycle) |
| API Rate Limiter | [`rate-limiter/`](./rate-limiter) |
| Distributed Key-Value Store | [`kv-store/`](./kv-store) — includes follow-up Q&A on stale reads, quorum math, gossip/SWIM mechanics, and hinted handoff, folded into the main doc |
| Document Version History Service | [`doc-version-history/`](./doc-version-history) |

Each folder contains the design doc (`*_system_design.md`) and its rendered architecture diagram (`*_flow.png`). The multi-tenant risk assessment folder additionally has an interview delivery script.
