---
name: Airing schedule feature reality
description: Which schedule features actually work and which are placeholder logic (as of the exact-alarm/filter rework).
---

- Preferred title language still only applies in UI (card/search/add); notification content still always uses `titleUserPreferred` — not fixed.
- Notification bell now uses `setExactAndAllowWhileIdle` with a `canScheduleExactAlarms()`/`SCHEDULE_EXACT_ALARM` permission-prompt flow (falls back to inexact only if permission is denied). Series alarms are re-armed on every `ScheduleDataRefreshWorker` weekly fetch, not just the currently loaded week.
- Favorite-source filter and source-availability filter are now real per-anime checks, but bounded to anime already in the user's library: `AiringScheduleScreenModel.applyFilters` matches a schedule entry's title against library anime titles to resolve which of the user's favorite/pinned sources actually carry it, then gates/reorders by that specific match — not a global proxy. Entries for anime not yet in the library can never pass these filters, by design (can't search every source for every anime, too slow).
- Both filter toggles must check `favoriteSourceIds() + sourcePreferences.pinnedSources()` together (not favorites alone) — a pinned-only setup (no favorites marked) still needs the filters to work.
- Auto-sync source upload time (`ScheduleRefreshWorker`) now fetches the real episode list from each matched library anime's actual source (`SourceManager.get(source).getEpisodeList(...)`) and diffs real `date_upload` against AniList's `airingAt`, bounded to library anime on tracked sources with a per-run source-call cap (avoids scanning full catalogues).
- `ScheduleNotifications`'s persisted `scheduledAlarmKeys` set is mutated from multiple callers (UI, workers, alarm receiver) — all mutations must go through its internal mutex-guarded helpers; a direct read-modify-write from the receiver reintroduces a lost-update race.

**Why:** The original schedule model used a week-view AniList payload with no source mapping. Real per-anime filtering/delay-learning only became feasible by cross-referencing against the user's own library data (title match), which bounds the work to a small, already-fetched set instead of every source's catalogue.

**How to apply:** When touching schedule filtering/delay code, keep matching bounded to library anime; never add a per-anime feature that would require querying every installed source for every scheduled anime.
