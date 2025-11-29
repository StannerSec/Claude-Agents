# Blue Team Report Findings Skill

## Overview
This skill enables you to create professional incident response reports using the SysReptor API. Use this skill after completing your analysis to document findings, IOCs, impact assessments, and remediation recommendations in a structured format.

## Purpose
Transform technical analysis results into professional incident reports that include:
- Commands executed during investigation
- IOCs (Indicators of Compromise) identified
- Impact assessment and affected systems
- Timeline of events
- Remediation recommendations
- MITRE ATT&CK technique mapping

---

## Prerequisites

### Environment Setup
Before creating reports, ensure the SysReptor environment is activated:

```bash
source ~/venvs/reptor-main-env/bin/activate
```

---

## SysReptor Workflow

### Step 1: Create a Project

Create a new project for your incident investigation:

```bash
reptor createproject \
  --name "Blue Team Incident Response - 2025-11-29" \
  --design "8a6ebd7b-637f-4f38-bfdd-3e8e9a24f64e" \
  --tags blue-team,incident-response
```

**Note**: This command automatically updates `~/.sysreptor/config.yaml` with the new project ID.

---

### Step 2: Create Findings

#### Finding Structure

Create findings using JSON format. Example `finding.json`:

```json
{
  "status": "in-progress",
  "data": {
    "title": "Unauthorized Access Attempt Detected",
    "cvss": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N",
    "summary": "Multiple failed SSH authentication attempts from suspicious IP",
    "description": "Log analysis revealed 247 failed SSH login attempts from IP 192.168.1.100 targeting admin accounts over a 15-minute period. Pattern indicates potential brute-force attack.",
    "recommendation": "1. Block source IP at firewall level\n2. Enable fail2ban with stricter thresholds\n3. Implement MFA for SSH access\n4. Review and disable unnecessary admin accounts",
    "affected_components": [
      "SSH Server (10.0.1.50:22)",
      "Admin accounts: root, admin, sysadmin"
    ],
    "references": [
      "https://attack.mitre.org/techniques/T1110/001/",
      "https://www.cisa.gov/ssh-brute-force-attacks"
    ]
  }
}
```

#### Submit a Finding

```bash
cat finding.json | reptor finding
```

#### Update an Existing Finding

```bash
cat updated_finding.json | reptor finding --update <FINDING_ID>
```

**Note**: Finding ID is found in the URL when viewing the finding in SysReptor

---

### Step 3: Submit Multiple Findings

You can submit multiple findings at once using a JSON array:

```json
[
  {
    "status": "in-progress",
    "data": {
      "title": "First Finding",
      "cvss": "CVSS:3.1/...",
      ...
    }
  },
  {
    "status": "completed",
    "data": {
      "title": "Second Finding",
      "cvss": "CVSS:3.1/...",
      ...
    }
  }
]
```

```bash
cat multiple_findings.json | reptor finding
```

---

### Step 4: Alternative Format - TOML

You can also use TOML format for findings:

```toml
status = "in-progress"

[data]
title = "Malware Detection on Endpoint"
cvss = "CVSS:3.1/AV:L/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:H"
summary = "Trojan detected in user downloads directory"
description = "EDR solution flagged suspicious executable with ransomware indicators"
recommendation = "Isolate endpoint, run full scan, restore from clean backup"
affected_components = ["Workstation-042", "User: jdoe"]
references = ["https://attack.mitre.org/techniques/T1204/"]
```

```bash
cat finding.toml | reptor finding
```

---

### Step 5: Generate Reports

Once all findings are documented, generate the final report:

```bash
# Render final PDF report
reptor project --render -o incident_report.pdf

# Search for projects
reptor project --search "blue-team"
```

---

## Quick Finding Creation Script

Use this bash script to quickly add findings during incident response:

```bash
#!/bin/bash
# Quick script to add findings during incident response

create_finding() {
  cat << EOF | reptor finding
{
  "status": "in-progress",
  "data": {
    "title": "$1",
    "cvss": "$2",
    "summary": "$3",
    "description": "$4",
    "recommendation": "$5",
    "affected_components": ["$6"],
    "references": []
  }
}
EOF
}

# Usage example
create_finding \
  "Suspicious PowerShell Execution" \
  "CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H" \
  "Obfuscated PowerShell command detected" \
  "Process monitoring detected encoded PowerShell execution with network callbacks" \
  "Kill process, analyze memory dump, check for persistence mechanisms" \
  "Server-WEB-01"
```

