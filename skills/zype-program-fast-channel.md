---
name: zype-program-fast-channel
description: Build, schedule, review and publish a Zype Playout FAST/linear channel, and export its EPG.
api: Zype Playout Scheduler API
base_url: https://api.zype.com
spec: openapi/zype-playout-scheduler.json
operations:
  - getPlayoutSchedulerChannels
  - createPlayoutSchedulerChannel
  - getPlayoutSchedulerChannel
  - updateChannel
  - getPlayoutSchedulerChannelDraft
  - getPlayoutSchedulerChannelDraftsRundown
  - getPlayoutSchedulerOperations
  - createPlayoutSchedulerOperation
  - revertPlayoutSchedulerOperation
  - revertAllPlayoutSchedulerOperations
  - publishPlayoutSchedulerChannelDraft
  - getPlayoutSchedulerChannelPublished
  - getPlayoutSchedulerChannelPublishedJsonRundownEPG
  - channelXMLTVEPG
  - getRokuEPG
  - getVizioEPG
  - getTclEPG
  - getWurlEPG
  - getChannelsDestinations
  - createChannelDestination
  - getRecurrenceRules
  - createRecurrenceRule
  - stopLiveChannel
generated: '2026-08-28'
method: generated
source: openapi/zype-playout-scheduler.json
---

# Program a Zype Playout channel

82 operations live in `openapi/zype-playout-scheduler.json`. This is the one Zype surface
with an explicit undo model — use it.

## Authentication is different here

The Playout Scheduler is the **only** Zype API that takes its key in a header:
`X-API-Key: <key>`. Every other Zype product takes `?api_key=` in the query string.
Sending the Playout key as a query parameter will fail.

## The draft / published split

A channel has a **draft** and a **published** version. Edits land on the draft. Air sees
the published version. They are separate resources.

1. **Create or select the channel.** `createPlayoutSchedulerChannel` /
   `getPlayoutSchedulerChannels`. Attach a delivery profile — HLS
   (`the List HLS Profiles operation (GET /scheduler/v1/profiles/hls — NO operationId is declared)`), UDP or RTMP.
2. **Add destinations.** `getDestinationTypes`, then `createChannelDestination`.
   Standard connectors are HLS (Local Now, Plex, Pluto TV, Samsung TV+, Xumo, web,
   mobile); premium connectors are RTMP/SRT/RTP/RIST/Zixi (Twitch, YouTube, The Roku
   Channel). Connectors are billed per destination per channel per month.
3. **Edit the timeline.** Every edit is a **timeline operation**:
   `createPlayoutSchedulerOperation`. List them with `getPlayoutSchedulerOperations`.
   Assets come from `getPlayoutSchedulerBlocksV2`, `listAdsV2`, `listBumpersV2`,
   `listVideosV2`, `listPlayoutPlaylistsV2` and `getGraphics`.
   Recurring programming uses `createRecurrenceRule`.
4. **Review before air.** `getPlayoutSchedulerChannelDraftsRundown` renders the draft
   rundown. Read it and show it to a human.
5. **Undo if wrong.** `revertPlayoutSchedulerOperation` rewinds the draft to a chosen
   operation; `revertAllPlayoutSchedulerOperations` discards the whole edit set. Both act
   on the **draft only**.
6. **Publish.** `publishPlayoutSchedulerChannelDraft`.

## The irreversible step

**`publishPlayoutSchedulerChannelDraft` cannot be undone through the API.** There is no
unpublish and no revert-published operation — `getPlayoutSchedulerChannelPublished` is
read-only, and Zype publishes no rollback window. The only recovery is to edit the draft
again and republish, which means the wrong schedule is on air in the meantime.
Never publish without explicit human confirmation. `stopLiveChannel` stops an on-air
channel and has no matching start operation on this resource.

## Export the guide

- **XMLTV** (open interchange): `channelXMLTVEPG` —
  `GET /scheduler/v1/channels/{id}/published/rundown/xmltv.xml`.
- **JSON rundown**: `getPlayoutSchedulerChannelPublishedJsonRundownEPG`.
- **Per-platform**: `getRokuEPG`, `getVizioEPG`, `getTclEPG`, `getWurlEPG`.

Prefer XMLTV for anything that already speaks it; the platform feeds are vendor formats.
