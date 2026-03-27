# otpot Roadmap

## Phase 1 — Python 3.10 Stable (In Progress)

The foundation. A clean, documented, reliable otpot on Python 3.10.

**Completed:**
- [x] Remove `--force` requirement for default configuration
- [x] Remove undocumented `-f` flag requirement for non-local interfaces
- [x] Fix misleading "Can't find temp directory" log message
- [x] Document Python version support ceiling
- [x] Updated README with accurate install instructions

**In Progress:**
- [ ] Fix HTTP service leaking raw template tags (fingerprinting vulnerability)
- [ ] Run and document test suite status on Python 3.10
- [ ] Fix noisy "Running on non-local interface" warning (fires once per protocol)
- [ ] Replace deprecated `pkg_resources` usage in bin/conpot

## Phase 2 — Python 3.11 Support

The blocker is `cpppo`, the EtherNet/IP library that breaks on Python 3.11.

- [ ] Investigate maintained cpppo forks
- [ ] Evaluate `pycomm3` as a replacement
- [ ] Stub EtherNet/IP if no clean replacement found, restore in Phase 4
- [ ] Achieve clean install and run on Python 3.11

## Phase 3 — Anti-Fingerprinting

Published research has identified conpot deployments via protocol inaccuracies.
otpot should be harder to fingerprint than its predecessor.

- [ ] Fix HTTP template rendering — raw `<condata>` tags leak in responses
- [ ] Audit S7Comm response accuracy against real Siemens hardware
- [ ] Audit Modbus response timing and error codes
- [ ] Audit SNMP OID responses for consistency across protocols
- [ ] Ensure cross-protocol databus consistency (same values via HTTP and SNMP)

## Phase 4 — New Protocol Templates

otpot's unique contribution to the open source honeypot ecosystem.

**Ignition Gateway (Priority)**
- [ ] HTTP endpoints: `/StatusPing`, `/main`, `/system/webdev`
- [ ] Realistic Perspective login page
- [ ] Correct vendor banners: `Inductive-Automation/8.x.x`
- [ ] Gateway Network port 8060 basic response
- [ ] OPC-UA endpoint advertisement

**OPC-UA**
- [ ] Basic OPC-UA server emulation
- [ ] Endpoint discovery responses
- [ ] Realistic node browsing responses
- [ ] Evaluate PLC4X as protocol foundation

**MQTT**
- [ ] CONNECT/CONNACK handling
- [ ] SUBSCRIBE/SUBACK handling
- [ ] PUBLISH/PUBACK logging
- [ ] Broker identity spoofing (emulate common ICS broker software)

**EtherNet/IP Improvements**
- [ ] Deeper CIP object emulation
- [ ] Allen-Bradley/Rockwell device identity
- [ ] Evaluate PLC4X EIP implementation as replacement for cpppo

## Phase 5 — Honeynet Project Submission

- [ ] Active commit history spanning 3+ months
- [ ] At least one published writeup
- [ ] Python 3.11 support complete
- [ ] At least one new protocol template beyond conpot
- [ ] Responsive issue handling
- [ ] Email projects@honeynet.org

## Long Term

- Deploy on cloud VPS and publish threat intelligence findings
- Submit talk proposal to S4 Conference
- Explore Google Summer of Code mentorship via Honeynet
- Evaluate PLC4X integration for protocol emulation layer
