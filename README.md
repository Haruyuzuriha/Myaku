# 脈 Myaku

**Read the pulse of your network.**

Myaku is an open-source, enterprise-grade network health check tool for Java, built to give infrastructure teams a clear, deterministic read on the state of their network — without shipping their data anywhere.

Part of the [Haruyuzuriha (春楪)](#) project family.

---

## Philosophy

Most network monitoring tools ask you to trust a vendor's cloud with your infrastructure data. Myaku doesn't.

- **Fully local.** No cloud service. Nothing leaves your environment.
- **Bring your own storage.** Plug in the time-series backend you already run (TimescaleDB by default, Prometheus and others via an adapter layer).
- **Read-only by default.** Myaku diagnoses. It doesn't touch your infrastructure unless you explicitly opt in to remediation, and even then, every action starts as a dry run.
- **Explainable, not magic.** Root cause analysis is deterministic and rule-based — not a black box. Every conclusion shows the observations and rules that led to it.

## What it does

Myaku polls and correlates signals across your network to answer one question fast: *what's actually wrong, and why?*

- ICMP/SNMP polling, DNS/DHCP/NTP validation
- Deterministic root cause analysis via correlated identifiers (target, timestamp, network segment)
- L5–L7 protocol and application-layer checks (HTTP/2–3, TLS handshake validation, EDNS0/DNSSEC, BGP/OSPF neighbor state)
- Cloud telemetry ingestion (AWS/Azure/GCP flow logs, read from your own storage — no infrastructure of ours involved)
- Pluggable time-series backend via an internal query abstraction layer
- Tactical utilities: throughput generation, IPAM, PCAP capture, visual BGP/ASN traceroute
- Optional AI layer (bring-your-own-model, for historical trend analysis only — never in the active detection or remediation path)

See the [roadmap](#ROADMAP.md) for the full milestone breakdown (M0–M11).

## Status

🚧 Early development. Following a milestone-based roadmap (M0: Foundation & Trust → M1: Core MVP → ... → M11: AI layer). Not yet ready for production use.

## Why "Myaku"?

脈 (myaku) means *pulse* — the same word used for a doctor checking a patient's vital signs, and for describing a network of connections (人脈, 山脈). That's what this tool does: it reads your network's vital signs and tells you, plainly, what they mean.

## License

TBD — license selection is part of the M0 milestone.

## Contributing

Contribution guidelines are being finalized as part of M0. Check back soon, or open an issue to discuss.
