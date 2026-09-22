# Roadmap

Myaku is a standalone project. It shares an author, a design paradigm (data-oriented + functional pipeline + structured concurrency), and lessons learned from an earlier project — but not a codebase, and not a feature set.

## Core Architectural Principles

This is a solo-maintained app, not a company with an incident response team, a support contract, or insurance. The principles below exist to keep the liability surface something one person can actually stand behind. Every milestone below is checked against them.

- **Fully local.** The app runs on the user's own machine or their own server. There is no backend operated by the maintainer — no service to breach, no uptime to owe anyone.
- **No cloud service.** The maintainer never operates, and never needs credentials to, any cloud infrastructure. Where the app reads from a cloud provider (M3), that's the user's own account and credentials — never proxied through anything the maintainer runs.
- **Bring your own.** Any external dependency — a cloud account, an AI model or API — is the user's own choice and the user's own account. Cost, access, and accuracy risk sit with whatever the user connects, not with the app.
- **Read-only.** The app observes and reports. It does not execute changes against a user's infrastructure. (M10 is the one milestone with a genuine tension here — see below — and it's deliberately scoped to keep this principle true in spirit even where it bends.)

## Milestones

### M0 — Foundation & Trust *(ongoing from day one)*
- Japanese-language UI, error messages, and documentation — the first product work item, ahead of further feature milestones. Tied to targeting the Japanese enterprise market first: docs that are English-only get filtered out early by non-technical decision-makers there, even when engineers like the tool.
- Docs, architecture decisions, security policy, contribution guide.
- Signed releases established immediately, not bolted on later.

### M1 — Core MVP
- ICMP/SNMP polling, DNS/DHCP/NTP validation.
- Structured JSON/Protobuf output.
- Time-series storage (Micrometer + TimescaleDB/QuestDB) — introduced once data volume actually needs it, not preemptively.

### M2 — Root Cause Analysis *(deterministic)*
- Graph/rule-based root cause detection over time-series and relationship data. No AI dependency — this is the core differentiator, done deterministically.
- Time-series data is read back through an internal query abstraction, not by calling InfluxQL/PromQL/Graphite directly — mirrors the emission-side abstraction Micrometer already provides in M1, and is what makes "pluggable time-series backend" true end-to-end rather than just on the write side. Before claiming multi-backend support publicly: verify against at least two backends (e.g. TimescaleDB and Prometheus) with the adapter layer in place.
- Built-in rule coverage is deliberately narrow at launch — a small set of high-confidence, well-tested patterns (DNS resolution failure, TLS handshake failure, packet-loss threshold breach) rather than broad, thin coverage. A deterministic engine either matches a known pattern or it doesn't; it can't hallucinate a plausible-sounding wrong cause, so "no match" is the honest, correct output of a narrow rule set.
- "No root cause identified" is a designed, first-class result state, not a blank failure screen — it shows what was checked and ruled out.
- User-authored custom rules are additive only, and never override a built-in rule's conclusion. A custom rule can add a new finding to the ranked output, but can't suppress or replace a built-in one — a case an operator is certain "can never happen" could still happen someday, and silently masking the built-in engine's correct answer would cause real damage.
- Real-world custom rules collected after launch are a natural path to widening built-in coverage over time.

### M3 — Cloud Telemetry Ingestion
- Parsers for AWS VPC Flow Logs, Azure NSG flow logs, GCP VPC flow logs.
- Reads from the user's own storage (S3 / Blob / GCS) using the user's own credentials — the one place the app talks to a cloud provider's API at all, and it never runs infrastructure to do it.
- Off by default. A fully-local install never touches this milestone unless the user explicitly configures cloud credentials.
- Data minimization (strip/hash source IPs) and the GDPR "data processor" question are tracked under Legal & Compliance, below.

### M4 — Lightweight Topology View
- Force-directed graph from existing scan/relationship data.
- Deliberately minimal — not a full graph-database-backed version, just enough to remove "no visual map" as a disqualifier. Full topology support waits for real scale.

### M5 — Integration Surface
- Webhooks, Slack/Teams, PagerDuty/Opsgenie, documented REST API.
- These are outbound notifications the app sends, not a service the app depends on — consistent with "no cloud service."

