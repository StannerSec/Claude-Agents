# CTF Mode Skill

## Overview
This skill activates specialized CTF (Capture The Flag) competition mode, optimized for rapid blue team analysis, flag discovery, and competitive incident response challenges. Use this skill when participating in CTF competitions, security assessments, or time-sensitive test scenarios.

## Purpose
Transform into a CTF-optimized analyst that excels at:
- **Rapid evidence analysis** under time constraints
- **Flag identification and extraction** from forensic artifacts
- **Pattern recognition** for CTF-specific challenges
- **Comprehensive documentation** for scoring and write-ups
- **Strategic prioritization** of high-value targets

---

## CTF Mindset and Strategy

### Core Principles

1. **Time is Critical**: Every second counts in competitions
   - Prioritize quick wins and obvious flags first
   - Use automated tools to speed up analysis
   - Document as you go - don't wait until the end

2. **Think Like a Challenge Designer**:
   - CTF challenges often have deliberate clues
   - Look for patterns in filenames, timestamps, and metadata
   - Hidden flags are often in predictable locations (comments, metadata, steganography)
   - Challenge descriptions contain hints - read carefully

3. **Be Thorough But Efficient**:
   - Check common flag formats first (e.g., `HTB{...}`, `FLAG{...}`, `CTF{...}`)
   - Use grep/strings on everything
   - Don't go too deep on rabbit holes - if stuck for >15 minutes, move on

4. **Document Everything**:
   - Record all commands executed
   - Screenshot/save all flags found
   - Note the discovery method for each flag
   - Prepare for post-CTF write-up requirements

---

## CTF Analysis Workflow

### Phase 1: Reconnaissance (0-5 minutes)

**Objective**: Quickly understand what you're dealing with

```bash
# Identify file types
file * | tee file_identification.txt

# Get quick overview
ls -lah
tree -L 2  # If directory structure

# Check for obvious flags
grep -r "HTB{" .
grep -r "FLAG{" .
grep -r "CTF{" .
strings * | grep -E "(HTB|FLAG|CTF)\{"
```

**Questions to answer**:
- What types of evidence are provided? (pcap, disk image, memory dump, logs?)
- Are there any obvious patterns or hints?
- What's the flag format for this CTF?
- What's the scenario description telling me?

---

### Phase 2: Rapid Analysis (5-30 minutes)

**Objective**: Use the **analyze-data skill** with CTF optimization

#### For Network Traffic (PCAP files):
```bash
# Quick traffic overview
tcpdump -r capture.pcap -n | head -50

# Extract HTTP traffic
tshark -r capture.pcap --export-objects http,./http_objects/

# Look for credentials in clear text
tshark -r capture.pcap -Y "http.request or http.response" -T fields -e http.request.uri -e http.file_data | grep -i "password\|user\|flag"

# DNS exfiltration check (common in CTFs)
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | sort -u

# Extract files from traffic
foremost -i capture.pcap -o extracted_files/
binwalk -e capture.pcap
```

#### For Disk Images:
```bash
# Mount if possible
mkdir /mnt/evidence
mount -o ro,loop disk.img /mnt/evidence

# File system analysis
fls -r disk.img | tee file_list.txt

# Search for flags in deleted files
strings disk.img | grep -E "(HTB|FLAG|CTF)\{"

# Check browser history, bash history
grep -r "history" /mnt/evidence/
cat /mnt/evidence/home/*/.bash_history
cat /mnt/evidence/home/*/.zsh_history

# Look for hidden files
find /mnt/evidence -name ".*" -type f
```

#### For Memory Dumps:
```bash
# Identify OS profile
volatility3 -f memory.dmp windows.info
# or
volatility -f memory.dmp imageinfo

# Look for processes with flags in command line
volatility3 -f memory.dmp windows.cmdline | grep -i flag

# Dump clipboard (common flag location)
volatility3 -f memory.dmp windows.clipboard

# Check for suspicious processes
volatility3 -f memory.dmp windows.pslist
volatility3 -f memory.dmp windows.pstree

# Dump interesting processes
volatility3 -f memory.dmp windows.dumpfiles --pid <PID>
```

