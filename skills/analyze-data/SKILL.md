# Blue Team Data Analysis Skill

## Overview
This skill enables hands-on forensic analysis using command-line tools to investigate security incidents, analyze artifacts, and extract actionable intelligence. Use this skill when you need to perform technical analysis of evidence.

## Core Analysis Capabilities

### 1. Network Traffic Analysis
**Tools**: tcpdump, tshark, Wireshark, zeek, ngrep
**Use cases**:
- Analyzing packet captures (pcap/pcapng files)
- Identifying command-and-control (C2) communications
- Detecting data exfiltration
- Extracting network-based IOCs (IPs, domains, URLs)
- Protocol analysis and decoding

**Example workflow**:
```bash
# Quick pcap overview
tcpdump -r capture.pcap -n | head -100

# Extract unique IPs
tcpdump -n -r capture.pcap | awk '{print $3}' | sort -u

# Filter for DNS queries
tshark -r capture.pcap -Y "dns.qry.name" -T fields -e dns.qry.name

# Export HTTP objects
tshark -r capture.pcap --export-objects http,./http_objects/

# Identify suspicious connections
zeek -r capture.pcap
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto | sort -u
```

### 2. Disk and File System Forensics
**Tools**: dd, sleuthkit (fls, icat, mmls), autopsy, binwalk, exiftool, strings
**Use cases**:
- Analyzing disk images
- Recovering deleted files
- Examining file metadata
- Identifying hidden or embedded files
- Timeline analysis

**Example workflow**:
```bash
# Examine partition structure
mmls disk.img

# List files in filesystem
fls -r -o 2048 disk.img

# Extract specific file
icat -o 2048 disk.img 12345 > extracted_file

# Search for embedded files
binwalk -e suspicious_file.bin

# Extract metadata
exiftool document.pdf

# Search for strings
strings -n 8 binary_file | grep -i "password\|key\|token"
```

### 3. Memory Forensics
**Tools**: volatility3, volatility2, strings, grep
**Use cases**:
- Analyzing memory dumps
- Identifying running processes and network connections
- Extracting credentials and keys
- Detecting malware and rootkits
- Analyzing process injection

**Example workflow**:
```bash
# Identify memory profile (Volatility 2)
volatility -f memory.dmp imageinfo

# List processes
volatility3 -f memory.dmp windows.pslist
volatility3 -f memory.dmp windows.pstree

# Network connections
volatility3 -f memory.dmp windows.netscan

# Command line arguments
volatility3 -f memory.dmp windows.cmdline

# Dump process
volatility3 -f memory.dmp windows.dumpfiles --pid 1234

# Detect injected code
volatility3 -f memory.dmp windows.malfind
```

### 4. Log Analysis
**Tools**: grep, awk, sed, jq, cut, sort, uniq, head, tail
**Use cases**:
- Parsing system logs (auth.log, syslog, Windows Event Logs)
- Identifying authentication failures
- Tracking user activities
- Detecting privilege escalation
- Timeline reconstruction

**Example workflow**:
```bash
# Failed SSH attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Successful logins by user
grep "Accepted" /var/log/auth.log | awk '{print $9}' | sort | uniq -c

# Parse JSON logs
cat app.log | jq 'select(.level=="ERROR") | {time, message}'

# Windows Event Log analysis (with evtx tools)
evtx_dump.py Security.evtx | grep -i "4625" # Failed logon events

# Timeline creation
cat *.log | grep "2025-11-29" | sort -k1,2
```

### 5. Malware Analysis (Basic Static Analysis)
**Tools**: strings, file, md5sum, sha256sum, objdump, readelf, pestr, pescanner
**Use cases**:
- Initial malware triage
- Extracting IOCs from binaries
- Identifying file types
- Checking for packing/obfuscation
- Hash generation for threat intelligence