---

## Reference Documentation

- [SysReptor Projects](https://docs.sysreptor.com/cli/projects-and-templates/project/)
- [SysReptor Findings](https://docs.sysreptor.com/cli/projects-and-templates/finding/)
- [SysReptor Configuration](https://docs.sysreptor.com/cli/configuration/)
- [Create Project Command](https://docs.sysreptor.com/cli/projects-and-templates/createproject/)

---

## Incident Report Field Schema

### Required Fields

| Field Name         | Type            | Description                                              |
|--------------------|-----------------|----------------------------------------------------------|
| title              | String          | Incident title/name                                      |
| incident_id        | String          | Unique incident identifier (e.g., INC-2025-11-29-001)    |
| incident_severity  | String          | Severity level (Low/Medium/High/Critical)                |
| incident_status    | String          | Current status (detected/contained/eradicated/recovered) |
| incident_overview  | String          | High-level summary of the incident                       |
| key_findings       | String/Markdown | Bullet points of key discoveries                         |
| immediate_actions  | String/Markdown | Actions taken during response                            |
| stakeholder_impact | String          | Business/operational impact                              |
| affected_systems   | String/Markdown | Compromised or targeted systems                          |
| evidence_sources   | String/Markdown | SIEM queries, logs, tools used for detection             |
| ioc                | String/Markdown | Indicators of Compromise for threat hunting              |
| root_cause         | String/Markdown | Root cause analysis and vulnerabilities                  |
| timeline           | String/Markdown | Chronological sequence of events                         |
| nature             | String/Markdown | Attack type, TTPs, MITRE ATT&CK mapping                  |

---

## Example: Complete Incident Report

### Sample Project Structure

```json
{
  "id": "5078d82a-0ad2-4d09-a76e-950c1e8845dd",
  "created": "2025-11-29T18:08:39.329167Z",
  "updated": "2025-11-29T18:08:39.329383Z",
  "name": "Blue Team HTB Incident Response - 2025-11-29",
  "project_type": "e8f40d49-26ec-455d-9e7e-369aa3a564e4",
  "language": "en-US",
  "tags": [
    "reptor",
    "blue-team",
    "incident-response"
  ],
  "readonly": false,
  "source": "created",
  "copy_of": null,
  "override_finding_order": false,
  "members": [
    {
      "id": "cbba05a5-e445-4477-9172-7894af87af17",
      "username": "stanner",
      "name": "",
      "color": "#5ce86f",
      "title_before": null,
      "first_name": "",
      "middle_name": null,
      "last_name": "",
      "title_after": null,
      "is_active": true,
      "roles": [
        "pentester"
      ]
    }
  ],
  "imported_members": [],
  "details": "http://127.0.0.1:8000/api/v1/pentestprojects/5078d82a-0ad2-4d09-a76e-950c1e8845dd",
  "notes": "http://127.0.0.1:8000/api/v1/pentestprojects/5078d82a-0ad2-4d09-a76e-950c1e8845dd/notes",
  "images": "http://127.0.0.1:8000/api/v1/pentestprojects/5078d82a-0ad2-4d09-a76e-950c1e8845dd/images",
  "findings": [
    {
      "id": "507ddbf3-6d48-4386-9b70-75d095748e63",
      "created": "2025-11-29T18:10:00.234211Z",
      "updated": "2025-11-29T18:10:00.235592Z",
      "project": "5078d82a-0ad2-4d09-a76e-950c1e8845dd",
      "project_type": "e8f40d49-26ec-455d-9e7e-369aa3a564e4",
      "language": "en-US",
      "template": null,
      "assignee": {
        "id": "cbba05a5-e445-4477-9172-7894af87af17",
        "username": "stanner",
        "name": "",
        "color": "#5ce86f",
        "title_before": null,
        "first_name": "",
        "middle_name": null,
        "last_name": "",
        "title_after": null,
        "is_active": true
      },
      "status": "in-progress",
      "order": 1,
      "data": {
        "title": "Unauthorized Access Attempt Detected",
        "incident_id": "TODO TO BE FILLED BY THE SECURITY ANALYST",
        "incident_severity": "TODO: TO BE FILLED BY THE SECURITY ANALYST",
        "incident_status": null,
        "incident_overview": "TODO TO BE FILLED BY THE SECURITY ANALYST",
        "key_findings": "TODO TO BE FILLED BY THE SECURITY ANALYST",
        "immediate_actions": "TODO TO BE FILLED BY THE SECURITY ANALYST",
        "stakeholder_impact": "TODO TO BE FILLED BY THE SECURITY ANALYST",
        "affected_systems": "Highlight all systems and data that were either potentially accessed or definitively compromised during the incident. If data was exfiltrated, specify the volume or quantity, if ascertainable.\n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n",
        "evidence_sources": "Emphasize the evidence scrutinized, the results, and the analytical methodology employed. Each detection should be elucidated step by step, inclusive of the associated data sources, SIEM queries, and tool commands.\n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n",
        "ioc": "IoCs are instrumental for hunting potential compromises across our broader environment or even among partner organizations. These can range from abnormal outbound traffic to unfamiliar processes and scheduled tasks initiated by the attacker.\n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n",
        "root_cause": "Within this section, detail the root cause analysis conducted and elaborate on the underlying cause of the security incident (vulnerabilities exploited, failure points, etc.).\n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n",
        "timeline": "This is a pivotal component for comprehending the incident's sequence of events. The timeline should include:\n* Reconnaissance\n* Initial Compromise\n* C2 Communications\n* Enumeration\n* Lateral Movement\n* Data Access & Exfiltration\n* Malware Deployment or Activity (including Process Injection and Persistence)\n* Containment Times (can be excluded)\n* Eradication Times (can be excluded)\n* Recovery Times (can be excluded)\n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n",
        "nature": "Deep-dive into the type of attack, as well as the tactics, techniques, and procedures (TTPs) employed by the attacker. \n\nTODO TO BE FILLED BY THE SECURITY ANALYST\n"
      }
    }
  ]
}
```

**Note**: This is a partial example showing the project structure. The actual finding data follows below.

---

## Complete SSH Brute Force Attack Example

This is a comprehensive example of a complete incident finding with all required fields properly populated:

```json
{
  "status": "in-progress",
  "data": {
    "title": "SSH Brute Force Attack - Multiple Failed Authentication Attempts",
    "incident_id": "INC-2025-11-29-001",
    "incident_severity": "High",
    "incident_status": "contained",
    "incident_overview": "Detection of coordinated SSH brute-force attack targeting administrative accounts from external IP address 192.168.1.100. Attack was detected through SIEM correlation rules and automated alerting.",
    "key_findings": "- 247 failed SSH authentication attempts in 15-minute window\n- Targeting admin, root, and sysadmin accounts\n- Attack originated from single IP: 192.168.1.100\n- Attack stopped after automatic IP blocking triggered\n- No successful authentication detected",
    "immediate_actions": "1. Blocked source IP 192.168.1.100 at firewall level\n2. Reviewed all SSH access logs for successful logins\n3. Verified integrity of targeted systems\n4. Notified security team and system administrators\n5. Initiated threat hunting for lateral movement indicators",
    "stakeholder_impact": "Minimal - Attack was blocked before successful compromise. Brief service disruption for legitimate users during automated blocking response. No data breach or system compromise occurred.",
    "affected_systems": "**Primary Targets:**\n- SSH Server: 10.0.1.50:22 (production web server)\n\n**Targeted Accounts:**\n- root\n- admin  \n- sysadmin\n\n**Current Status:** All systems verified clean, no compromise detected",
    "evidence_sources": "**SIEM Query Results:**\n```\nsource_ip=\"192.168.1.100\" AND event_type=\"ssh_failed_auth\" \n| stats count by dest_ip, user, _time\n```\nResults: 247 events between 14:22:15 - 14:37:43 UTC\n\n**SSH Log Analysis:**\n```bash\ngrep \"Failed password\" /var/log/auth.log | grep \"192.168.1.100\"\n```\n\n**IDS Alert:** Suricata rule 2001219 - SSH Brute Force Attempt\n\n**Firewall Logs:** Automated block triggered at 14:38:02 UTC",
    "ioc": "**Network Indicators:**\n- IP Address: 192.168.1.100\n- User-Agent: libssh-0.7.0 (SSH client library)\n- Connection attempts every 3-5 seconds\n\n**Behavioral Indicators:**\n- Sequential username enumeration pattern\n- Common password dictionary attack\n- Persistent reconnection attempts\n\n**MISP Export:** IoCs exported to MISP event #2025-1129-001",
    "root_cause": "**Vulnerability Analysis:**\n- SSH port (22) exposed to internet without IP whitelisting\n- Password-based authentication enabled\n- No MFA/2FA requirement for administrative accounts\n- Fail2ban configured but threshold too high (50 attempts vs observed 247)\n\n**Attack Vector:** Direct internet exposure of SSH service with weak authentication controls allowed attacker to conduct brute-force attack.",
    "timeline": "**Reconnaissance Phase:**\n- 14:20:00 UTC - Initial port scan detected from 192.168.1.100\n- 14:21:30 UTC - Banner grabbing attempts on SSH port\n\n**Initial Compromise Attempt:**\n- 14:22:15 UTC - First failed authentication attempt (user: root)\n- 14:22:18 UTC - Second attempt (user: admin)\n\n**Attack Execution:**\n- 14:22:15 - 14:37:43 UTC - 247 failed authentication attempts\n- Attack rate: ~16 attempts per minute\n- Username rotation observed: root → admin → sysadmin → root\n\n**Detection & Response:**\n- 14:37:50 UTC - SIEM correlation rule triggered alert\n- 14:38:02 UTC - Automated firewall block activated\n- 14:40:15 UTC - Security analyst notification sent\n- 14:45:00 UTC - Investigation initiated\n\n**Containment:**\n- 14:50:00 UTC - Verified no successful logins\n- 15:00:00 UTC - Threat hunting completed, no lateral movement\n\n**Eradication:**\n- 15:30:00 UTC - Permanent IP block applied\n- 16:00:00 UTC - Fail2ban thresholds adjusted\n\n**Recovery:**\n- 16:30:00 UTC - Systems verified operational\n- 17:00:00 UTC - Incident report completed",
    "nature": "**Attack Type:** SSH Brute Force Attack (MITRE ATT&CK: T1110.001 - Brute Force: Password Guessing)\n\n**Tactics, Techniques, and Procedures (TTPs):**\n\n**Tactic:** Initial Access (TA0001)\n- **Technique:** T1110.001 - Brute Force: Password Guessing\n- **Procedure:** Automated dictionary attack against SSH service using common administrative usernames\n\n**Attack Characteristics:**\n- Unsophisticated attack using common credential pairs\n- No advanced evasion techniques observed\n- Linear attack pattern suggesting automated tool (likely Hydra or Medusa)\n- No attempt to modify logs or hide activity\n\n**Threat Actor Assessment:**\n- Likely opportunistic attacker scanning for exposed SSH services\n- Low sophistication level\n- No indicators of targeted attack or APT activity\n- Similar attack patterns observed across internet (CVE scanning campaigns)"
  }
}
```

---

## Advanced: Submitting to Different Projects

You can submit findings to a specific project using the `-p` flag:

```bash
cat finding.json | reptor finding -p "ad7acc23-d007-4a73-afcd-f4029899b4e2"
```

This is useful when managing multiple concurrent investigations.

---

## Integration with CTF Mode Skill

This skill is designed to work seamlessly with the **CTF Mode** skill:

1. **CTF Mode** performs rapid analysis and identification of flags/IOCs
2. **CTF Mode** passes findings to this skill
3. **This skill** creates structured, professional reports for CTF submissions

This integration allows you to:
- Win CTF competitions with rapid analysis
- Document findings in real-time
- Generate professional reports for CTF write-ups
- Track all IOCs and techniques used

---

## When to Use This Skill

✅ **Use this skill when**:
- You've completed analysis and need to document findings
- You have IOCs, timelines, and impact assessments ready
- You need to generate professional incident reports
- You're working with CTF Mode and need to create reports

❌ **Don't use this skill when**:
- You need to perform hands-on analysis (use analyze-data skill)
- You're still investigating and haven't identified findings yet
- You need CTF-specific rapid assessment (use ctf-mode skill first) 