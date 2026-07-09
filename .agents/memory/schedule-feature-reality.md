---
name: Airing schedule feature reality
description: Which schedule features actually work and which are placeholder logic.
---

- Preferred title language works in the UI card/search/add flow but not in notification content (alarm receiver always uses titleUserPreferred).
- Notification bell single tap/long press toggles preference states correctly, but alarms use `setAndAllowWhileIdle` without `SCHEDULE_EXACT_ALARM`/`USE_EXACT_ALARM`, so exact timing is unreliable on Android 12+ and missing from `AndroidManifest.xml`.
- "Series" notifications only schedule entries currently in the loaded week; future weeks are not pre-scheduled.
- Favorite-source filter and source-availability filter are global all-or-nothing checks, not per-anime filters. `AiringScheduleEntry` has no source identifiers, so per-anime filtering is impossible with the current model.
- Auto-sync source upload time does not verify per-source availability; it records the same computed delay for every favorite source. It also uses `favoriteSourceIds` while the UI says pinned sources take priority.

**Why:** The schedule feature was built around a week-view AniList model that does not carry source mappings. Filtering, delay learning, and multi-week scheduling would need a redesign to become accurate.

**How to apply:** Before treating these toggles as reliable, verify the data model supports the claim; for notifications, add exact-alarm permission and permission UX if exact timing is required.
