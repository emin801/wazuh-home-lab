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
| 1 | Password spraying against Windows login | T1110.003 | Custom Wazuh rule on repeated 4625 (failed logon) events | ✅ Working |
| 2 | *(next detection)* | *(technique ID)* | *(rule/decoder used)* | 🚧 In progress |

Each detection has its own writeup in [`/detections`](./detections) with:
- The exact attack steps taken from Kali
- The raw log/event that got generated
- The Wazuh rule (or custom rule XML) that fires on it
- A screenshot of the resulting alert in the dashboard
- Notes on tuning (false positives encountered, thresholds adjusted, etc.)

### Example: Password Spray Detection

**Attack:** Simulated repeated failed logon attempts against the Windows victim from Kali using [Hydra/CrackMapExec — *fill in tool*].

**Detection logic:** Sysmon/Windows Security event 4625 (failed logon) is forwarded to Wazuh via the agent. A custom rule groups failed logons by source IP over a rolling window and fires when the count exceeds a threshold, correlating against the standard Windows authentication rule group.

```xml
<!-- example — replace with your actual rule -->
<rule id="100010" level="10">
  <if_group>authentication_failed</if_group>
  <same_source_ip />
  <description>Possible password spray: multiple failed logons from same source</description>
  <mitre>
    <id>T1110.003</id>
  </mitre>
</rule>
```

**Result:** Alert fires in the Wazuh dashboard with rule level 10, tagged to MITRE T1110.003. See [`/detections/password-spray`](./detections/password-spray) for the full alert JSON and screenshot.

## What I learned

- [e.g., "Default Sysmon config is noisy — had to filter out X to cut false positives"]
- [e.g., "Wazuh's rule correlation (`frequency`/`timeframe`) is what actually makes a threshold-based detection work, not just a single-event rule"]
- [e.g., "Understanding *why* a detection maps to a specific ATT&CK technique matters more than just getting an alert to fire"]

## Repo structure

```
wazuh-home-lab/
├── README.md
├── architecture/       # diagram source + image
├── configs/             # sanitized ossec.conf, sysmon config, custom rules/decoders
├── detections/          # one folder per attack/detection scenario
└── evidence/            # screenshots, sample alert JSON
```

## Next steps

- [ ] Add detection for [e.g., PsExec-style lateral movement]
- [ ] Add detection for [e.g., LSASS credential dumping]
- [ ] Write custom decoder for [tool/log source]
- [ ] Explore active response (auto-blocking on high-severity alerts)

## Tools used

Wazuh · Sysmon · Kali Linux · [Atomic Red Team, if used] · VirtualBox/VMware

---

*This is a learning project built for hands-on detection engineering practice. Configs in this repo are sanitized (no real IPs, hostnames, or credentials).*
