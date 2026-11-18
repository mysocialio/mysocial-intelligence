# Evidence contracts

What each call returns, what the labels mean, and what is deliberately absent.

## Retrieval scope, in order

| Level | Calls | What it covers |
|---|---|---|
| Personalized | `universe_content`, `search_universe` with `topicKeywordId` / `clusterId` / `topic` | posts discovered for this creator's own researched topics, ranked for them |
| Broader stored | `search_universe` (no topic), `get_universe_post` | every discovered post Mysocial holds, any topic, any customer's research |
| Live discovery | `search_creators`, `search_social`, `transcribe_video_url` | provider calls at request time; searches spend the plan's monthly research window, an audio transcript the video-processing allowance |

Empty at one level is a fact about that level only. `search_creators` replies carry `failed[]` (platforms that errored) and `fromStorage`; `search_social` says how it ran in `searched` and why a list is empty or short in `note`. A `limit_reached` status is a plan gate to relay, not an empty market.

## Live public posts (`search_social`)

- One platform per call: `tiktok`, `youtube`, `instagram`, `x` or `threads`. Instagram searches the query as one hashtag (`searched.hashtag`); Threads searches the exact phrase and returns one sampled window (`searched.sampled: true`).
- Each row: `url`, `author` (handle), `authorName` when it differs, `text` (cut at 280 characters), `format` (`video`, `image`, `carousel` or `text`), `metrics`, `durationSeconds`, `createdAt`. `metrics` holds only what the platform reported: an absent count is unknown, never zero, and a reported 0 is a real 0. YouTube search reports views only; X reports views, likes, replies (`comments`), reposts, quotes and bookmarks.
- `searched`: `provider` (`rapidapi` or `socialcrawl`), `days` (the window actually applied: YouTube widens 90 and 180 days to a year, Instagram has none and says so in `note`), `page`, and `fallbackFrom` when X was served by its backup.
- Paging: `nextCursor` while there is more; pass it back as `cursor` with the same platform and query (a cursor from another search is refused). A page is the provider's whole page, never sliced. Posts an earlier page showed are left out and counted in `note`.
- Cost: each page that brings new posts is one action from the monthly research pool; an empty page or a page of repeats is free. `allowance.searchesRemaining` comes back with every answer.
- A row is a candidate. It carries no creator baseline: Instagram, TikTok and YouTube authors get one through `get_channel_posts`; X and Threads authors have none.

## Creator and catalogue

- `search_creators` hits: `entityId`, `accountId` (null when the provider gave no stable identity), `postsMatched`, `topicViews` and `medianTopicViews` (reach on this topic only), `coverage`, `next`.
- `get_entity`: `research[]`, one row per channel with `accountId`, `coverage`, `next`.
- `get_channel_posts` (by `accountId`, `url` or platform + handle): posts with `comparison`, plus `catalog`, `coverage`, `next`. `status: not_found` means nothing is held; `next.collect` is the `refresh_channel` call.
- `refresh_channel` with `url` or platform + handle: collects the profile and about 25 recent posts once (one research action; 24-hour cooldown; `status: queued` then read `get_entity` or retry `get_channel_posts` after about a minute). It files the creator as search memory (`leadState: seen`), never as a saved lead, and never starts tracking. `identityStatus: pending` means the provider has not yet confirmed the profile exists.

### Coverage

`coverage` has four independent parts. Do not read one as another.

- `profile.refreshedAt` - when the profile (followers, bio) was last fetched.
- `baseline.builtAt`, `baseline.postCount` - when the recent-posts sample was collected and how many posts it had.
- `catalogue.status` (`none` | `sample` | `walking` | `partial` | `complete`), `catalogue.basis`, `postsHeld`, `oldestAt`, `newestAt` - what Mysocial holds. `sample` is about 25 recent posts; `complete` is a tracked walk up to three years.
- `metrics.observedAt` - when the held posts' metrics were last read. Metrics are lifetime counts at that moment, never a time series.
- `collection.status` (`idle` | `queued` | `running` | `ready` | `error`) with `message` - a running or failed collection leaves the previous snapshot readable.

