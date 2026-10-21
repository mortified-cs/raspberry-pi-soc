# SOC LogQL Query Library

A collection of LogQL queries for exploring and investigating logs collected by the Raspberry Pi SOC.

The current Alloy configuration forwards Suricata EVE JSON logs and system journal entries to Grafana Loki.

## Prerequisites

- Grafana connected to the Loki data source.
- Alloy forwarding logs to Loki.
- Log streams labeled with `job="suricata"` and `job="ssh"`.

The queries below are starting points. Validate their results against the events actually present in your environment.

## 1. Suricata Logs

### View all Suricata events

```logql
{job="suricata"}
```

Returns log entries from the Suricata stream.

### Search for alert events

```logql
{job="suricata"} |= "\"event_type\":\"alert\""
```

Filters for entries containing the JSON field/value text associated with Suricata alert events. This depends on the JSON formatting in the stored log lines.

### Search for DNS events

```logql
{job="suricata"} |= "\"event_type\":\"dns\""
```

Filters for entries containing the JSON event type `dns`.

### Search for flow events

```logql
{job="suricata"} |= "\"event_type\":\"flow\""
```

Filters for entries containing the JSON event type `flow`.

## 2. System Journal Logs

### View the journal stream

```logql
{job="ssh"}
```

Returns entries collected by the Alloy systemd journal source.

**Note:** The `job="ssh"` label is assigned in Alloy. It does not independently filter the journal to SSH-only events.

### Search for SSH-related messages

```logql
{job="ssh"} |= "sshd"
```

Filters for journal lines containing `sshd`. Confirm the actual message format in your environment.

### Search for authentication failures

```logql
{job="ssh"} |= "Failed password"
```

Searches for a common SSH authentication-failure message. Results depend on which journal entries Alloy collects.

## 3. Investigation Workflow

1. Select an appropriate time range in Grafana Explore.
2. Start with `{job="suricata"}` or `{job="ssh"}`.
3. Narrow the results with a text filter.
4. Inspect surrounding events and timestamps.
5. Record the relevant evidence and findings without exposing sensitive network details.

## 4. Future Query Improvements

- Parse EVE JSON fields with LogQL.
- Filter alerts by signature and severity.
- Build event-count queries for dashboard panels.
- Create validated queries for repeated authentication failures.
- Develop alerting rules based on verified event patterns.

## Security Notes

Do not publish unredacted log output, public IP addresses, internal device identifiers, or personal information. Review screenshots and examples before committing them to a public repository.
