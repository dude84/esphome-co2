# esphome-co2

Public reusable ESPHome library for SCD41 CO2 sensors (ESP32-C3), built on the
remote-packages model. A device is a thin consumer that pulls all logic from this
repo, pinned by one `sentinel_version` substitution.

## Layout

```
sg-co2/packages/
  network.yaml      # wifi / api / ota / captive_portal / web_server  (${subs}, no !secret)
  co2.yaml          # i2c + scd4x sensor
  diagnostics.yaml  # logger + onboard LED
sg-co2-office.yaml      # example production consumer (pinned @tag)
sg-co2-office.dev.yaml  # local-dev consumer (local !include, no fetch)
secrets.yaml.example    # template; real secrets.yaml is gitignored
```

## Use on a device

Drop a consumer like `sg-co2-office.yaml` into `/config/esphome/`, provide
`secrets.yaml` (see `secrets.yaml.example`), and flash from Home Assistant.
Take a new release by bumping `sentinel_version`.

## Required secrets

`wifi_ssid`, `wifi_password`, `api_password` (API encryption key), `ota_password`,
`failsafe_ap_password` (fallback hotspot password).

## Develop

```
esphome config sg-co2-office.dev.yaml   # validate against working tree
git tag vX.Y.Z && git push origin main vX.Y.Z
gh release create vX.Y.Z --generate-notes
```