#### For Log Files:
```bash
# Quick scan for anomalies
cat *.log | grep -i "error\|fail\|attack\|suspicious"

# Timeline of events
cat *.log | grep "2025-11-29" | sort

# Look for encoded data (base64 common in CTFs)
grep -E "[A-Za-z0-9+/]{20,}={0,2}" *.log | base64 -d

# Check for exfiltration
grep -i "POST\|upload\|download" *.log
```

---

### Phase 3: Deep Dive Techniques

#### Steganography (Very Common in CTFs):
```bash
# Check image metadata
exiftool image.jpg | grep -i flag

# Extract hidden data from images
steghide extract -sf image.jpg
binwalk -e image.jpg
zsteg image.png --all  # For PNG files
stegseek image.jpg wordlist.txt  # Brute force steghide

# LSB analysis
stegsolve image.jpg  # GUI tool

# Strings in images
strings image.jpg | grep -E "(HTB|FLAG|CTF)\{"
```

#### Encoding/Encryption Detection:
```bash
# Base64
echo "SGVsbG8gV29ybGQ=" | base64 -d

# Hex
echo "48656c6c6f" | xxd -r -p

# ROT13
echo "Uryyb Jbeyq" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# CyberChef is your friend for multi-layer encoding
# https://gchq.github.io/CyberChef/
```

#### Registry Analysis (Windows):
```bash
# Extract registry hives
volatility3 -f memory.dmp windows.registry.hivelist

# Dump specific registry keys
volatility3 -f memory.dmp windows.registry.printkey --key "Software\\Microsoft\\Windows\\CurrentVersion\\Run"

# Look for persistence mechanisms
reglookup system_hive | grep -i "run\|service"
```

---

### Phase 4: Flag Extraction

**Common flag locations in CTFs**:
- File metadata (EXIF data, file comments)
- Steganography in images/audio
- Encoded in DNS queries
- Hidden in HTTP headers or POST data
- Embedded in process memory
- Base64/Hex encoded in logs
- ROT13 or simple ciphers
- ZIP file comments
- Hidden partitions or alternate data streams
- Network traffic payloads

**Flag validation checklist**:
- [ ] Matches the specified format
- [ ] Complete flag (not truncated)
- [ ] Screenshot/copy saved
- [ ] Discovery method documented
- [ ] Submitted to platform

---

### Phase 5: Documentation & Reporting

**Use the create-report-findings skill** to generate professional CTF write-ups:

```json
{
  "status": "completed",
  "data": {
    "title": "CTF Challenge: Memory Forensics - Flag Extraction",
    "incident_id": "CTF-2025-11-29-MEMORY-01",
    "incident_severity": "N/A (CTF Challenge)",
    "incident_status": "completed",
    "incident_overview": "Memory forensics challenge requiring extraction of flags from Windows memory dump",
    "key_findings": "- Flag found in process memory of suspicious PowerShell process\n- Encoded flag discovered in clipboard data\n- Hidden persistence mechanism in registry",
    "immediate_actions": "1. Identified memory profile using volatility\n2. Analyzed process tree\n3. Extracted clipboard contents\n4. Dumped suspicious process memory",
    "evidence_sources": "**Commands Executed:**\n```bash\nvolatility3 -f memory.dmp windows.info\nvolatility3 -f memory.dmp windows.pstree\nvolatility3 -f memory.dmp windows.clipboard\nvolatility3 -f memory.dmp windows.cmdline\n```",
    "ioc": "**Flags Discovered:**\n- Primary Flag: HTB{m3m0ry_f0r3ns1cs_r0cks}\n- Location: Process PID 2344 (powershell.exe) command line\n- Encoding: Base64\n\n**Discovery Method:**\n1. Listed all processes\n2. Identified suspicious PowerShell with encoded command\n3. Decoded base64 string to reveal flag",
    "timeline": "**00:00 - 00:05**: Initial triage, identified Windows 10 memory dump\n**00:05 - 00:15**: Process analysis, found suspicious PowerShell\n**00:15 - 00:20**: Command line extraction revealed base64 string\n**00:20 - 00:22**: Decoded flag and validated format\n**00:22 - 00:25**: Submitted flag successfully",
    "nature": "**Challenge Type**: Memory Forensics\n**Difficulty**: Medium\n**Key Techniques Used:**\n- Volatility 3 Framework\n- Process tree analysis\n- Command line argument extraction\n- Base64 decoding\n\n**MITRE ATT&CK Mapping** (if applicable):\n- T1059.001 - PowerShell execution\n- T1027 - Obfuscated Files or Information"
  }
}
```

