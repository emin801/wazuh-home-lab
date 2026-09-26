# Wazuh Home SOC Lab

A self-built home lab for practicing log investigation and detection engineering with Wazuh: simulating attacks against a Windows endpoint and building/tuning detections to catch them.

## Why I built this

I'd been practicing offensive/defensive security concepts on TryHackMe and learning through other platforms, but wanted hands-on experience with a real SIEM.


## Architecture

```
┌─────────────────┐        ┌──────────────────┐        ┌─────────────────┐
│   Kali Linux    │        │  Windows 10        │      │  Ubuntu Server  │
│   (Attacker)    │──────▶ │  (Victim)         │─────▶│  Wazuh (SIEM)   │
│   2.5 GB RAM    │ attack │  Sysmon + Wazuh    │ logs │  4 GB RAM       │
│                 │        │  agent, 3 GB RAM   │      │                 │
└─────────────────┘        └──────────────────┘        └─────────────────┘
```

- **Host:** Windows 11, 16 GB RAM, VMs run in OracleVirtualBox
- **Wazuh server (Ubuntu):** collects and analyzes logs, runs the rule engine and dashboard
- **Windows 10 victim:** Sysmon installed and configured (using SwiftOnSecurity), Wazuh agent forwarding events
- **Kali attacker:** source of simulated attacks
- **Atomic Red Team:** Automated attacks mapped to MITRE framework

Full config files (sanitized) are in [`/configs`](./configs).


## Detections built so far

| # | Attack Simulated | MITRE ATT&CK | Detection Method | Status |
|---|---|---|---|---|
| 1 | Brute force attacks against Windows login | T1110.001 | Custom Wazuh rule on repeated 4625 (failed logon) events |  Working |
| 2 | *(next detection)* | *(technique ID)* | *rule* |  In progress |

Each detection has its own writeup in [`/detections`](./detections) with:
- The exact attack steps taken from Kali/ Atomic Red Team
- The raw log/event that got generated
- The Wazuh rule that fires on it
- A screenshot of the resulting alert in the dashboard
- Notes on tuning (reducing false positives, etc.)


## Repo structure

```
wazuh-home-lab/
├── README.md
├── architecture/        # diagram image
├── configs/             # sanitized ossec.conf, sysmon config, custom rules
├── detections/          # one folder per attack/detection scenario
└── evidence/            # screenshots, sample alert JSON
```

## Next steps

- [ ] Add detections
- [ ] Write more custom rules
- [ ] Explore active response (auto-blocking on high-severity alerts)

## Tools used

Wazuh · Sysmon · Kali Linux · Atomic Red Team · VirtualBox

---
