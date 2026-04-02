# otpot Deployment Guide

This document covers deploying otpot on a fresh Ubuntu 24.04 VPS (tested on DigitalOcean).

---

## Prerequisites

- Ubuntu 24.04 droplet (1GB RAM minimum)
- Root SSH access
- Your SSH key added to the droplet at creation time

---

## 1. Initial System Setup

SSH in as root and update the system:

```bash
apt update && apt upgrade -y
apt install git python3 python3-dev gcc -y
```

Note: if the system prompts about a pending kernel upgrade, reboot after setup.

---

## 2. Create the otpot User

```bash
useradd -m -s /bin/bash otpot
```

---

## 3. Clone the Repository

```bash
su - otpot
git clone https://github.com/ethan-thomason/otpot.git
cd otpot
```

---

## 4. Install uv and Sync Dependencies

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
cd ~/otpot
uv sync
```

`uv sync` will download and install all dependencies into a `.venv` directory. This takes about 30-60 seconds.

---

## 5. Create Required Directories

Use `/var/lib/otpot` for the temp directory — **do not use `/tmp`**, as it is cleared on reboot and will cause the service to fail to start.

```bash
exit  # back to root
mkdir -p /var/lib/otpot
chown -R otpot:otpot /home/otpot/otpot/
chown otpot:otpot /var/lib/otpot
```

---

## 6. Create the systemd Service

```bash
cat > /etc/systemd/system/otpot.service << 'EOF'
[Unit]
Description=otpot OT/ICS Honeypot
After=network.target

[Service]
Type=simple
User=otpot
WorkingDirectory=/home/otpot/otpot
ExecStart=/home/otpot/otpot/.venv/bin/python /home/otpot/otpot/bin/otpot --template default --config /home/otpot/otpot/conpot/testing.cfg --temp_dir /var/lib/otpot -f
Restart=always
RestartSec=10
WatchdogSec=300

[Install]
WantedBy=multi-user.target
EOF
```

---

## 7. Enable and Start the Service

```bash
systemctl daemon-reload
systemctl enable otpot
systemctl start otpot
systemctl status otpot
```

The service should show `active (running)`. Allow 5-10 seconds for all protocols to initialize.

---

## 8. Verify Ports Are Listening

```bash
ss -tlnp | grep -E "44818|502|47808|2121|8800"
```

Expected output:

```
LISTEN   0.0.0.0:44818   (EtherNet/IP)
LISTEN   0.0.0.0:5020    (Modbus)
LISTEN   0.0.0.0:8800    (HTTP)
LISTEN   0.0.0.0:2121    (FTP)
```

---

## 9. Validate EtherNet/IP Response

From a remote machine, confirm the honeypot is responding and the serial number is randomized:

```bash
python3 -c "
import socket, struct
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('YOUR_IP', 44818))
s.send(bytes.fromhex('63000000000000000000000000000000000000000000000000'))
data = s.recv(1024)
serial = struct.unpack_from('<I', data, 58)[0]
print('Serial number:', serial)
print('Is old hardcoded value:', serial == 7079450)
s.close()
"
```

Serial number should not be `7079450`. Restart the service and run again — the value should be different each time, confirming randomization is working.

Note: offset 58 is the correct position of the serial number field in the EtherNet/IP List Identity response. Offset 32 is the socket address field (sin_family + sin_port) and will always return a value derived from port 44818 regardless of the serial number.

---

## 10. Check Logs

```bash
tail -f /home/otpot/otpot/conpot.log
```

When searching logs, always use `grep -a` to handle binary data that may be embedded in log lines:

```bash
grep -a "search term" /home/otpot/otpot/conpot.log
```

---

## Known Issues

- **`-f` flag required**: Fresh deployments need the `-f` flag to bypass a VFS initialization issue. Root cause unknown, tracked as an open issue.
- **HTTP handler hang**: The HTTP handler can silently stop responding without crashing the process. The `WatchdogSec=300` setting mitigates this by triggering a restart. Monitor for recurrence.
- **Do not use `/tmp` for temp_dir**: `/tmp` is cleared on reboot, causing the service to fail on startup. Use `/var/lib/otpot` as shown above.

---

## Updating

To pull new code and restart:

```bash
su - otpot -c "cd ~/otpot && git pull origin main"
systemctl restart otpot
```