## Comparison (outlier evidence)

Every post-level `comparison` has the same fields: `metric`, `value`, `observedAt`, `baseline { kind, median, sampleSize, basis, windowDays, computedAt, platform, format, eligibility }`, `method`, `multiple`, `verdict`, `limitations[]`.

- `verdict`: `above_baseline` (at least 1.2x), `at_baseline`, `below_baseline` (at most 0.8x), or `insufficient_evidence` (no baseline, no value, or fewer than 8 posts in the sample). With `insufficient_evidence` a `multiple` may still be present as arithmetic; quote it only with its limitation.
- `basis`: `trailing_window` - the creator's own posts of the same format in the 90 days before the post (needs a walked catalogue; the strongest read). `recent_sample` - a one-off sample of about 25 recent posts, lifetime metrics, not age-matched. "topic_cohort" - other creators' posts on the same topic and platform (`cohortComparison`), which says something about the topic, not about this creator's form.
- `reach` is the raw count on its own. High reach with `insufficient_evidence` is "a big post", not "an outlier for this creator".
- Views on different platforms are different units. Compare multiples across platforms, not views; compare views only within one creator on one platform.
- Nothing here is a probability of virality, and no field estimates one.

## Post evidence

`transcribe_video_url` (any supported post URL; `save: false` for a pure read) and `get_content_idea` (a saved source) return evidence kept apart:

- `caption` - the post's own text. Never a transcript.
- `transcript` + `transcriptStatus`: `spoken` (verbatim spoken words; `transcriptSource` says `captions` or `audio`), `no_speech_detected` (a video nobody speaks in: the opening is visual, on-screen or in the caption), `transcription_deferred` (the allowance is spent; `gate` says which; the rest of the evidence is returned), `transcript_failed` (retryable), `not_applicable` (image, carousel, thread), `unavailable` (nothing fetched).
- `comments[]` - top comments by likes (over 5 likes or over 5 replies, max 20), each with `author`, `likes`, `replies`. `commentsStatus` says why the list is empty when it is.
- `media` - `thumbnailUrl`, `images[]` (carousel slides in order, durable copies), `mediaUrl` (the platform's video URL; not stored). `includeImages: true` attaches the pictures so you can look at the opening; a video attaches its cover still only.
- `metrics` + `metricsObservedAt` - the platform's counts at fetch time.
- `creator` - `held` and the `next` calls; `comparison` against the held creator's baseline when there is one.
- `get_universe_post` has no transcript and no comments (`evidence.transcript: not_stored`); `evidence.readWith` is the `transcribe_video_url` call.

On-screen text is not extracted for external posts. When the opening is visual, attach the images and describe what you see as your own observation.

## Saving and the import contract

- `save_content_idea` with `url` imports the post through the same pipeline as sharing it into the app and returns `import`: `state` (`saved` | `processing` | `complete` | `partial` | `failed`), `available[]`, `pending[]`, `unavailable[{ field, reason }]`, `retryable`, `retryWith`, `readWith`. `complete` means every supported field landed for this post; it never means every optional field exists. A re-save of the same URL within 30 minutes returns the same idea (`reused: true`) and retries a failed or deferred source in place.
- `update_content_idea`: `retryImport` (same source row, never a duplicate), `research` (your references, hypothesis, variant, successMetric, authoredBy - stored as interpretation with authorship), `published` (platform + postId of the creator's own post that executed the idea).
- `get_content_idea`: every source with `status`, the separated evidence above, `import`, `research`, `published` (with `readWith` for the post's metrics via `get_content`).
- Both writes require explicit user approval (`userApproved: true` after a yes in the conversation).

## Own-account intelligence

`get_strategy_context` (opens with its own playbook slice), `get_hit_dna`, `search_content`, `get_content`, `get_topics`, `get_audience`. Measured fields are platform data; `mechanism`, `audienceReward`, psychology and segments are stored model hypotheses with their own labels. Read their `coverage` and `warnings`.
