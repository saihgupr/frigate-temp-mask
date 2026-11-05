# Frigate Temporary Mask

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://hacs.xyz)
[![GitHub Release](https://img.shields.io/github/v/release/saihgupr/frigate-temp-mask?style=flat-square)](https://github.com/saihgupr/frigate-temp-mask/releases)
[![Hassfest](https://github.com/saihgupr/frigate-temp-mask/actions/workflows/hassfest.yml/badge.svg)](https://github.com/saihgupr/frigate-temp-mask/actions/workflows/hassfest.yml)
[![HACS Validation](https://github.com/saihgupr/frigate-temp-mask/actions/workflows/hacs.yml/badge.svg)](https://github.com/saihgupr/frigate-temp-mask/actions/workflows/hacs.yml)

A Home Assistant integration that dynamically creates and manages temporary false-positive masks in [Frigate NVR](https://frigate.video/).

It works fully standalone — no dashboard card required. It lets you silence false alarms (e.g. parked delivery vans, moved wheelbarrows, packages, shadows) from an automation, mobile notification, or Developer Tools, applies an expanded bounding-box polygon directly to Frigate's configuration, and automatically prunes the mask once its timer expires.

Pairs optionally with the [Frigate Events Card](https://github.com/saihgupr/frigate-events-card) if you want a visual Mask Manager, but the card is never required.

<p align="center">
  <img src="https://raw.githubusercontent.com/saihgupr/frigate-events-card/main/images/snapshots/temp-mask.png" width="49%" alt="Temporary Mask Manager" />
  <img src="https://raw.githubusercontent.com/saihgupr/frigate-events-card/main/images/snapshots/temp-mask-1.png" width="49%" alt="Frigate Config Object Mask" />
</p>

---

## Features

- **One-Tap False Alarm Masking**: Creates a polygon bounding box (with configurable padding, default +10%) around any Frigate false detection.
- **Automatic Detector Sync**: Restarts the Frigate detector backend automatically (~1–2 seconds) so false alarms stop immediately.
- **Frigate 0.18+ Instant Removal**: On modern Frigate versions, masks are removed via the Frigate API (`PUT /api/camera/{camera}/set/object_mask/{name}`) and cleaned from disk with **zero detector restarts**.
- **Cross-Version Architecture**: Full automatic format detection and preservation:
  - **Frigate 0.18+**: Dictionary-format object masks and live API control.
  - **Frigate 0.14–0.17**: List-format polygon coordinates.
  - **Frigate 0.13 & earlier**: Automatic normalized-to-pixel coordinate translation.
- **Diagnostic Sensor (`sensor.frigate_active_masks`)**: Real-time mask count, expiration countdowns, camera names, and pending-restart indicators.
- **Actionable Push Notifications**: Includes a ready-to-import blueprint for iOS and Android actionable notifications to mask false alarms right from your lock screen.
- **Native UI Setup**: Full Home Assistant Config Flow support — configure directly from the UI or via YAML.

---

## Installation

### Option 1: HACS (Recommended)

1. Click the button below to add the repository directly to HACS:

   [![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=saihgupr&repository=frigate-temp-mask&category=Integration)

2. Or add manually in HACS:
   - In Home Assistant, open **HACS > Integrations**.
   - Click the three dots in the top-right corner and select **Custom repositories**.
   - Enter `https://github.com/saihgupr/frigate-temp-mask`, choose **Integration** as the category, and click **Add**.
   - Find **Frigate Temporary Mask** in HACS and click **Download**.
3. **Restart Home Assistant**.

---

### Option 2: Manual Installation

1. Download the latest release from the [Releases](https://github.com/saihgupr/frigate-temp-mask/releases) page.
2. Copy the `custom_components/frigate_temp_mask` directory into your Home Assistant `/config/custom_components/` directory.
3. Restart Home Assistant.

---

## Configuration

### Via User Interface (Recommended)
1. In Home Assistant, navigate to **Settings > Devices & Services**.
2. Click **Add Integration** and search for **Frigate Temporary Mask**.
3. Confirm to finish setup.

### Via YAML (Optional)
Add the following line to your `configuration.yaml`:
```yaml
frigate_temp_mask:
```
Then reload YAML or restart Home Assistant.

---

## Services Reference

The integration registers the following services under **Developer Tools > Actions / Services**:

| Service | Description | Parameters |
|---|---|---|
| `frigate_temp_mask.add_mask` | Adds a temporary bounding-box mask around a false detection in Frigate. | `event_id` (required), `duration_hours` (default: 24), `camera` (optional), `padding` (default: 0.10), `mask_id` (optional) |
| `frigate_temp_mask.set_duration` | Updates the expiration duration for an active temporary mask. | `mask_id` (required), `duration_hours` (required) |
| `frigate_temp_mask.remove_mask` | Removes a temporary mask from Frigate config and API. | `mask_id` (required) |
| `frigate_temp_mask.prune_all` | Clears all temporary masks from Frigate configuration. | *None* |
| `frigate_temp_mask.restart` | Restarts the Frigate detector backend to apply changes. | *None* |
| `frigate_temp_mask.sync` | Synchronizes active mask state with Frigate uptime and config on disk. | *None* |
| `frigate_temp_mask.dismiss_pending` | Dismisses a pending restart notification without restarting Frigate. | `mask_id` (optional) |
| `frigate_temp_mask.delete_event` | Deletes an event, snapshot, and recording from Frigate. | `event_id` (required) |

---

## Mobile Notification Blueprint (iOS & Android)

Add 1-tap **Temporary Mask (24h)** and **Mask (8h)** action buttons directly to Frigate mobile push notifications. Tapping an action automatically creates the temporary mask in Frigate without opening the dashboard.

<p align="center">
  <img src="https://raw.githubusercontent.com/saihgupr/frigate-events-card/main/images/snapshots/temp-mask-2.png" width="340" alt="Frigate Mobile Notification with Temporary Mask Actions" />
</p>

[![Import Blueprint to Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fsaihgupr%2Ffrigate-temp-mask%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Ffrigate_temp_mask_notification.yaml)

**Import Manually**: Go to **Settings > Automations & Scenes > Blueprints > Import Blueprint** and paste:
```text
https://github.com/saihgupr/frigate-temp-mask/blob/main/blueprints/automation/frigate_temp_mask_notification.yaml
```

---

## Using Without the Card

No card needed for core use. All logic lives in this integration:

- **What you need:** the official Frigate integration configured (this integration reads its Frigate URL from it), plus network access from HA to Frigate's HTTP API. No other dependencies.
- **How to mask:** call `frigate_temp_mask.add_mask` with just an `event_id` — camera, label, and box are looked up from Frigate automatically. Works from **Developer Tools > Actions**, any automation/script, or the notification blueprint above.
- **How to manage:** use `sensor.frigate_active_masks` to see active masks, expirations, and pending restarts, and use `set_duration`, `remove_mask`, `prune_all`, `sync`, `restart` to manage them. Expiry, persistence across HA restarts, and Frigate config reconciliation all run internally.
- **What the card adds (optional):** Mask Manager modal, context-menu add/remove, polygon editor, and live mask overlay. Same backend, just a visual frontend.

## Frontend Companion Card (Optional)

For the full interactive Mask Manager UI, timeline scrubber, and live polygon overlays, pair this integration with the **[Frigate Events Card](https://github.com/saihgupr/frigate-events-card)**.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

