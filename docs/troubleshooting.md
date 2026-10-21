# Troubleshooting Notes

This document records implementation issues and configuration decisions encountered while building the Raspberry Pi SOC monitoring pipeline.

## 1. Alloy Suricata Log Collection

### Problem

The initial Alloy file-discovery approach did not provide the intended Suricata EVE JSON log collection behavior.

### Investigation

The Suricata event log was located at:

`/var/log/suricata/eve.json`

The Alloy configuration was reviewed to ensure that the log source explicitly targeted this file and forwarded its entries to Loki.

### Resolution

The final configuration uses `loki.source.file` with an explicit `__path__` target:

```alloy
loki.source.file "suricata" {
  targets = [
    {
      __path__ = "/var/log/suricata/eve.json",
      job      = "suricata",
    },
  ]

  forward_to = [loki.write.local.receiver]
}
```

This directly configures Alloy to read the Suricata EVE JSON file instead of relying on the earlier discovery approach.

### Loki Destination

Both configured sources forward entries to the local Loki writer:

```alloy
loki.write "local" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}
```

The destination uses the Docker service name `loki`, which requires the Alloy and Loki containers to have appropriate network connectivity and name resolution.

### Outcome

The Suricata-to-Alloy-to-Loki-to-Grafana pipeline is operational.

The explicit file target is part of the final working configuration.

## 2. System Journal Collection

The final Alloy configuration also includes a systemd journal source:

```alloy
loki.source.journal "ssh" {
  max_age    = "24h"
  forward_to = [loki.write.local.receiver]

  labels = {
    job = "ssh",
  }
}
```

This source collects journal entries and labels them with `job="ssh"`.

**Important:** The label does not itself filter entries to SSH events. Additional filtering or verification is required before treating this stream as exclusively SSH-related.

The `max_age = "24h"` setting controls the maximum age of entries considered when the source starts; it should not be interpreted as a guarantee that only SSH authentication events are collected.

## 3. Follow-up Validation

The following checks should be documented as they are performed:

- Confirm Suricata events appear in Loki with `job="suricata"`.
- Confirm journal entries appear with `job="ssh"`.
- Verify that the journal source is collecting the intended events.
- Restart the SOC stack and confirm log collection resumes.
- Record any errors and the corresponding verified fix.

## Lessons Learned

- Explicit log paths can make collection behavior easier to reason about than a discovery-based setup.
- Alloy source components need to forward entries to a correctly configured destination.
- Loki labels classify streams but do not automatically filter the underlying log content.
- Successful deployment should be confirmed by querying the destination, not just by checking whether containers are running.
