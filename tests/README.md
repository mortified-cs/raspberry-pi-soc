# SOC Validation and Testing

This document tracks validation procedures for the Raspberry Pi SOC pipeline.

The goal is to verify that security events can be collected, forwarded to Loki, and investigated through Grafana.

## Environment

- Host: Raspberry Pi 5
- Operating system: Debian 13 (Trixie)
- Network detection: Suricata 7.0.10
- Log collection: Grafana Alloy
- Log storage: Grafana Loki
- Visualization: Grafana

## Test 1: Suricata Log Generation

**Objective:** Confirm Suricata is producing EVE JSON events.

**Log source:**

`/var/log/suricata/eve.json`

**Procedure:**

```bash
sudo tail -n 5 /var/log/suricata/eve.json
```

**Expected result:** Recent JSON-formatted events are present.

**Status:** Previously established as the configured event source. Repeat the check when collecting fresh evidence.

## Test 2: Alloy Log Forwarding

**Objective:** Confirm Alloy is configured to read the Suricata log and forward entries to Loki.

**Configuration:**

`/opt/soc/alloy/config.alloy`

**Procedure:**

```bash
docker logs --tail 50 soc-alloy
```

Review the output for relevant errors.

**Expected result:** Alloy runs without relevant collection or forwarding errors.

**Status:** Pending a documented validation run.

## Test 3: Loki Ingestion

**Objective:** Verify that Suricata entries are queryable in Loki.

**Procedure:**

1. Open Grafana.
2. Navigate to Explore.
3. Select the Loki data source.
4. Run:

```logql
{job="suricata"}
```

**Expected result:** Suricata log entries appear for a time range containing available events.

**Status:** Pending a documented validation run.

## Test 4: Event Filtering

**Objective:** Verify that LogQL filters can isolate specific event types.

**Query:**

```logql
{job="suricata"} |= "\"event_type\":\"alert\""
```

**Expected result:** Entries matching the filter appear when corresponding alert events exist in the selected time range.

**Status:** Pending a documented validation run.

## Test 5: Controlled End-to-End Test

**Objective:** Validate the complete pipeline using an authorized, controlled test event.

**Procedure:**

1. Generate a benign test event in an environment you own or are authorized to test.
2. Confirm the event is recorded by the appropriate source.
3. Search for the event in Loki through Grafana.
4. Record the timestamp, relevant event fields, and result.
5. Redact private network information before publishing evidence.

**Expected result:** The test event can be traced from its source log to the Loki query results.

**Status:** Planned.

## Evidence Log

Record each completed test using this template:

| Field | Details |
|---|---|
| Test date | YYYY-MM-DD |
| Test performed | Name of the validation test |
| Result | Pass / Fail |
| Evidence | Sanitized screenshot or command output |
| Findings | Relevant observations |
| Follow-up | Any fixes or additional testing required |

## Security Considerations

- Perform tests only on authorized systems.
- Do not publish credentials, tokens, or unredacted security logs.
- Redact sensitive IP addresses and device identifiers in public evidence.
- Distinguish verified results from expected outcomes and planned tests.
