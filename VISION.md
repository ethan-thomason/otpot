# otpot Vision

## What otpot is trying to be

otpot is not trying to be the honeypot with the most protocols. It is trying to be the honeypot that is hardest to distinguish from a real OT device — and the easiest to deploy, extend, and integrate into existing threat intelligence infrastructure.

The open source OT honeypot space is effectively empty. Both conpot and riotpot are listed as inactive projects on the Honeynet Project website. The tools that exist were built by academics during funded periods and then abandoned. The ICS/OT threat landscape has evolved significantly since then. otpot exists to fill that gap with something practitioners actually want to run.

---

## Core principles

### Realism over protocol count

A honeypot that emulates twenty protocols poorly is less valuable than one that emulates five protocols convincingly. Sophisticated attackers and automated scanners have learned to identify honeypots by their inaccuracies — wrong timing, inconsistent responses, static register values, missing vendor quirks. otpot's primary competitive advantage is believability, not breadth.

This means:

**Believable identity.** Every emulated device should have a consistent, realistic vendor and model identity — correct firmware version strings, accurate SNMP OIDs, matching HTTP banners, consistent hardware descriptions across all protocols. A device that identifies as a Siemens S7-300 via S7Comm but returns generic strings via SNMP is immediately suspicious.

**Timing jitter.** Real PLCs have scan cycles. Real devices have response latency that varies slightly with load. Static, instant responses are a fingerprinting signal. otpot should introduce configurable timing jitter that mimics the behavior of real hardware.

**Stateful register behavior.** A real PLC's register values change over time as the process runs. A honeypot that returns the same values every time is trivially identified. otpot should maintain stateful register/tag memory that evolves plausibly — temperature values that drift, motor speeds that ramp, level sensors that fluctuate within realistic bounds.

**Read coherence.** Values read via Modbus, S7Comm, OPC-UA, and the HMI web interface should be consistent with each other. Cross-protocol inconsistency is a fingerprinting vector.

**Limited write behavior with plausible side effects.** When an attacker writes to a register, something should appear to happen. The value should change. Related values should update plausibly. This encourages attackers to continue interacting and reveal more of their playbook.

**Honeytokens.** SNMP MIB files, project files, HMI artifacts, and manual pages embedded in the honeypot that, when accessed, generate canary alerts. A threat actor who downloads a "ladder logic project file" has revealed significant intent.

---

### Container-native from the start

The deployment experience is part of the product. A honeypot that takes hours to set up correctly — dealing with Python version conflicts, undocumented flags, permission issues, and missing build dependencies — will not be widely deployed. Wide deployment means more data, more research, more community.

otpot should be deployable with a single command:

```bash
docker run -d --name otpot -p 502:502 -p 102:102 -p 80:80 ghcr.io/ethan-thomason/otpot
```

Container-native architecture also solves several practical problems:
- No root requirement — container handles privilege separation
- Reproducible deployments across environments
- Easy integration with T-Pot and other honeypot orchestration platforms
- Clean separation between the honeypot process and the host system

---

### T-Pot compatible packaging

T-Pot is the most widely deployed honeypot platform in the world with thousands of active instances. It is actively maintained by Telekom Security and listed as an active Honeynet Project. A honeypot module that integrates cleanly with T-Pot inherits that deployment footprint immediately.

otpot should be designed to run as a T-Pot module from the beginning. This means following T-Pot's container conventions, outputting logs in a format compatible with T-Pot's ELK stack, and participating in the T-Pot data sharing ecosystem.

---

### Build on the best ideas in the ecosystem

Several prior projects have contributed valuable architectural thinking that otpot should build on and extend.

riotpot, though no longer actively maintained, introduced a control-plane architecture with genuinely good ideas worth preserving:

**Device profiles.** A profile is a named configuration that makes otpot resemble a specific real-world device — a Siemens S7-300, an Allen-Bradley MicroLogix, a Schneider Modicon, an Inductive Automation Ignition gateway. Profiles define which protocols are active, what identity strings are presented, what register values look like, and what the HMI surface shows. Switching profiles should be a one-line configuration change.

**REST API.** otpot should expose a REST API for runtime management — querying active sessions, retrieving captured data, updating register values, switching profiles, and configuring logging destinations. This enables integration with SIEM platforms, dashboards, and automated threat intel pipelines.

