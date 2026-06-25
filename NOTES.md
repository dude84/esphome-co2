# NOTES

Working notes / project state. Checkpoint captured before account migration (2026-06-26).

## Status at checkpoint

- Working tree clean, `main` up to date with `origin/main`, no unpushed commits.
- Tags `v0.1.0`–`v0.1.3` all exist locally **and** on remote.
- Example consumer (`sg-co2-office.yaml`) pins `v0.1.3`.
- No outstanding in-flight plan or open tasks at this checkpoint. The prior Claude
  Code session was cleared, so there was no unsaved session plan to preserve — this
  file records genuine project state, not reconstructed conversation.

## Known constraints / gotchas

- **Wi-Fi SSID 32-char cap** — fallback AP name must stay short. Adding words like
  "Hotspot" overflows for longer room names. See commit 28d9c0f.
- **Remote packages can't `!secret`** — keep `sg-co2/packages/*` secret-free; consumers
  inject secrets as substitutions.
- **Config Version blank-on-boot trap** — template text_sensor with
  `update_interval: never` never publishes on its own; it's published once `on_boot`.
  Don't "simplify" that away. See commit 8a63d85.

## Possible next steps (not committed to — backlog ideas only)

- Onboard more rooms: copy `sg-co2-office.yaml` → `sg-co2-<room>.yaml`, set
  `room`/`room_id`/`static_ip`.
- Optional: external C++ component for CO2 DSP if filtering needs grow (see the
  `esphome-remote-packages` skill).
- Consider CI that runs `esphome config` on the dev consumer per push.

## Local-only, NOT in git (will NOT migrate with the repo)

- `secrets.yaml` — real Wi-Fi/API/OTA/AP secrets. Gitignored by design. Recreate on
  any new machine from `secrets.yaml.example`. **This is the only at-risk local state
  and it must stay out of git.**
- `.esphome/` — build cache, regenerated on demand.
