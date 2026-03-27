# otpot

**An actively maintained OT/ICS honeypot framework.**

otpot is a fork of [conpot](https://github.com/mushorg/conpot) by Lukas Rist and the MushMush Foundation, modernized for Python 3.10+ and extended with new industrial protocol support. It is designed to collect threat intelligence about adversaries targeting operational technology and industrial control systems.

## Why otpot?

Both conpot and riotpot are listed as inactive projects on the Honeynet Project website. There is currently no actively maintained open source OT honeypot. otpot exists to fill that gap.

**What's different from conpot:**
- Runs on Python 3.10 without workarounds or undocumented flags
- Clean install process with documented dependencies
- Active maintenance and issue response
- New protocol templates targeting modern ICS environments (in progress)
- Roadmap toward Ignition, OPC-UA, and EtherNet/IP improvements

## Supported Protocols

| Protocol | Port | Status |
|----------|------|--------|
| Modbus | 502 | ✅ Working |
| S7Comm | 102 | ✅ Working |
| BACnet | 47808 | ✅ Working |
| EtherNet/IP | 44818 | ✅ Working |
| HTTP | 80 | ✅ Working |
| FTP | 21 | ✅ Working |
| TFTP | 69 | ✅ Working |
| SNMP | 161 | ✅ Working |
| IPMI | 623 | ✅ Working |
| OPC-UA | 4840 | 🔜 Planned |
| MQTT | 1883 | 🔜 Planned |
| Ignition Gateway | 8088 | 🔜 Planned |

## Quick Start

### Requirements

- Python 3.10
- Ubuntu 22.04 / 24.04 (or WSL2)
- gcc

### Install
```bash
# Install Python 3.10 (Ubuntu 24.04)
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.10 python3.10-venv python3.10-dev gcc git

# Clone and install
git clone https://github.com/ethan-thomason/otpot.git
cd otpot
python3.10 -m venv venv
source venv/bin/activate
pip install -e .

# Run
conpot --template default --config conpot/testing.cfg
```

## Roadmap

See [ROADMAP.md](ROADMAP.md) for the full plan. High level:

- **Phase 1** — Python 3.10 stable, documented, test suite passing
- **Phase 2** — Python 3.11 support (blocked on cpppo replacement)
- **Phase 3** — HTTP fingerprinting fix
- **Phase 4** — New protocol templates: Ignition gateway, OPC-UA, MQTT
- **Phase 5** — Honeynet Project submission

## Contributing

Issues and PRs welcome. If you're interested in ICS/OT security and want to contribute protocol knowledge, open an issue and introduce yourself.

## Attribution

otpot is a fork of [conpot](https://github.com/mushorg/conpot), originally created by Lukas Rist, Johnny Vestergaard, Daniel Haslinger and contributors. Original work licensed under GPL-2.0.

## Author

Ethan Thomason — [CedarTech](https://cedartech.com) — ICS/SCADA engineer and OT security researcher.

Research and writing: [ethomason.com](https://ethomason.com)
