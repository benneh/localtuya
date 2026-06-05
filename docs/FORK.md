# Better LocalTuya Fork

This fork is a drop-in replacement for `rospogrigio/localtuya` that keeps the
same Home Assistant integration domain: `localtuya`.

## Goals

- Preserve existing LocalTuya config entries and entity IDs where possible.
- Use Tuya Cloud metadata to infer common device capabilities automatically.
- Import newly linked Tuya Cloud devices during normal sync.
- Improve runtime recovery when devices change IP address or briefly disappear
  from the LAN.
- Keep locally unavailable or cloud-event-only devices honest: imported entities
  should report unavailable rather than faking stale cloud values.

## Installation Notes

Do not install this fork side by side with upstream LocalTuya. Because the
integration domain remains `localtuya`, deploy this fork over the existing
`/config/custom_components/localtuya` directory.

Before installing:

1. Back up `/config/custom_components/localtuya`.
2. Disable HACS auto-updates for upstream LocalTuya while testing this fork.
3. Restart Home Assistant after replacing the custom component.

Cloud credentials are still configured through the normal LocalTuya options
flow. They are required for auto-import and richer capability detection.

## Sync Behavior

The fork runs a cloud sync during startup and then hourly. The sync imports new
supported devices, updates cloud-imported device names/capabilities, and keeps
manually edited devices conservative. Devices removed from Tuya Cloud are not
deleted automatically during background sync.

Offline devices may still be imported if cloud metadata contains enough
information. Local control remains unavailable until the device can be reached
on the LAN.

## Upstream Compatibility

This fork intentionally keeps the `localtuya` domain and config entry shape as
close to upstream as practical. That makes it easier to move between upstream
LocalTuya and this fork, but users should always keep a Home Assistant backup
before switching custom component code.
