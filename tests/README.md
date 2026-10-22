# SOC Validation and Testing

This document records validation procedures and test results for the Raspberry Pi SOC monitoring pipeline.

The goal is to verify that security events are generated, collected by Grafana Alloy, forwarded to Loki, and available for investigation through Grafana.

## Environment

- **Host:** Raspberry Pi 5
- **Operating system:** Debian 13 (Trixie)
- **Network detection:** Suricata 7.0.10
- **Log collection:** Grafana Alloy
- **Log storage:** Grafana Loki
- **Visualization:** Grafana
- **Deployment:** Docker Compose

## Validation Results — October 22, 2026

### Test 1: Suricata Log Generation

**Result: PASS**

Recent JSON-formatted flow events were observed in the Suricata EVE JSON log.

**Log source:**

`/var/log/suricata/eve.json`

This confirmed that Suricata was generating network telemetry during validation.

### Test 2: SOC Container Status

**Result: PASS**

The following SOC containers were running during validation:

- `soc-grafana`
- `soc-alloy`
- `soc-loki`

The containers had been running for approximately three days at the time of the check.

### Test 3: Alloy Service Logs

**Result: PASS — basic health check**

The Alloy container was running, and the supplied recent log output contained routine informational messages without obvious errors.

This check alone does not prove that every log entry was forwarded successfully.

### Test 4: Loki Ingestion and Grafana Querying

**Result: PASS**

Both of the following LogQL queries returned results in Grafana Explore using the configured Loki data source.

**Suricata events:**

```logql
{job="suricata"}
```

**System journal entries:**

```logql
{job="ssh"}
```

These results confirmed that both labeled log streams were available for querying through Grafana.

The `job="ssh"` label is assigned to the systemd journal source in Alloy. It does not guarantee that the stream contains only SSH authentication events.

### Validation Summary

The validation confirmed:

- Suricata was generating recent EVE JSON events.
- The core SOC containers were running.
- Alloy's supplied logs contained no obvious errors.
- Suricata and journal streams returned results through Grafana Explore.

The successful queries provide evidence that log ingestion was working for the tested streams.

## Repeatable Validation Procedures

### Test 5: Check Suricata Log Generation

**Objective:** Confirm Suricata is producing EVE JSON events.

Run:

```bash
sudo tail -n 5 /var/log/suricata/eve.json
```

**Expected result:** Recent JSON-formatted events are present.

### Test 6: Inspect Alloy Logs

**Objective:** Check for obvious log collection or forwarding errors.

Run:

```bash
docker logs --tail 50 soc-alloy 2>&1
```

**Expected result:** No relevant errors are present in the inspected output.

### Test 7: Query Loki

**Objective:** Confirm that Suricata events are queryable.

In Grafana Explore, select the Loki data source and run:

```logql
{job="suricata"}
```

**Expected result:** Log entries appear for the selected time range when data is available.

### Test 8: Filter Suricata Alert Events

**Objective:** Search for entries containing Suricata alert event types.

Run:

```logql
{job="suricata"} |= "\"event_type\":\"alert\""
```

**Expected result:** Matching entries appear if alert events exist in the selected time range and use the expected JSON representation.

This query is a text filter, not a structured JSON-field parser.

### Test 9: Controlled End-to-End Test

**Objective:** Trace an authorized test event from its source to Loki.

Procedure:

1. Generate a benign test event on a system you own or are authorized to test.
2. Confirm the event appears in the appropriate source log.
3. Query the relevant stream in Grafana Explore.
4. Confirm the event can be identified in Loki.
5. Record the timestamp, relevant fields, and outcome.
6. Redact sensitive details before publishing evidence.

**Status:** Planned — results have not yet been documented.

## Evidence Log

For future tests, record:

| Field | Details |
|---|---|
| Test date | YYYY-MM-DD |
| Test performed | Name of validation test |
| Result | Pass / Fail |
| Evidence | Sanitized screenshot or command output |
| Findings | Relevant observations |
| Follow-up | Required fixes or additional testing |

## Security Considerations

- Test only on systems you own or are authorized to assess.
- Never publish passwords, API keys, authentication tokens, or private keys.
- Redact sensitive IP addresses and device identifiers from public evidence where appropriate.
- Review screenshots and log excerpts before committing them.
- Distinguish verified results from expected outcomes and planned tests.

## Future Improvements

- Validate structured JSON parsing and field extraction.
- Build security-focused Grafana dashboards.
- Develop reusable LogQL queries for investigations.
- Configure and validate meaningful security alerting.
- Integrate selected host authentication and system logs.
- Test recovery after service restarts.
- Document controlled detection exercises and investigation findings.