**Proxy/routing logic.** The ability to route specific protocol traffic to external services — a real simulator, a high-interaction backend, a containerized PLC — is architecturally valuable. otpot should support this as an optional advanced mode without requiring it for basic operation.

**Optional UI.** A lightweight web dashboard for monitoring active sessions and captured data is valuable for non-technical operators and demo contexts. It should be optional and not required for core functionality.

Apache PLC4X provides a comprehensive open source ICS protocol library that otpot should evaluate as a foundation for protocol implementations — particularly for OPC-UA and EtherNet/IP, where existing Python libraries are either unmaintained or incomplete.

---

### Design for hybrid interaction later

otpot v1 is a low-interaction honeypot. It emulates protocol behavior without running real industrial software. This is the right starting point — low-interaction honeypots are safe, lightweight, and easy to deploy at scale.

But the interesting research starts at higher interaction levels. Projects like HoneyPLC are already exploring higher-interaction PLC emulation, including capturing and storing ladder logic. otpot's architecture should leave the door open for hybrid interaction without requiring it upfront.

The right design is a clean bridge interface: low-interaction protocol handlers that can optionally hand off to a higher-interaction backend when one is available. This means otpot can be useful immediately as a low-interaction sensor while growing toward more sophisticated emulation over time.

---

## v1 Target: believable PLC-level emulation

The v1 milestone is a honeypot that passes casual inspection by a human operator and defeats automated fingerprinting tools. Specifically:

- **Believable vendor/model identity** — correct version strings, firmware identifiers, and hardware descriptions across all active protocols
- **Realistic protocol handshakes** — responses that match what real devices return, including correct error codes for unsupported functions
- **Stateful register/tag memory** — values that persist across reads and evolve plausibly over time
- **Read support that looks coherent** — consistent values across protocols, realistic data types and ranges
- **Limited write behavior with plausible side effects** — writes are accepted and reflected in subsequent reads
- **Timing jitter and scan-cycle-ish behavior** — response latency that mimics real hardware
- **A tiny web/HMI surface** — a realistic status page with live-ish process values
- **Honeytokens** — SNMP MIBs, project files, and manual artifacts that generate canary alerts when accessed

---

## Protocol priorities

**Tier 1 — ship with v1:**
- Modbus TCP (already working, needs realism improvements)
- S7Comm (already working, needs realism improvements)
- HTTP/HMI surface (already working, fingerprinting fix shipped)
- SNMP (already working, needs cross-protocol consistency)

**Tier 2 — ship before Honeynet submission:**
- OPC-UA (evaluate PLC4X as protocol foundation)
- EtherNet/IP CIP improvements (replace cpppo with pycomm3 or PLC4X)
- Ignition gateway fingerprint (unique contribution, no open source equivalent exists)
- MQTT (common in modern ICS/IoT environments)

**Tier 3 — longer term:**
- BACnet improvements (building automation targeting)
- Profinet (European manufacturing)
- DNP3 (utilities/energy sector)
- IEC 60870-5-104 (power grid)

---

## The Ignition opportunity

Inductive Automation's Ignition is a dominant SCADA platform in North American manufacturing. It has a specific, identifiable network fingerprint — HTTP gateway interface, OPC-UA endpoint, Gateway Network port, Perspective web client. Attackers scanning for industrial targets actively look for it.

No open source honeypot currently emulates Ignition. This is otpot's most unique potential contribution. A convincing Ignition gateway honeypot would capture targeted threat intelligence that does not currently exist in the community — specific tooling, specific exploits, specific attacker behavior directed at the most widely deployed SCADA platform in North America.

This contribution is only possible because of deep domain expertise with the platform. It cannot be built from a spec document alone.

---

## What success looks like

**6 months:** Active Honeynet Project listing. Python 3.11 support. Container-native deployment. At least one published threat intel writeup from live deployment data.

**12 months:** T-Pot integration. Ignition template live. OPC-UA support. REST API. Multiple community contributors. Conference talk accepted.

**24 months:** The de facto open source OT honeypot. GSoC mentorship. Cited in academic research. Meaningfully harder to fingerprint than any predecessor.
