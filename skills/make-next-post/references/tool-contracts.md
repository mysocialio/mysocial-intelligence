# Mysocial Tool Contracts

## Evidence tools

- Call `get_strategy_context` first (`task: create_post`). Pass `platform`, `topic`, `draftText`, and `objective` when known. Its retrieval scores rank evidence only; they are not virality predictions. `get_hit_dna` describes recurring patterns associated with measured hits; `search_content` finds precedents and siblings.
- Compare candidates yourself. `check_if_idea_already_posted` scores one candidate against everything already posted; `search_content` with a phrase finds its precedents. Nothing on the server chooses a winner.
- Use `get_content` with `include: ["full_content", "post_dna"]` when compact evidence is insufficient to recover the argument. Add `comments`, `media` or `siblings` when needed. Where the v2 contract is available, explicit `responseVersion: 2` exposes revision-scoped evidence; include `audience_response`, `performance`, `sharing` or `relationships` as needed. Keep v1 usable and never infer absent v2 fields from legacy labels.
- Preserve returned post refs, source IDs, revisions, coverage and resource links. Follow a returned transcript resource if response clipping hides the decisive condition or ending. Hypothetical sharing in Post DNA is separate from actual comments and metrics. Reads do not trigger fresh analysis; missing or stale evidence stays a limitation.

## External research tools

- Use `search_tiktok` and `search_youtube` for current platform evidence.
- Transcribe at most two standout results with `transcribe_video_url`.

## Write tools

- Call `save_content_idea` only after explicit approval, with the complete package as `idea` (and `images` for a carousel). There is no separate draft object.
- Call `update_content_idea` only after the user approves a change to a saved idea (status, planned day, title, text, images).

Never use deprecated tools that generate ideas or return a final virality verdict.
