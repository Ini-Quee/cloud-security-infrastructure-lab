<div align="center">

# 🛡️ Cloud Security Infrastructure Lab

**A real-world cloud security monitoring environment built on Microsoft Azure and Linux**

[![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com)
[![Linux](https://img.shields.io/badge/OS-Ubuntu%2022.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-FF6633?style=for-the-badge)](https://wazuh.com)
[![Fail2Ban](https://img.shields.io/badge/IPS-Fail2Ban-8B0000?style=for-the-badge)](https://www.fail2ban.org)
[![UFW](https://img.shields.io/badge/Firewall-UFW-228B22?style=for-the-badge)](https://help.ubuntu.com/community/UFW)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)](https://github.com/Ini-Quee/cloud-security-infrastructure-lab)

</div>

---

## 📌 Project Overview

This lab demonstrates how to design, deploy, and monitor a **cloud security infrastructure** from scratch using real tools on a live Azure environment. Every component was manually configured, tested, and documented — including real attack simulations against the server.

The goal was to build something that mirrors what security teams actually deal with: a live server exposed to the internet, actively receiving attack attempts, with detection and automated response systems in place.

> ⚠️ **Note:** This is a real environment. During testing, genuine external IP addresses were observed attempting brute-force SSH logins — not simulated traffic.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Microsoft Azure Cloud                  │
│                                                          │
│   ┌──────────────────────────────────────────────────┐  │
│   │          Ubuntu 22.04 Virtual Machine             │  │
│   │                                                   │  │
│   │  ┌─────────────┐     ┌──────────────────────┐    │  │
│   │  │  UFW Firewall│────▶│   SSH (Port 22)      │    │  │
│   │  │  (Perimeter) │     │   Auth Monitoring    │    │  │
│   │  └─────────────┘     └──────────┬───────────┘    │  │
│   │                                 │                 │  │
│   │                                 ▼                 │  │
│   │                    ┌────────────────────┐         │  │
│   │                    │   Wazuh Agent       │         │  │
│   │                    │  (Log Collection)   │         │  │
│   │                    └──────────┬─────────┘         │  │
│   │                               │                   │  │
│   │  ┌─────────────┐              │                   │  │
│   │  │  Fail2Ban   │◀─────────────┘                   │  │
│   │  │ (Auto-Block)│                                   │  │
│   │  └─────────────┘                                   │  │
│   └──────────────────────────────────────────────────┘  │
│                          │                               │
│                          ▼                               │
│              ┌────────────────────┐                      │
│              │   Wazuh Manager    │                      │
│              │   + Dashboard      │                      │
│              │  (SIEM & Alerts)   │                      │
│              └────────────────────┘                      │
└─────────────────────────────────────────────────────────┘

         Attacker ──▶ UFW blocks ──▶ Fail2Ban bans IP
                   ──▶ Wazuh detects & alerts
```

---

## 🔧 Tools & Technologies

| Tool | Role | Why It Was Used |
|------|------|-----------------|
| **Microsoft Azure** | Cloud infrastructure | Deploy and manage the VM in a real cloud environment |
| **Ubuntu 22.04 LTS** | Operating system | Industry-standard Linux for security environments |
| **Wazuh SIEM** | Security monitoring & alerting | Collect logs, detect threats, visualize security events |
| **Fail2Ban** | Intrusion prevention | Automatically ban IPs after repeated failed logins |
| **UFW Firewall** | Network perimeter control | Restrict inbound/outbound traffic to only what's needed |
| **SSH Logs** | Authentication monitoring | Track login attempts, failures, and unauthorized access |

---

## ⚙️ Configuration Highlights

### UFW Firewall Rules
```bash
# Allow SSH only
sudo ufw allow 22/tcp

# Allow Wazuh manager communication
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp

# Enable firewall
sudo ufw enable

# Verify status
sudo ufw status verbose
```

### Fail2Ban — SSH Jail Config
```ini
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 3600
findtime = 600
```

### Wazuh Agent Registration
```bash
# Install agent and connect to Wazuh Manager
sudo WAZUH_MANAGER='<manager-ip>' \
     WAZUH_AGENT_NAME='azure-ubuntu-vm' \
     apt-get install wazuh-agent

# Enable and start agent
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## 🚨 Attack Simulation & Detection

To validate the monitoring setup, SSH brute-force attempts were simulated against the server. The following was observed and detected:

- ❌ Multiple failed authentication attempts from external IPs
- ❌ Invalid username login attempts
- ❌ Repeated unauthorized access attempts within short time windows
- ✅ Wazuh detected and alerted on all suspicious activity
- ✅ Fail2Ban automatically banned offending IPs after 3 failed attempts
- ✅ UFW firewall blocked all non-whitelisted traffic

> During live testing, **real external IP addresses** were also observed attempting logins — demonstrating that any internet-exposed server faces constant automated attack attempts.

---

## 📸 Screenshots

### Azure Virtual Machine — Deployment
![Azure VM](screenshots/01-azure-vm.png)

### Wazuh Dashboard — Security Events
![Wazuh Dashboard](screenshots/02-wazuh-dashboard.png)

### Wazuh Agent — Connected & Active
![Agent Status](screenshots/03-agent-status.png)

### UFW Firewall — Active Rules
![UFW Rules](screenshots/04-ufw-rules.png)

### Fail2Ban — Banned IPs After Attack Simulation
![Fail2Ban](screenshots/05-fail2ban-status.png)

---

## 🧠 Key Lessons Learned

| Challenge | What Happened | What I Learned |
|-----------|--------------|----------------|
| Wazuh service failing to start | Invalid tag in `ossec.conf` broke the entire service | One misconfiguration can take down critical security infrastructure — log reading and service debugging are essential skills |
| VM disk full | Expanded virtual disk in VirtualBox but Linux still showed old size | Virtual disk size and Linux filesystem size are two separate layers — both must be expanded |
| Clipboard not working between host and VM | VirtualBox Shared Clipboard setting enabled but still not working | Guest Additions must be installed inside the VM for clipboard sharing to actually function |
| Git push failing | Large files caused connection reset during push | Increasing Git's HTTP buffer resolves large file transfer issues |
| Real attacker IPs in logs | Genuine external IPs attempting SSH brute-force within hours of deployment | Any internet-exposed server is actively targeted — this is not theoretical |

---

## 🔮 What's Next

This lab is actively evolving. Current work in progress:

- [ ] **Multi-VM monitoring** — connect a second Azure VM as an additional Wazuh agent
- [ ] **Kubernetes security lab** — deploy a K8s cluster and monitor container workloads with Wazuh
- [ ] **Custom Wazuh rules** — write detection rules for specific attack patterns
- [ ] **Alert automation** — trigger automated responses based on Wazuh alert levels

---

## 📁 Repository Structure

```
cloud-security-infrastructure-lab/
│
├── screenshots/
│   ├── 01-azure-vm.png
│   ├── 02-wazuh-dashboard.png
│   ├── 03-agent-status.png
│   ├── 04-ufw-rules.png
│   └── 05-fail2ban-status.png
│
└── README.md
```

---

## 👩🏽‍💻 About

Built by **Erica Innocent Effiong** — Cloud & Security Enthusiast based in Lagos, Nigeria.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/erica-innocent-542147265/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github)](https://github.com/Ini-Quee)

---

<div align="center">

*This project is part of an ongoing hands-on cloud security portfolio.*
*Every problem documented here was a real problem encountered and solved.*

</div>
