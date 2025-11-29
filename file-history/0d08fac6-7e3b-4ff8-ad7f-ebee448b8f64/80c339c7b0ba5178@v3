# Generate Detection Logic Skill

This skill creates Wazuh detection rules using threat intelligence research provided by the Threat Research Engineer.

## Detection Philosophy: The Pyramid of Pain

Build rules using the **Pyramid of Pain** approach to create resilient detections that are difficult for attackers to evade:

### Pyramid Levels (Bottom to Top):
1. **Hash Values** (Trivial) - Easy for attackers to change
2. **IP Addresses** (Easy) - Simple to rotate infrastructure
3. **Domain Names** (Simple) - Attackers can register new domains
4. **Network/Host Artifacts** (Annoying) - Requires some operational changes
5. **Tools** (Challenging) - Forces attackers to learn new tools
6. **TTPs (Tactics, Techniques, Procedures)** (Tough) - Requires fundamental changes to attack methodology

### Capability Abstraction

**Attack tools are abstractions of attack capabilities.** When building detection rules:
- Evaluate tool abstractions to understand what capabilities they provide attackers
- Build detection logic that targets the capability, not just the specific tool
- This approach helps avoid false negatives when attackers use alternative tools
- Understand telemetry needs to detect capabilities across different tool implementations
- Ensure coverage of the underlying technique, not just one tool manifestation

### Detection Strategy

**Goal**: Create a balanced mix of detection rules across the pyramid:
- **IOC-based rules** (bottom of pyramid) - Fast deployment, catches known threats
- **TTP-based rules** (top of pyramid) - Resilient detection, harder to evade

Nothing is wrong with IOC-based rules, but we want to also create rules that are at the top of the pyramid and are harder for attackers to evade without completely changing a technique.

### Detection Spectrum: Precision vs. Breadth

Detections range along a spectrum from "precise" to "broad":

- **Precise Detections**: Prioritize reducing false positives
  - Very specific indicators (exact hashes, known malicious IPs)
  - High confidence when triggered
  - May miss variants or evasion attempts (higher false negative risk)

- **Broad Detections**: Prioritize reducing false negatives
  - Behavioral patterns and capability-based detection
  - Catch more attack variants and evasion attempts
  - May generate more false positives requiring triage

**Best Practice**: Build both types for comprehensive coverage.

### Key Resources

Review these resources for detection strategies:
- **MITRE ATT&CK Detection Strategies**: https://attack.mitre.org/detectionstrategies/ - Detection analytics and attack chaining ideas
- **Pyramid of Pain**: https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html - Understanding detection effectiveness
- **Summiting the Pyramid**: https://center-for-threat-informed-defense.github.io/summiting-the-pyramid/ - In-depth guide to building great detection rules

## Rule Development Requirements

### 1. Create Detection Rules
Generate Wazuh XML rules that detect the threat across multiple pyramid levels when possible.

### 2. Document Required Log Sources
After building detections, **always document the required log sources** in Wazuh format so the user can easily configure them:

**Example Log Source Documentation:**
```yaml
Required Log Sources (Wazuh Configuration):
---
Windows Event Logs:
  - Security (EventID 4688 - Process Creation)
  - Sysmon (EventID 1 - Process Creation, EventID 3 - Network Connection)

Linux Logs:
  - auditd (execve syscalls)
  - syslog (/var/log/auth.log)

File Integrity Monitoring:
  - Monitor: C:\Windows\System32\drivers\etc\hosts
  - Monitor: /etc/passwd, /etc/shadow

Network Logs:
  - DNS queries
  - Firewall logs (source/destination IPs and ports)
```

### 3. Balance Detection Strategy
For each threat, create rules at multiple pyramid levels:
- **Low Pyramid** (IOCs): Hashes, IPs, domains - for quick wins
- **High Pyramid** (TTPs): Behavioral detection, command patterns, attack chains - for resilience

## Advanced Detection Concepts

### The Funnel of Fidelity

The detection and response process flows through multiple stages:

1. **Collection** - Gathering telemetry and log data from systems
2. **Detection** - Rules and analytics that identify potential threats
3. **Triage** - Initial assessment to filter false positives
4. **Investigation** - Deep analysis of confirmed threats
5. **Remediation** - Response actions to contain and eliminate threats

**⚠️ Avoid a "Clogged Funnel"**: Issues in one stage hinder success in subsequent stages. For example:
- Poor collection = No data to detect threats
- Too many false positives = Triage becomes overwhelmed
- Ineffective triage = Investigation teams waste time on noise

