# Create Synthetic Logs Skill

This skill generates and validates synthetic logs to test Wazuh detection rules before deployment.

## Purpose

Before deploying detection rules to production, you must validate that they work correctly. This skill uses the `synthetic-log-generator` MCP server to:
1. Generate realistic synthetic logs that simulate threat activity
2. Validate that detection rules correctly trigger on those logs
3. Ensure 100% match rates before declaring rules production-ready

## Available MCP Tools

### 1. `generate_logs_from_threat_intel`
Generate synthetic logs directly from threat intelligence data.

**Use when:** You have threat intelligence data and need to create realistic test logs.

**Input:**
```json
{
  "threat_intel": {
    "name": "MalwareName",
    "description": "Description",
    "techniques": ["T1234", "T5678"],
    "iocs": {
      "filenames": ["malware.exe"],
      "paths": ["C:\\temp"],
      "registry_keys": ["HKLM\\Software\\..."]
    },
    "log_patterns": [
      {
        "rule_id": "100200",
        "type": "process_creation",
        "technique": "T1234",
        "fields": {
          "program": "sysmon",
          "win.eventdata.commandLine": "malware.exe -install"
        }
      }
    ]
  },
  "count": 5
}
```

**Output:** Array of synthetic logs matching threat behavior patterns.

### 2. `generate_logs_from_rule`
Generate synthetic logs that would match a specific Wazuh rule.

**Use when:** You've created a rule and need test data to validate it works correctly.

**Input:**
```json
{
  "rule_xml": "<rule id=\"100100\" level=\"10\"><match>Failed password</match><description>SSH brute force</description></rule>",
  "count": 5
}
```

**Output:** Array of synthetic logs with matching content.

### 3. `validate_logs_against_rule`
Test whether generated logs would trigger a Wazuh rule.

**Use when:** You need to verify rules correctly detect threat behaviors.

**Input:**
```json
{
  "logs": [
    {
      "expected_rule_id": "100100",
      "fields": {
        "program": "sshd",
        "message": "Failed password for invalid user admin"
      }
    }
  ],
  "rule_xml": "<rule id=\"100100\" level=\"10\"><match>Failed password</match></rule>"
}
```

**Output:** Validation results with match rate and per-log details.

### 4. `generate_custom_logs`
Generate synthetic logs with custom field values.

**Use when:** You want to create specific test scenarios with exact field values.

**Input:**
```json
{
  "fields": {
    "program": "sshd",
    "srcip": "203.0.113.100",
    "message": "Failed password for invalid user admin"
  },
  "rule_id": "100100",
  "count": 3
}
```

**Output:** Array of custom logs ready for testing.

## Mandatory Testing Workflow

### For Each Rule Created:

1. **Generate Test Logs**: Use `generate_logs_from_rule` with the rule XML (5-10 logs)
2. **Validate Rule**: Use `validate_logs_against_rule` to test the logs against the rule
3. **Check Match Rate**: Must achieve 100% match rate
4. **If Failed**:
   - Analyze mismatch
   - Refine rule XML
   - Regenerate logs
   - Revalidate
   - Repeat until 100% pass

**Do not move to the next rule until current rule achieves 100% match rate.**

## Output Requirements

**Keep outputs simple and clean:**
- Show **PASS** or **FAIL** status
- Show match rate percentage
- Do NOT show thousands of lines of log output
- Summarize results concisely

**Example Clean Output:**
```
Rule 110001 Validation:
  Logs Generated: 5
  Match Rate: 100%
  Status: ✓ PASS

Rule 110002 Validation:
  Logs Generated: 5
  Match Rate: 80%
  Status: ✗ FAIL
  Issue: Regex missing escape character
  Action: Refining rule and retesting...
```

## Testing Phases

### Phase 1: Individual Rule Testing
Test each rule independently with 5-10 synthetic logs.

### Phase 2: Batch Testing
Test all rules together with comprehensive synthetic log dataset to ensure no cross-contamination.

### Phase 3: Edge Case Testing
Use `generate_custom_logs` to test:
- Attack pattern variations
- Similar-but-legitimate activity (should NOT trigger)
- Obfuscation variants

## Exit Criteria

Before declaring rules production-ready:
- ✓ All individual rules achieve 100% match rate
- ✓ Batch testing shows no cross-contamination
- ✓ Edge cases validate correctly
- ✓ No false positives on legitimate activity
- ✓ All test results documented 

