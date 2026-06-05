# Upstream PR Plan

The feature set is easier to review if it is submitted as a small stack instead
of one broad pull request.

## 1. Tuya Cloud Metadata

Scope:

- Refresh expired Tuya Cloud tokens without signing token requests with the old
  token.
- Fetch bulk device details.
- Fetch device specifications, shadow properties, and thing model data.
- Store partial metadata errors per device so import can continue when one
  endpoint is unavailable.

Tests:

- Token refresh and retry handling.
- Metadata enrichment with model, shadow, and partial permission failures.

## 2. Auto-Import And Sync

Scope:

- Infer LocalTuya entity config from Tuya Cloud metadata and local DPS data.
- Add an options-flow action for auto-import.
- Run startup/hourly sync to import new supported cloud devices and update
  cloud-imported devices.
- Avoid deleting devices during normal background sync.

Tests:

- Auto-mapping for lights, plugs, heaters, covers, sensors, binary sensors, and
  unsupported devices.
- Sync behavior for new, existing, renamed, manually edited, and removed
  devices.

## 3. Runtime Resilience

Scope:

- Reconnect with bounded exponential backoff.
- Recover stale IP addresses using discovery, ARP/MAC correlation, and local-key
  LAN probes.
- Apply runtime-safe config changes without requiring a full Home Assistant
  restart.
- Throttle repeated identical connection warnings.

Tests:

- Backoff scheduling.
- IP recovery from MAC and local-key probing.
- Warning throttling and recovery reset.

## 4. Device Capability Improvements

Scope:

- Improve light color/color-temperature handling from Tuya ranges.
- Add safer climate/heater mapping from model enum values.
- Map cover direction reversal from Tuya metadata.
- Preserve useful read-only datapoints such as countdown timers.

Tests:

- Color temperature scaling.
- Cover direction inversion.
- Climate/fan enum range handling through auto-config tests.
