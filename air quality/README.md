# CO₂ – Progressive Alerts (multi-device, area-aware, cooldown, “only-if-home” toggle)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgist.github.com%2Fjubnl%2F25ef574395ba8fe4535311f24e866489)

[GitHub Link Click Here for Issues](https://github.com/jubnl/ha-blueprints/blob/main/air%20quality/ha_ppm_threshold_notification.yaml)

[Gist Link Click Here](https://gist.github.com/jubnl/25ef574395ba8fe4535311f24e866489)

**TL;DR**  
A robust CO₂ alert blueprint that:

- Triggers on **Warn / Alert / Critical** thresholds (each with its own `for:` duration)
- Notifies **multiple Companion App devices** and/or any **`notify.*` entities**
- Optional **“only if devices are at home”** gate (default **on**)
- Adds the **sensor’s Area** to the notification title (if defined)
- Has a **cooldown** to prevent spam
- Emits rich **Logbook diagnostics** for troubleshooting
- Uses **`notify.send_message`** where available and falls back to legacy `notify.mobile_app_<slug>` for phones

---

## What it does

- Monitors a CO₂ sensor (device class `carbon_dioxide`).
- Sends progressive notifications:
    - **WARN** — default `1000 ppm`, `for: 5 min`
    - **ALERT** — default `1400 ppm`, `for: 3 min`
    - **CRITICAL** — default `2000 ppm`, `for: 1 min`
- Title includes the **Area** of the CO₂ sensor: e.g., `Air quality — Kitchen`.
- **Cooldown** (default 15 min) between any two notifications.
- Mobile notifications go to selected **Companion App devices**:
    - If **“Notify only if devices are at home”** is enabled (default), the blueprint resolves each device’s
      `device_tracker.*` and sends only when it’s `home`.
    - If disabled, it sends regardless of presence.
- Also supports **additional `notify.*` entities** (notify groups, Telegram, etc.).

---

## Inputs

| Input                                  | Type                                         |      Default | Notes                                              |
|----------------------------------------|----------------------------------------------|-------------:|----------------------------------------------------|
| **CO₂ sensor**                         | `sensor` (`device_class: carbon_dioxide`)    |            — | Source for ppm                                     |
| **Mobile app devices**                 | list of `device` (integration: `mobile_app`) |         `[]` | One or more phones/mac with Companion App          |
| **Notify entities (optional)**         | list of `entity` (domain: `notify`)          |         `[]` | Any `notify.*` targets (groups, Telegram, etc.)    |
| **Notify only if devices are at home** | `boolean`                                    |       `true` | Presence gate using each device’s `device_tracker` |
| **Warn threshold / for**               | `number` / `minutes`                         | `1000` / `5` | First level                                        |
| **Alert threshold / for**              | `number` / `minutes`                         | `1400` / `3` | Second level                                       |
| **Critical threshold / for**           | `number` / `minutes`                         | `2000` / `1` | Third level                                        |
| **Cooldown (minutes)**                 | `number`                                     |         `15` | Minimum gap between any two sends                  |
| **Debug logging to Logbook**           | `boolean`                                    |       `true` | Verbose Logbook traces everywhere                  |

---

## How notifications are sent

- If your target exposes **`notify.*` entities**, the blueprint uses **`notify.send_message`** with `target.entity_id`.
- For Companion App **phones**, if no `notify.*` entity exists on the device, it falls back to legacy *
  *`notify.mobile_app_<slug>`** (slug derived from the device’s `device_tracker` object_id).  
  This avoids templating `device_id` (which device actions don’t support inside loops) and works reliably with multiple
  devices.

---

## Logbook diagnostics

When **Debug** is on, each run logs:

- Trigger level, ppm, thresholds, Area
- Cooldown timing (`last_triggered`, `seconds_since`, decision)
- Per device: found `device_tracker`, state, any `notify.*` on the device, computed legacy service name
- Which path was used to send (entity vs legacy) or why it was skipped

This makes it easy to see **exactly** why a device did or didn’t get a push.

---

## Install

1. Open **Settings → Automations & Scenes → Blueprints**.
2. **Import Blueprint** and paste the Gist URL above.
3. Create a new automation from this blueprint:
    - Pick your **CO₂ sensor**.
    - Select your **phones** (Companion App devices).
    - (Optional) select extra **`notify.*` entities**.
    - Adjust thresholds/durations/cooldown if needed.

---

## Tips & troubleshooting

- **No notification on a phone?**  
  Check the Logbook entry. Common causes:
    - Presence gate: device not `home` (toggle **“Notify only if devices are at home”** off to test).
    - No `device_tracker.*` on that device (Companion App presence/permissions).
    - The device doesn’t expose a `notify.*` entity; fallback uses `notify.mobile_app_<tracker_slug>`. The log shows the
      exact name used.

- **Want different thresholds per room?**  
  Create multiple automations from this blueprint, one per sensor/room.

- **Too chatty?**  
  Increase `for:` times and/or `cooldown_minutes`.

- **Noisy logs?**  
  Turn off **Debug logging to Logbook** in the inputs.

---

## Changelog

- **v1.0** – Progressive levels, multi-device, area-aware title, presence gate (toggle), cooldown, Logbook diagnostics,
  hybrid notify (entity + legacy mobile fallback).