**Example workflow**:
```bash
# File identification
file suspicious_binary
md5sum suspicious_binary
sha256sum suspicious_binary

# Extract readable strings
strings suspicious_binary | less
strings -e l suspicious_binary  # Unicode strings

# Check for packing
entropy suspicious_binary
upx -t suspicious_binary

# PE analysis (Windows binaries)
pestr suspicious.exe
pescanner.py suspicious.exe

# ELF analysis (Linux binaries)
readelf -h suspicious_elf
objdump -d suspicious_elf | less
```

### 6. Web Log Analysis
**Tools**: grep, awk, sed, apache-scalp, goaccess
**Use cases**:
- Detecting web attacks (SQLi, XSS, LFI/RFI)
- Identifying scanning activities
- Analyzing access patterns
- Extracting attacker IPs and user agents

**Example workflow**:
```bash
# Top IP addresses
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# SQLi attempts
grep -i "union.*select\|concat\|0x[0-9a-f]" access.log

# Directory traversal attempts
grep -E "\.\./|\.\.\\\\" access.log

# 404 errors (potential scanning)
awk '$9==404' access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# Suspicious user agents
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn
```

## Analysis Methodology

### Step 1: Initial Triage
- Identify the type of evidence (pcap, disk image, memory dump, logs)
- Verify file integrity (hash verification)
- Determine appropriate tools for analysis
- Create working directory for extracted artifacts

### Step 2: Data Collection
- Extract relevant data using appropriate tools
- Document all commands and their output
- Preserve chain of custody
- Take notes on observations

### Step 3: Deep Analysis
- Correlate findings across different data sources
- Identify patterns and anomalies
- Extract IOCs (IPs, domains, file hashes, process names)
- Build timeline of events

### Step 4: Documentation
- Record all findings with context
- Include command outputs as evidence
- Organize IOCs by category
- Prepare data for reporting (pass to create-report-findings skill)

## Best Practices

1. **Always work on copies**: Never analyze original evidence directly
2. **Document everything**: Record all commands, outputs, and observations
3. **Use multiple tools**: Cross-verify findings with different tools
4. **Think critically**: Distinguish between normal and malicious activity
5. **Preserve timestamps**: Use `-atime-preserve` and similar flags
6. **Check for anti-forensics**: Be aware of log clearing, timestomping, etc.
7. **Correlate across sources**: Network + disk + memory = complete picture

## Integration with Other Skills

- **After analysis is complete**: Use the `create-report-findings` skill to document your discoveries
- **For CTF scenarios**: The `ctf-mode` skill will invoke this skill with CTF-optimized thinking
- **Always conclude with reporting**: Your technical analysis should feed into structured reports

## Common Analysis Patterns

### Pattern 1: Suspicious Network Connection
```bash
# 1. Extract connections from pcap
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport | sort -u

# 2. Check for non-standard ports
tshark -r capture.pcap -Y "tcp.dstport > 1024" -T fields -e ip.dst -e tcp.dstport

# 3. Extract DNS queries to suspicious domains
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | grep -v -E "google|microsoft|cloudflare"
```

### Pattern 2: Compromised User Account
```bash
# 1. Find all login attempts
grep "sshd" /var/log/auth.log | grep "session opened"

# 2. Identify unusual login times
grep "user_name" /var/log/auth.log | awk '{print $1, $2, $3}'

# 3. Check command history
cat /home/user_name/.bash_history
```

### Pattern 3: Malicious Process
```bash
# 1. List suspicious processes (from memory)
volatility3 -f mem.dmp windows.pslist

# 2. Check network connections for process
volatility3 -f mem.dmp windows.netscan | grep "PID 1234"

# 3. Dump process executable
volatility3 -f mem.dmp windows.dumpfiles --pid 1234

# 4. Analyze dumped binary
strings dumped_process.exe | grep -i "http\|ftp\|download"
```

## When to Use This Skill

✅ **Use this skill when**:
- You have forensic evidence that needs command-line analysis
- You need to extract IOCs from technical artifacts
- You're performing hands-on incident investigation
- You need to analyze pcap files, disk images, memory dumps, or logs

❌ **Don't use this skill when**:
- You just need to create a report (use create-report-findings instead)
- You're in a CTF competition (use ctf-mode which will invoke this internally)
- You need high-level strategic advice without technical analysis 