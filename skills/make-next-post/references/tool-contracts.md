# Mysocial Tool Contracts

## Evidence tools

- Call `get_post_creation_context` first. Pass `platform`, `topic`, `draftText`, and `objective` when known. Its retrieval scores rank evidence only; they are not virality predictions.
- Call `compare_post_candidates` with two to five candidates. Preserve each `candidateId` when referring to results. The tool does not choose a winner.
- Use `read_post_transcript`, `read_post_comments`, and returned resource links only when the compact evidence is insufficient.

## External research tools

- Use `search_tiktok` and `search_youtube` for current platform evidence.
- Transcribe at most two standout results with `transcribe_video_url`.

## Write tools

- Call `save_post_draft` only after explicit approval. Save the complete package and the evidence references used to create it.
- Call `revise_post_draft` only after the user approves a revision to an existing saved draft.

Never use deprecated tools that generate ideas or return a final virality verdict.
