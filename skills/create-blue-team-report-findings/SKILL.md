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
  --design "7765a3f4-6d8c-4773-8206-cf726538a961" \
  --tags blue-team,incident-response
```

**Note**: This command automatically updates `~/.sysreptor/config.yaml` with the new project ID.

#### Submit a Finding

```bash
cat finding.json | reptor finding
```

#### Update an Existing Finding

```bash
cat updated_finding.json | reptor finding --update <FINDING_ID>
```

## Reference Documentation

- [SysReptor Projects](https://docs.sysreptor.com/cli/projects-and-templates/project/)
- [SysReptor Findings](https://docs.sysreptor.com/cli/projects-and-templates/finding/)
- [SysReptor Configuration](https://docs.sysreptor.com/cli/configuration/)
- [Create Project Command](https://docs.sysreptor.com/cli/projects-and-templates/createproject/)


## Complete SSH Brute Force Attack Example

This is a comprehensive example of a single incident finding with all required fields properly populated:

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