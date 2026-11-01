# Changelog

All notable changes to the Frigate Temporary Mask integration will be documented in this file.

## [1.0.0] - 2026-11-01

### Added
- Initial standalone release for HACS.
- Dynamic temporary mask generation with configurable padding (+10% default).
- Support for Frigate 0.18+ dictionary format masks and API dynamic updates without detector restarts.
- Backward compatibility for Frigate 0.14-0.17 (list coordinates) and 0.13 (pixel coordinates).
- `sensor.frigate_active_masks` diagnostic entity.
- Admin-only `frigate_temp_mask.delete_event` service for secure Frigate event pruning.
- Full Home Assistant Config Flow support for UI configuration.
- Companion blueprint for iOS and Android mobile actionable notifications.

