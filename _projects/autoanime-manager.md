---
layout: page
title: AutoAnime Manager
description: A local-first anime manager that unifies Bangumi metadata, media discovery, downloads, playback, watch progress, and file lifecycle management.
img: assets/img/projects/autoanime/architecture.png
importance: 1
category: fun
_styles: |
  article {
    font-size: 1.1rem;
    line-height: 1.75;
  }

  article blockquote {
    font-size: inherit;
  }
---

**2026 · Personal project · Qt Quick/QML · C++ · libmpv · FastAPI · SQLite · Bangumi API · qBittorrent**

[Source code on GitHub](https://github.com/ch-yu02/AutoAnimeManager)

AutoAnimeManager grew out of a simple annoyance: my Bangumi collection, local video files, download client, and playback history all lived in separate tools. I wanted one local application that could treat them as a single workflow.

The current version is a native desktop application with an end-to-end automation loop. It synchronizes Bangumi metadata, scans and matches local media, searches RSS sources for missing episodes, ranks release candidates, supports both manual and automatic downloads through qBittorrent, plays media through an embedded libmpv player, persists watch progress, and manages completed media through a recoverable cleanup workflow.

> **Core workflow:** Bangumi sync → local matching → release search → download → playback → progress → retention / cleanup

## Architecture

<div style="width: 78%; margin-inline: auto;">
{% include figure.liquid loading="eager" path="assets/img/projects/autoanime/architecture.png" title="AutoAnimeManager system architecture" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Local-first system architecture.
</div>

The application has three main layers. A **Qt Quick/QML desktop client** provides the native interface and embedded player. A **FastAPI backend** owns synchronization, media matching, release discovery, download jobs, playback sessions, scheduling, and cleanup logic. **SQLite** keeps the durable state for subjects, episodes, media files, matches, watch history, release candidates, downloads, scheduler runs, and cleanup records.

The desktop client communicates with the backend through `BackendClient`. Playback is managed separately by `PlayerController`, with libmpv rendering directly into a QML item through the Render API. The production client no longer depends on Vue, QWebEngine, or a browser-based playback layer.

## Desktop experience

<div style="width: 62%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/home.png" title="AutoAnimeManager home screen" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Home view.
</div>

The home screen brings playback and library state together. Partially watched episodes can be resumed directly, followed titles remain organized by Bangumi collection state, and recently imported media is visible independently of watch status.

A subject page then serves as the working view for a title: episode metadata, local availability, watch state, release search, download status, manual magnet entry, and playback all operate on the same Episode record. Collection status can also be updated from the client and is written to Bangumi before the local state is changed.

## Media library and matching

The backend keeps **Bangumi episodes** separate from **local media files**, connecting them through explicit match records. Scans skip unchanged files and only reprocess new or modified media, using filename parsing, partial hashes, optional `ffprobe` metadata, and full hashes when duplicate detection requires them.

Matching is intentionally conservative. Low-confidence matches, batch releases, manifest conflicts, and ambiguous multi-file cases are sent to a review queue rather than silently assigned. Manual matches can be locked and written to `manifest.json`, so later scans preserve decisions that have already been verified.

The resulting data relationship is straightforward:

**Bangumi Subject → Episode → Local Media File → Playback History**

That stable identity model is what allows search, downloads, playback, and cleanup to operate on the same episode instead of repeatedly inferring state from filenames.

## Release search and downloads

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/autoanime/release-search.png" title="Release candidate search" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/autoanime/downloads.png" title="Download management" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Release selection and download management.
</div>

Missing episodes can be resolved inside the application. Search queries are built from Bangumi titles and aliases, RSS results are merged and deduplicated, and each release is parsed for season or part, episode range, release group, resolution, codec, language, and batch information. Candidates are stored with scores and explicit acceptance or rejection reasons.

Manual mode lets me inspect and choose a candidate. Automatic mode is deliberately stricter: the demand planner only considers aired, unwatched, non-ignored **MAIN** episodes from currently followed titles, and it submits only high-confidence candidates. Selection also accounts for release timing, subtitle-language preferences, and consistency with an existing release group for the same subject.

Selected releases become persistent qBittorrent jobs. The backend monitors their state across restarts, writes completed files directly into the subject's media-library directory, associates successful downloads with the target episode, and writes the match back to the local manifest. Ambiguous batches or unexpected file layouts fall back to Library Review instead of being imported blindly.

Manual magnet links and BTIH hashes remain available as a fallback.

## Native playback and watch state

<div style="width: 62%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/player.png" title="Embedded libmpv player" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Native libmpv playback.
</div>

Playback uses the **libmpv Render API** rather than browser transcoding or an external player window. The client supports precise seeking, volume and mute control, playback speed, embedded and external subtitles, audio-track switching, fullscreen, keyboard shortcuts, previous/next episode navigation, EOF handling, and continuous playback.

Progress is reported to a backend playback session instead of being owned by the UI. The backend decides when progress should be persisted, when an episode qualifies as watched, and which later MAIN episode is eligible for continuous playback. Very short accidental playback does not overwrite meaningful progress, while explicit watched/unwatched actions take precedence over automatic rules.

The desktop client also inhibits system sleep only while media is actively playing, preventing long sessions from being interrupted without changing normal idle behavior elsewhere.

## Scheduling and media lifecycle

The current application also runs the recurring work that turns the interactive workflow into an automated one. A persistent scheduler coordinates **BangumiSync**, **LibraryScan**, **DemandRefresh**, **ReleaseSearch**, **DownloadMonitor**, and **Cleanup**. Runs are recorded, same-type tasks cannot re-enter, failures back off automatically, and interrupted work is recovered after restart.

Completed media can follow a separate lifecycle:

**watched → retained → quarantined → restored or permanently deleted**

Automatic cleanup is opt-in. A subject is only eligible after it is confirmed complete, all existing MAIN episodes are watched, the retention period has elapsed, and there are no active downloads, unresolved matches, or currently playing files. Quarantine keeps deletion reversible; permanent cleanup removes media files while preserving metadata, watch history, download history, and cleanup records.

## Current state

The main workflow is now connected from metadata synchronization through automated acquisition and playback to post-watch file management:

**sync → match → plan → search → download → play → save progress → retain / clean up**

What began as a local playback utility has become a single-user media system in which metadata, local files, acquisition decisions, playback state, scheduled work, and file retention all share one persistent model.
