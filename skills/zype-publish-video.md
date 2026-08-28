---
name: zype-publish-video
description: Ingest a video into a Zype library, enrich it with AI-generated metadata, add subtitles, and place it in a playlist and category.
api: Zype Platform API
base_url: https://api.zype.com
spec: openapi/zype-platform.json
operations:
  - createVideo
  - getVideo
  - updateVideo
  - bulkCreateAIMetadata
  - createAIMetadata
  - getAIMetadata
  - applyAIMetadata
  - transcribeVideo
  - translateVideo
  - listLanguages
  - listTranscriptions
  - createSubtitle
  - listSubtitles
  - listCategories
  - createPlaylist
  - addVideosToPlaylist
  - listVideoSourcesForVideo
generated: '2026-08-28'
method: generated
source: openapi/zype-platform.json + https://docs.zype.com/reference/welcome-to-the-zype-api-documentation
---

# Publish a video to Zype

Every operationId below is verified present in `openapi/zype-platform.json`. Do not
invent endpoints; if something you need is not listed here it is not in the contract.

## Before you call anything

- **Credential.** Pass an **Admin key** as the `api_key` **query parameter**:
  `GET https://api.zype.com/videos?api_key=...`. Read Only and Player keys will 403 on
  every write in this skill. The key travels in the query string — never log the full URL.
- **No idempotency.** Zype publishes no idempotency key. A retried `createVideo` creates a
  **second video**. Before retrying a create that timed out, call `listVideos` filtered by
  `source_id` or `q` and confirm the record is absent.
- **Pagination is mandatory** on every list call: `page` (zero-indexed), `per_page`
  (default 10, maximum 100).
- **Errors** carry only `{"message": "..."}`. Branch on the HTTP status: 401 bad key,
  403 wrong key class, 404 wrong id or wrong site, 422 validation.

## Steps

1. **Create the video.** `POST /videos` (`createVideo`). Keep the returned `_id`; every
   later step is keyed on it.
2. **Confirm the sources landed.** `GET /videos/{id}/video_sources`
   (`listVideoSourcesForVideo`). An empty list means ingest has not completed — poll,
   do not re-create.
3. **Generate metadata.** `POST /videos/{id}/ai_metadata/suggestions` (`createAIMetadata`)
   returns a session. Read it with `GET /videos/{id}/ai_metadata/suggestions/{session_id}`
   (`getAIMetadata`). **Nothing is written to the video until you call**
   `POST /videos/{id}/ai_metadata/suggestions/{session_id}/apply` (`applyAIMetadata`) —
   treat `apply` as the point of no return and surface the suggestions to a human first.
   For a batch, `bulkCreateAIMetadata` takes many videos at once.
4. **Transcribe and translate.** `POST /videos/{id}/transcriptions/transcribe`
   (`transcribeVideo`), then `translateVideo` for each target language. Check
   `listLanguages` first for what is supported; read results with `listTranscriptions`.
5. **Attach subtitles.** `POST /videos/{video_id}/subtitles` (`createSubtitle`);
   verify with `listSubtitles`.
6. **Categorize and place.** `listCategories` to find the category, then
   `updateVideo` to set it. `createPlaylist` if needed, then
   `PUT /playlists/{id}/add_videos` (`addVideosToPlaylist`).

## Reversibility — read before deleting

`deleteVideo` is **terminal**. No restore operation exists anywhere in the Zype contract.
The reversible alternative is `updateVideo` with `active: false` — list operations accept
`active=true|false|all`, so a deactivated video is still reachable. Prefer deactivation;
only call `deleteVideo` on an explicit human instruction that names the video.
