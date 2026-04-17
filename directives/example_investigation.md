# Directive: Example Investigation

## Purpose

Given an event ID or identifier, pull all available data from relevant systems and produce a complete investigation report. The pattern: collect everything first, then synthesize.

## When to Use

When the user provides an ID and asks what happened. Common triggers:
- "Look up [ID]"
- "What happened on [event/transaction/call]?"
- "Investigate [ID]"
- Any message containing a UUID or numeric ID in the context of incident review

## Inputs

| Input | Source | Notes |
|-------|--------|-------|
| Event ID | User message | Extract from message. Accept any format - the script normalizes it. |
| `--format` | CLI arg | `json` (default) or `text` |

## Execution Policy

Act immediately when given a valid ID. This is read-only. No confirmation needed before running.

## Steps

### 1. Extract the ID

Pull the ID from the user's message. If multiple IDs are present, ask which one to investigate.

### 2. Run the primary investigation script

```bash
python execution/investigate_event.py <event_id>
```

This pulls the core record plus all linked entities. Takes 5-15 seconds depending on the number of linked records.

### 3. If the primary pull is incomplete, run supplementary scripts

```bash
# Pull detailed logs if the event has associated log entries
python execution/fetch_event_logs.py <event_id>

# Pull the audit trail if the event was modified after creation
python execution/fetch_audit_trail.py <event_id>
```

Only run supplementary scripts if the primary result indicates they're needed (e.g., `log_count > 0`).

### 4. Synthesize and present

Read all outputs and produce a structured summary:
- What happened (in plain English)
- Timeline of key events
- Who or what was involved
- Any anomalies (unexpected values, errors, gaps)
- Recommended next steps if action is required

Do not dump raw JSON at the user. Interpret it.

## Output Format

```
Investigation: <event_id>
Timestamp: <when it occurred>
Summary: <2-3 sentence plain English description>

Timeline:
  [HH:MM:SS] <event 1>
  [HH:MM:SS] <event 2>
  ...

Anomalies:
  - <anything unexpected>

Next steps:
  - <recommended action if applicable>
```

## Error Handling

**Event not found (404):** Tell the user the ID wasn't found. Ask if the ID might be in a different format or system.

**Partial data (some APIs returned errors):** Note which data is missing in the report. Do not present incomplete data as complete.

**ID is ambiguous (matches multiple records):** Show the user a list of matches and ask which one.

## Learnings

- IDs are case-insensitive. The scripts accept any case.
- Audit trail data is only retained for 90 days. Investigations older than that will have incomplete audit records.
- If `linked_entity_count` exceeds 50, cap the supplementary fetches at 50 to avoid timeout. Note the cap in the report.
