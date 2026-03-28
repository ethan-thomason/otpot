# otpot

**An actively maintained OT/ICS honeypot framework.**

otpot is a fork of [conpot](https://github.com/mushorg/conpot) by Lukas Rist and the MushMush Foundation, modernized for Python 3.10+ and extended with new industrial protocol support. It is designed to collect threat intelligence about adversaries targeting operational technology and industrial control systems.

## Why otpot?

Both conpot and riotpot are listed as inactive projects on the Honeynet Project website. There is currently no actively maintained open source OT honeypot. otpot exists to fill that gap.

**What's different from conpot:**
- Runs on Python 3.10 without workarounds or undocumented flags
- Clean install process with documented dependencies
- HTTP template rendering fixed — no more raw template tags leaking in responses
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

## Quick Start (Local / Development)

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
otpot --template default --config conpot/testing.cfg
```

## Production Deployment

### Requirements

- A VPS or server running Ubuntu 22.04 / 24.04
- **Do not run as root.** Create a dedicated user.

### Setup

```bash
# Create a dedicated user
useradd -m -s /bin/bash otpot

# Install dependencies as root
apt update && apt upgrade -y
apt install -y python3.10 python3.10-venv python3.10-dev gcc git software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt update && apt install -y python3.10 python3.10-venv python3.10-dev

# Switch to otpot user
su - otpot

# Clone and install
git clone https://github.com/ethan-thomason/otpot.git
cd otpot
python3.10 -m venv venv
source venv/bin/activate
pip install -e .
```

### Run as a systemd service

As root, create `/etc/systemd/system/otpot.service`:

```ini
[Unit]
Description=otpot OT/ICS Honeypot
After=network.target

[Service]
Type=simple
User=otpot
WorkingDirectory=/home/otpot/otpot
ExecStart=/home/otpot/venv/bin/otpot --template default --config /home/otpot/otpot/conpot/testing.cfg
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Then enable and start:

```bash
systemctl daemon-reload
systemctl enable otpot
systemctl start otpot
systemctl status otpot
```

### Viewing logs

```bash
# Via journald
journalctl -u otpot -f

# Via log file
tail -f /home/otpot/otpot/conpot.log

# Filter out your own IP when sharing logs
grep -v "YOUR_IP_HERE" /home/otpot/otpot/conpot.log
```

### DNS

For more convincing deception, point a subdomain at your honeypot IP. A hostname like `plc01.yourcompany.com` looks significantly more like real OT infrastructure than a bare IP address. Avoid obviously research-oriented names.

## Roadmap

See [ROADMAP.md](ROADMAP.md) for the full plan. See [VISION.md](VISION.md) for the longer-term product direction.

High level:

- **Phase 1** — Python 3.10 stable, documented, test suite passing
- **Phase 2** — Python 3.11 support (blocked on cpppo replacement)
- **Phase 3** — Anti-fingerprinting improvements
- **Phase 4** — New protocol templates: Ignition gateway, OPC-UA, MQTT
- **Phase 5** — Container-native deployment, T-Pot integration, Honeynet submission

## Contributing

Issues and PRs welcome. If you are interested in ICS/OT security and want to contribute protocol knowledge, open an issue and introduce yourself.

## Attribution

otpot is a fork of [conpot](https://github.com/mushorg/conpot), originally created by Lukas Rist, Johnny Vestergaard, Daniel Haslinger and contributors. Original work licensed under GPL-2.0.

## Author

Ethan Thomason — [CedarTech](https://cedartech.com) — ICS/SCADA engineer and OT security researcher.

Research and writing: [ethomason.com](https://ethomason.com)