When designing rules, consider the entire funnel:
- Ensure required log sources are collected (Collection)
- Balance precision vs. breadth appropriately (Detection)
- Provide sufficient context for triage (Detection → Triage)
- Include investigation guidance in rule descriptions (Triage → Investigation)

### Detection-in-Depth

**Multiple detections for a single technique increase likelihood of detection.**

Implement detection at various "choke points" throughout the attack lifecycle:

**Example: Credential Dumping (T1003)**
- **Process Execution**: Detect mimikatz.exe process creation
- **Memory Access**: Detect unusual LSASS memory reads
- **Command Line**: Detect suspicious sekurlsa commands
- **File System**: Detect credential dump file creation
- **Network**: Detect credential material exfiltration

**Benefits**:
- If attacker evades one detection, others may still trigger
- Different detections provide different investigation contexts
- Comprehensive coverage across attack stages

### Robust Detection

A **robust detection** is both accurate and resistant to adversary evasion over time.

**Characteristics of Robust Detections**:
- Detect the capability or technique, not just specific tools
- Resilient to minor variations (obfuscation, encoding, alternative syntax)
- Based on behavioral patterns difficult for attackers to change
- Regularly validated against new attack variants
- Tuned to minimize both false positives and false negatives

**Building Robust Rules**:
- Use capability abstraction (detect what the attack does, not just how)
- Implement detection-in-depth (multiple rules per technique)
- Combine IOC-based and TTP-based approaches
- Test with synthetic logs including evasion variants
- Update rules as new TTPs emerge

## Output Format

When creating detection rules, provide:

1. **Detection Strategy Summary** - Which pyramid levels you're targeting
2. **Wazuh XML Rules** - Complete, production-ready rules
3. **Required Log Sources** - Wazuh configuration format for easy deployment
4. **MITRE ATT&CK Mapping** - Techniques and tactics detected
5. **Detection Rationale** - Why these rules are effective against the threat

---

## ⚠️ CRITICAL: The 5 Rules That Will Break Production

**READ THIS BEFORE WRITING ANY RULE XML.**

These are REAL production failures. Following these rules is mandatory.

### The 5 Critical Rules (Memorize These)

1. **✅ NO TRAILING COMMAS in `<group>` tags**
   - ❌ WRONG: `<group>pci_dss_11.4,gdpr_IV_35.7.d,nist_800_53_SI.4,tsc_CC7.2,</group>`
   - ✅ CORRECT: `<group>pci_dss_11.4,gdpr_IV_35.7.d,nist_800_53_SI.4,tsc_CC7.2</group>`
   - **Impact**: Trailing commas cause XML parse failures

2. **✅ Frequency rules MUST have `frequency` and `timeframe` attributes**
   - ❌ WRONG: `<rule id="110900" level="13"><if_matched_sid>110001,110002</if_matched_sid><same_source_ip/></rule>`
   - ✅ CORRECT: `<rule id="110900" level="13" frequency="2" timeframe="300"><if_matched_sid>110001</if_matched_sid><same_source_ip/></rule>`
   - **Impact**: Rules fail to load with error "Invalid use of frequency/context options"

3. **✅ Use `<if_matched_sid>` for correlation, `<if_sid>` for parent rule references**
   - ❌ WRONG: Mixing both in same rule: `<if_sid>110006</if_sid><if_matched_sid>110007</if_matched_sid>`
   - ✅ CORRECT: Use only one. For correlation: `<if_matched_sid>110001</if_matched_sid>` with `frequency` and `timeframe`
   - **Impact**: Syntax errors prevent rules from loading

4. **✅ ALWAYS add `type="pcre2"` to regex patterns**
   - ❌ WRONG: `<field name="win.eventdata.image">\\bcp\.exe$</field>`
   - ✅ CORRECT: `<field name="win.eventdata.image" type="pcre2">(?i)\\bcp\.exe$</field>`
   - **Impact**: Regex patterns don't work without the type attribute

5. **✅ TEST RULES with synthetic logs BEFORE declaring them "ready"**
   - Use the "Create Synthetic Logs" skill to validate every rule
   - **Impact**: Broken rules get deployed without testing

### Pre-Deployment Validation Checklist

Before declaring ANY rule ready:

- [ ] XML syntax is valid: `xmllint --noout rule_file.xml`
- [ ] NO trailing commas in `<group>` tags
- [ ] Frequency/correlation rules have `frequency="X"` and `timeframe="Y"`
- [ ] NOT mixing `<if_sid>` and `<if_matched_sid>`
- [ ] All regex patterns have `type="pcre2"`
- [ ] Tested with synthetic logs: 100% match rate 
