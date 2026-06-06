# Better Local Tuya

Better Local Tuya is a Home Assistant custom integration for Tuya-based devices.
It uses Tuya Cloud metadata to discover devices and infer their capabilities,
then controls supported devices locally on your LAN for faster response, lower
cloud dependency, and more Home Assistant friendly customization.

This is a fork of
[`rospogrigio/localtuya`](https://github.com/rospogrigio/localtuya). It keeps
the same Home Assistant integration domain, `localtuya`, so existing LocalTuya
config entries can continue to load through this fork.

## Why This Fork Exists

Classic LocalTuya is powerful, but adding devices often requires manually
finding device IDs, local keys, datapoints, entity types, and device-specific
options. Better Local Tuya aims to make that low-touch or no-touch for common
devices:

1. Link your Tuya Cloud account as usual.
2. Let the integration read device identity, local keys, datapoints, enum
   ranges, shadow values, and thing model metadata from Tuya Cloud.
3. Auto-import supported devices into Home Assistant.
4. Operate those devices locally over the LAN whenever local Tuya control is
   available.

Tuya Cloud is used as a metadata and sync source. Day-to-day control is still
local, which keeps lights, switches, covers, heaters, and other supported
devices responsive even when Home Assistant automations or HomeKit bridges are
driving them.

## Highlights

- Automatic import for common Tuya devices.
- Startup and hourly sync with devices linked in Tuya Cloud.
- Automatic capability inference from:
  - bulk cloud device details
  - device specifications
  - shadow properties
  - `/v2.0/cloud/thing/{device_id}/model`
  - locally observed DPS values where available
- Local operation after import for supported LAN-reachable devices.
- Better RGB, color temperature, and brightness mapping for lights.
- Better switch and plug mapping, including energy datapoints when present.
- Heater/climate inference from Tuya enum ranges.
- Cover/blind mapping with support for reversed direction metadata.
- Read-only sensors and binary sensors for safe status datapoints.
- Conservative handling of cloud-event-only or locally unreachable devices:
  entities are unavailable rather than populated with fake stale values.
- Runtime recovery for LAN flaps and IP changes.
- Reconnect backoff and log throttling for wall-switched or offline devices.
- Existing LocalTuya config entries are preserved because the domain remains
  `localtuya`.

## Supported Device Types

Better Local Tuya inherits upstream LocalTuya platform support and adds improved
auto-detection for common devices, including:

- Lights, including RGB, RGBW, CCT, and dimmable bulbs where metadata supports it
- Switches and smart plugs
- Energy monitoring datapoints such as voltage, current, power, and energy
- Covers and smart blinds
- Heaters and other climate-like devices
- Sensors and binary sensors
- Fans and vacuums supported by upstream LocalTuya

Tuya device schemas vary widely. Unsupported or ambiguous devices are skipped
during auto-import rather than being configured incorrectly.

## Installation

Better Local Tuya is a drop-in replacement for LocalTuya. Do not install it side
by side with upstream LocalTuya.

Because the Home Assistant domain remains `localtuya`, the code must live at:

```text
/config/custom_components/localtuya
```

### Manual Install

1. Back up your current `/config/custom_components/localtuya` directory.
2. Download or clone this fork.
3. Copy `custom_components/localtuya` from this repository into
   `/config/custom_components/localtuya` on Home Assistant.
4. Restart Home Assistant Core.

### HACS Custom Repository

If using HACS, add this fork as a custom repository and install it as an
integration. Keep in mind that it still installs as `localtuya` internally.

After installing this fork, avoid updating upstream LocalTuya over the top of it
unless you intentionally want to return to upstream.

## Tuya Cloud Setup

Cloud credentials are strongly recommended. Without Tuya Cloud access, the fork
can still operate manually configured LocalTuya devices, but automatic import
and richer capability inference will be limited.

Use the normal LocalTuya cloud setup flow with:

- Tuya IoT region
- Client ID
- Client secret
- Tuya app user ID

These credentials allow Better Local Tuya to retrieve device metadata, local
keys, and capability descriptions. They are not used for normal device control
when a device is locally reachable.

## Auto-Import And Sync

Once cloud credentials are configured, Better Local Tuya can import devices from
Tuya Cloud automatically.

The sync behavior is intentionally conservative:

- New supported devices in Tuya Cloud are imported.
- Existing cloud-imported devices can receive updated names and capabilities.
- Existing manually configured devices are preserved where possible.
- Devices missing from Tuya Cloud are not automatically deleted during normal
  background sync.
- Offline devices can still be imported if cloud metadata is sufficient.
- Devices that cannot be mapped safely are skipped and reported.

The integration runs sync during startup and then hourly.

## Local Control Model

Better Local Tuya uses Tuya Cloud to learn about the device, but uses local LAN
Tuya connections for control whenever possible.

This provides:

- faster command response
- fewer cloud round trips
- better Home Assistant automation behavior
- better behavior when bridging entities into Apple Home through HomeKit Bridge
- more control over Home Assistant entity types and attributes

Some Tuya devices do not expose useful local control. Battery PIR sensors and
some event-only devices may report to Tuya Cloud but refuse local Tuya TCP
connections. Better Local Tuya does not fake those live values from old cloud
shadow data. If there is no real local value source, those entities should be
unavailable.

## LAN Recovery And Offline Devices

Many Tuya lights and plugs disappear from the LAN when someone turns them off at
the wall. Better Local Tuya is designed to recover without requiring a Home
Assistant restart.

Reconnect backoff:

```text
5s -> 10s -> 20s -> 40s -> 80s -> 160s -> 300s
```

After that, retries continue every 5 minutes. A successful connection resets the
backoff to 5 seconds. Tuya LAN discovery can also trigger an immediate reconnect
when a device comes back online.

For IP changes, the fork can use LAN discovery, ARP/MAC evidence, and local-key
probing to recover stale hosts.

Repeated identical connection failures are throttled in the Home Assistant log,
so powered-off devices do not create hundreds of warning entries while still
being retried.

## Device Capability Improvements

### Lights

Better Local Tuya uses Tuya metadata to infer light capability more accurately:

- power
- brightness ranges
- color temperature ranges
- RGB/RGBW color datapoints
- work mode
- scene datapoints where applicable

This is intended to expose warm/cool white controls and color controls when the
device actually supports them.

### Switches, Plugs, And Energy

Switch-like devices are mapped from `switch`, `switch_1`, and similar Tuya
codes. Energy datapoints such as current, power, voltage, and energy are added
where metadata provides them.

### Covers And Blinds

Covers are mapped only where the metadata gives high-confidence open, close,
stop, or position controls. If Tuya reports reversed motor direction, Better
Local Tuya maps that into Home Assistant control semantics so open and close are
not backwards.

Useful read-only datapoints such as `countdown_left` remain visible as sensors.

### Heaters And Climate Devices

Heaters and climate-like devices are inferred from Tuya model metadata where
possible, including supported mode and fan-mode enum values. This avoids sending
unsupported modes that a device would reject.

### Sensors And Binary Sensors

Safe read-only scalar and boolean datapoints can be imported as sensors or
binary sensors. If the device does not provide local updates, entities remain
unavailable rather than showing stale cloud values as if they were live.

## Migration From Upstream LocalTuya

This fork keeps the `localtuya` domain and is intended to replace upstream
LocalTuya in place.

Before switching:

1. Back up Home Assistant.
2. Back up `/config/custom_components/localtuya`.
3. Disable HACS auto-updates for upstream LocalTuya.
4. Replace the custom component code.
5. Restart Home Assistant.

Existing config entries should continue to load. If something goes wrong,
restore your backed-up `custom_components/localtuya` folder and restart Home
Assistant.

## Debugging

Enable debug logging when reporting issues:

```yaml
logger:
  default: warning
  logs:
    custom_components.localtuya: debug
    custom_components.localtuya.pytuya: debug
```

When possible, include:

- device name and device type
- Tuya product name/model
- whether the device is online in the Tuya app
- whether it is reachable on the LAN
- relevant LocalTuya logs
- which entities were imported or skipped

Do not include local keys, Tuya client secrets, or Home Assistant credentials in
public issues.

## Limitations

- Tuya device schemas are inconsistent across brands and firmware versions.
- Auto-import focuses on common devices and high-confidence mappings.
- Cloud-event-only devices may not be usable locally without a separate cloud
  event bridge.
- Better Local Tuya should not be installed beside upstream LocalTuya because
  both use the same Home Assistant domain.

## Project Status

This fork is currently maintained as Better Local Tuya. A subset of the work has
also been proposed upstream as smaller pull requests against
`rospogrigio/localtuya`.

See:

- [Fork notes](docs/FORK.md)
- [Upstream PR plan](docs/UPSTREAM_PR_PLAN.md)

## Thanks And Credits

Better Local Tuya is built on the work of
[`rospogrigio/localtuya`](https://github.com/rospogrigio/localtuya). Thank you to
the upstream maintainers and contributors for the integration this fork builds
upon.

The upstream project itself began from earlier Tuya local-control work by:

- [NameLessJedi](https://github.com/NameLessJedi/localtuya-homeassistant)
- [mileperhour](https://github.com/mileperhour/localtuya-homeassistant)
- [TradeFace](https://github.com/TradeFace/tuya/)
- sean6541, for Python handler work for Tuya devices

This fork would not exist without that foundation.

## License

This project follows the license of the upstream LocalTuya project. See
[LICENSE](LICENSE).
