---
layout: page
title: AutoAniManager
description: A local-first anime manager that connects Bangumi metadata, media discovery, downloads, playback, and watch progress in one desktop workflow.
img: assets/img/projects/autoanime/architecture.png
importance: 1
category: fun
---

**2026 · Personal project · Qt Quick/QML · libmpv · FastAPI · SQLite · Bangumi API**

[Source code on GitHub](https://github.com/ch-yu02/AutoAnimeManager)

AutoAnimeManager started from a simple annoyance: my Bangumi collection, local video files, download client, and playback history all lived in different places. I wanted a single local application that could treat them as one workflow instead of several unrelated tools.

The current version is a native desktop application. It synchronizes Bangumi metadata, scans and matches local media, searches configured RSS sources for missing episodes, scores release candidates, sends selected releases to qBittorrent, embeds playback through libmpv, and persists watch progress in the same local data model.

## Architecture

<div style="width: 75%; margin-inline: auto;">
{% include figure.liquid loading="eager" path="assets/img/projects/autoanime/architecture.png" title="AutoAnimeManager system architecture" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Overall architecture of the local-first workflow.
</div>

The application is split into three main layers. A **Qt Quick/QML desktop client** provides the user interface and embedded player. A **FastAPI backend** owns Bangumi synchronization, media-library matching, release search, download jobs, playback sessions, and application state. **SQLite** stores subjects, episodes, local files, matches, watch state, release candidates, and download history.

The desktop client talks to the backend through a dedicated `BackendClient`. Playback uses a separate `PlayerController`, with libmpv rendering directly into a QML item through the Render API.

## Native desktop workflow

<div style="width: 60%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/home.png" title="AutoAnimeManager home screen" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  A Qt Quick home screen showing Continue Watching, currently followed titles, and the current media-review state.
</div>

The home view combines playback state with library state. A partially watched episode can be resumed directly, followed titles remain grouped by Bangumi collection state, and newly imported media appears independently of whether it has already been watched.

The subject page then brings the entire lifecycle of one title together: episode metadata, local availability, watch state, download progress, release search, manual magnet entry, and playback all operate on the same Episode record.

## Media library and matching

The backend keeps **Bangumi episodes** separate from **local media files** and connects them through explicit matching records. Library scans only reprocess new or changed files, using filename parsing, partial hashes, optional `ffprobe` metadata, and full hashes when duplicates need to be distinguished.

Matching is deliberately conservative. Low-confidence matches, batch releases, conflicting manifests, and other ambiguous cases are sent to a review queue instead of being silently assigned to the wrong episode. Manual matches can be locked and persisted in `manifest.json`, so later scans preserve verified decisions.

The resulting relationship is simple but important:

**Bangumi Subject → Episode → Local Media File → Playback History**

This lets the rest of the application work with stable episode identities instead of trying to infer everything again from filenames.

## Release search and download flow

<div style="width: 60%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/release-search.png" title="Release candidate search for a missing episode" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  The release-candidate dialog.
</div>

Missing episodes can now enter a release-search workflow instead of requiring a magnet link from outside the application. Configured RSS sources are queried, release names are parsed, and candidates are scored against the target title and episode. The scorer also considers season/part scope, duplicate torrents, and user preferences such as release group, subtitle language, resolution, and codec.

Candidates that clearly do not match are filtered out. The remaining results are ranked and shown to the user with their parsed metadata and match reasons. Selecting one creates a persistent qBittorrent download job. Manual magnet input is still available as a fallback.

<div style="width: 60%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/downloads.png" title="qBittorrent download management" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  The Downloads page.
</div>

qBittorrent remains an external downloader, but the application owns the job lifecycle around it. Tasks persist across restarts and can be paused, resumed, retried, or deleted. Downloads are written directly into the configured media library so that completed files can flow back into the normal scan-and-match pipeline.

This creates a mostly closed path from a missing episode to a playable local file:

**missing episode → RSS search → candidate scoring → user selection → qBittorrent → media library → episode match → playback**

## Native playback and watch state

<div style="width: 60%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/autoanime/player.png" title="Embedded libmpv player" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  An episode playing inside the Qt Quick application.
</div>

The player is implemented with **libmpv Render API** rather than browser transcoding or a separately launched player window. It supports seeking, volume and mute control, playback speed, embedded and external subtitles, audio-track switching, fullscreen, keyboard shortcuts, previous/next episode navigation, EOF handling, and continuous playback.

Playback progress is reported to a backend playback session rather than stored only in the UI. The backend decides when progress should update the persistent episode state, when an episode should be marked watched, and which later MAIN episode is eligible for continuous playback. Short accidental playback does not overwrite useful progress, while explicit manual watched/unwatched choices take priority.

The desktop client also inhibits system sleep only while media is actively playing, so long playback sessions are not interrupted while normal idle behavior is preserved elsewhere in the application.

## Current state

The main interactive workflow is already connected end to end:

**sync Bangumi → scan local media → review/match files → search missing releases → download → play → save progress → continue later**

The project is still evolving toward the original goal of a mostly unattended anime manager, but the current implementation already treats metadata, local files, download state, and playback history as one coherent local system rather than independent utilities.