### M6 — L5–L7 Protocol / Application Checks
- HTTP/2–3, TLS handshake validation, EDNS0/DNSSEC, BGP/OSPF neighbor state checks.
- Design note: neighbor-absent is modeled as its own distinct monitored state, not inherited from generic SNMP polling's implicit "object exists / doesn't" assumption. Existing SNMP-based BGP/OSPF tooling has a documented failure pattern here — when a neighbor session drops, its SNMP objects disappear from the device entirely, so a naive poll-and-discover approach reports "item unsupported" rather than "neighbor down," right when that distinction matters most.

### M7 — Deployment Flexibility
- Headless/server mode, web dashboard served via the API.
- Still self-hosted by the user — "local" means "the user's own infrastructure," not "single machine only."

### M8 — Tactical Utilities
- iperf3-style throughput generator, IPAM, PCAP buffering, visual BGP/ASN traceroute.
- Shipped individually, by standalone value, rather than as one bundled release.

### M9 — Distributed Architecture *(deferred until real demand)*
- Lightweight worker nodes with central aggregation.

### M10 — Remediation *(suggestion-only, no execution capability)*
- Git-based config drift detection and a suggested fix, generated as a diff or script for a human to review and run themselves. The app never executes the fix.
- This keeps "read-only" true rather than treating it as a bent exception — the liability question changes from "did the app's write cause damage" to "was the app's suggestion reasonable," a materially smaller risk for a solo maintainer.
- Still last, still gated hardest (see Legal & Compliance). A bad suggestion is a much smaller problem than a bad action, but not a zero problem.

### M11 — AI Layer *(bring-your-own-model)*
- Pluggable interface: OpenAI-compatible endpoint, Anthropic API, or a local model (Ollama).
- The app's job is context assembly only. Cost, accuracy, and data-handling risk sit with the user's chosen model.

**Architectural note (not a milestone):** keep the data model (`ScanRequest`, `Report`, etc.) open to a "workspace" concept now, even though multi-tenancy/RBAC isn't being built yet.

## After M2 Stabilizes

One real deployment story does more for credibility than additional feature milestones — worth planning once M0–M2 are solid.

Go-to-market and discovery channel strategy (Qiita/Zenn technical articles, connpass meetups, SIer community word-of-mouth — the channels that matter for the Japanese market, distinct from GitHub trending or Hacker News) is deliberately deferred until after M2, not decided now.

## Legal & Compliance Track *(parallel to M0–M11)*

Two checkpoints are marked 🛑 **STOP — get a real review**; a checklist is enough for everything else.

**From M0** *(before any public code or release)*
- Choose an OSS license deliberately.
- Dependency license audit habit (flag copyleft dependencies before adopting).
- Visible abuse-reporting contact channel.
- Baseline ToS / liability disclaimer — easier to write plainly here, since "the app never writes to your infrastructure and never holds your data" is true by design, not just by policy.
- Public support/credibility commitment (even lightweight — e.g. a stated issue-response time expectation). Enterprise buyers in Japan typically go through an internal approval process that asks who they call when it breaks; a solo maintainer needs to state a realistic support posture upfront.

**At M3 — Cloud Telemetry Ingestion**
- Data minimization on flow logs (strip/hash source IPs).
- Review whether this makes the app a "data processor" under GDPR-type law, even though it's opt-in and credential-scoped to the user's own account.

**At M8 — Tactical Utilities** *(active scanning / PCAP)*
- Per-target authorization confirmation gate at the moment of the action.
- Passive/read-only by default; active scanning is explicit opt-in.
- Rate-limiting, backoff, local audit logging.

**At M10 — Remediation** *(suggestion-only)*
- ToS clause: suggested fixes are advisory; the user executing them is a deliberate human action outside the app.
- Lower bar than a full pre-launch legal review, since there's no execution path — but still worth a checklist pass given it touches config data.

**At M11 — AI Layer**
- ToS clause disclaiming liability for the user's own connected model's output.

**🛑 Gate before any public launch / marketing push**
- Trademark search on the app name.
- Business entity (LLC-equivalent), separate from personal liability.
- Hosting/package registry sanctions-list geofencing check.
- Professional legal review — the one remaining hard gate.