---

## CTF-Specific Tips and Tricks

### 1. **Automation is Key**
Create quick scripts for common tasks:

```bash
#!/bin/bash
# CTF Quick Scanner

echo "[+] Searching for flags..."
grep -r "HTB{" . 2>/dev/null
grep -r "FLAG{" . 2>/dev/null
grep -r "CTF{" . 2>/dev/null

echo "[+] Checking strings..."
strings * | grep -E "(HTB|FLAG|CTF)\{[^}]+\}" 2>/dev/null

echo "[+] Checking metadata..."
exiftool * 2>/dev/null | grep -i flag

echo "[+] Checking for encoded data..."
grep -rE "[A-Za-z0-9+/]{40,}={0,2}" . 2>/dev/null | head -20
```

### 2. **Common CTF Tools Checklist**
Ensure these are installed:
- **Network**: Wireshark, tshark, tcpdump, ngrep, zeek
- **Disk**: sleuthkit, autopsy, binwalk, foremost
- **Memory**: volatility2, volatility3, rekall
- **Steganography**: steghide, stegsolve, zsteg, stegseek
- **Forensics**: exiftool, strings, file, hashcat
- **General**: CyberChef, grep, awk, sed, jq

### 3. **Time Management**
- **Easy challenges first** (20-30 min each)
- **Medium challenges** if time permits (45-60 min)
- **Hard challenges** only if leading or final stretch
- Leave 30 minutes for write-ups and documentation

### 4. **Team Communication** (if team-based)
- Share findings immediately
- Document dead ends to avoid duplication
- Divide challenges by expertise
- Use shared notes (HackMD, Notion, etc.)

---

## Integration with Other Skills

### Using Analyze-Data Skill
CTF Mode **internally leverages** the analyze-data skill but adds:
- Time pressure optimization
- Flag-focused analysis
- CTF-specific pattern recognition
- Rapid pivoting strategies

**When to explicitly use analyze-data**:
- Need deep technical analysis beyond flag hunting
- Comprehensive forensic investigation required
- Standard incident response methodology needed

### Using Create-Report-Findings Skill
**Always conclude CTF challenges** with this skill to:
- Document the solution method
- Create write-ups for points/reputation
- Build your portfolio
- Share knowledge with team/community

---

## When to Use This Skill

✅ **Use CTF Mode when**:
- Participating in CTF competitions (HTB, TryHackMe, SANS, etc.)
- Time-sensitive security assessments
- Practice challenges with scoring
- Rapid triage scenarios
- Test-taking environments

❌ **Don't use CTF Mode when**:
- Conducting real-world incident response (use analyze-data)
- Need comprehensive documentation first (not rapid analysis)
- No time pressure or scoring involved
- Learning/educational scenarios where speed isn't critical

---

## Success Metrics

**Track your performance**:
- Time to first flag
- Total flags captured
- Write-up quality
- Tools/techniques learned
- Ranking/points achieved

**Post-CTF Review**:
- What worked well?
- What tools were most useful?
- What knowledge gaps were identified?
- What can be automated next time?

**Remember**: The goal is not just to win, but to learn and improve your blue team skills through practical application!