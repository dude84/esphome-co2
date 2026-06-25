# CLAUDE.md

Guidance for working in this repo.

## What this is

Public reusable **ESPHome library** for SCD41 CO2 sensors on **ESP32-C3**, built on
the official **remote-packages** model. A physical device runs a thin *consumer*
YAML that pulls all real logic from this repo over `github://`, pinned to one
release tag via the `sentinel_version` substitution.

## Architecture

```
sg-co2/packages/          # reusable logic — fetched remotely by consumers
  network.yaml            # wifi / api / ota / captive_portal / web_server
  co2.yaml                # i2c + scd4x sensor
  diagnostics.yaml        # logger + onboard LED + Config Version entity
sg-co2-office.yaml        # PROD consumer — pulls packages via remote url @ tag
sg-co2-office.dev.yaml    # DEV consumer  — pulls packages via local !include
secrets.yaml.example      # template; real secrets.yaml is gitignored
```

Two consumer flavors, same substitutions:
- **prod** (`*.yaml`): `packages.remote.url` + `ref: ${sentinel_version}` → fetches a tagged release.
- **dev** (`*.dev.yaml`): `packages.*: !include sg-co2/packages/...` → builds the working tree, no fetch. `sentinel_version: dev`.

## Key decisions (don't undo without reason)

- **Remote packages CANNOT use `!secret`.** Every secret in `sg-co2/packages/*` is a
  `${substitution}`. Only the consumer fills them via `!secret`. Keep package files
  secret-free.
- **One `room` knob** drives identity. Set `room` (display) + `room_id` (slug);
  `name`, `friendly_name`, `ap_ssid` derive from them.
- **AP SSID stays short.** Wi-Fi SSID hard limit is 32 chars. Fallback AP is
  `Sg-Co2 ${room} Fallback` (the word "Hotspot" was dropped to fit). Don't pad it.
- **Config Version entity** (`diagnostics.yaml`): a template `text_sensor` with
  `update_interval: never` would stay blank, so it publishes the static
  `${sentinel_version}` once `on_boot` at `priority: -100`. Surfaces which release a
  device runs (resolves to `dev` from the dev consumer).
- **CO2 filter**: publish on `delta: 25` OR `heartbeat: 5min` — cuts MQTT/API chatter
  while guaranteeing a value at least every 5 min.
- **Pins (ESP32-C3)**: `sda GPIO2`, `scl GPIO3`, `led GPIO8` (inverted, onboard).

## Build / run / test

```bash
esphome config sg-co2-office.dev.yaml    # validate against the working tree
esphome compile sg-co2-office.dev.yaml   # full local build (optional)
```
Flashing is done from Home Assistant (ESPHome add-on), not the CLI.
Requires a local `secrets.yaml` (copy `secrets.yaml.example`, fill real values).

## Release workflow

1. Edit logic in `sg-co2/packages/*`, iterate with the **dev** consumer.
2. `esphome config sg-co2-office.dev.yaml` — must pass.
3. Bump tag + push:
   ```bash
   git tag vX.Y.Z && git push origin main vX.Y.Z
   gh release create vX.Y.Z --generate-notes
   ```
4. Devices take the release by bumping `sentinel_version` in their consumer YAML.

Current released tags: v0.1.0 … v0.1.3. Example consumer pins `v0.1.3`.

## Conventions

- Per-device knobs live in the consumer's `substitutions:` block: `room`/`room_id`,
  `static_ip`, pins, `co2_update_interval`. Never hard-code these in packages.
- Required secrets: `wifi_ssid`, `wifi_password`, `api_password` (API encryption
  key), `ota_password`, `failsafe_ap_password`.
- `secrets.yaml`, `.esphome/`, `*.bin` are gitignored. Never commit them.
- Related skill: `esphome-remote-packages` (migrating devices onto / extending this model).
