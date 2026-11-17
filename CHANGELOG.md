# Changelog

All notable changes to the Frigate Temporary Mask integration will be documented in this file.

## [1.1.0] - 2026-11-17

### Added
- **Multi-label mask support**: Comma-separated labels (e.g. `person, dog`) write the mask polygon under every target object filter and automatically synchronize/clean up copies on expiration or removal.
- **Polygon geometry updates**: `add_mask` service now accepts explicit `polygon` coordinates and `label` fields to update active mask shapes directly.
- **Reliable iPhone & Apple Watch notification actions**: Packed event IDs directly into action identifiers (`TEMP_MASK_24H:<event_id>`), fixing dropped action data on iOS and watchOS taps, and deduplicated notification event listeners so taps fire exactly once.

### Fixed
- **Automatic pruning for expired masks**: Internal prune passes and expiration timers now call the removal helper directly instead of building empty ServiceCall objects, ensuring masks always prune upon expiry.
- **Adopted masks expiration**: Setting a duration for an adopted Frigate mask now clears its orphan status so it expires normally.
- **Offline recovery mask draining**: Masks that expire while Frigate is offline are reliably queued and pruned once Frigate reconnects.

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

