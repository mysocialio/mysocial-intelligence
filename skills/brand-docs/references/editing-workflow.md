# Brand doc tool contracts

## read_brand_source

Input: `identityId`, `sourceId`, optional `offset` / `maxChars` (default 20,000 chars per call), optional `version`.
Output: `text`, `nextOffset` (paginate until null), `totalCharacters`, `version`, `currentVersion`.

Read the region you plan to edit in full. `currentVersion` is the value to pass as `expectedVersion` when updating.

## update_brand_source

Input: `identityId`, `sourceId`, `expectedVersion`, `editSummary`, and exactly one of:

- `edits`: ordered array of `{ oldText, newText, replaceAll? }` (max 50)
- `replaceAllText`: full replacement Markdown (restructures only)

Output: `previousVersion`, `version`, `additions`, `deletions`, and `diff` (a unified patch — show it to the user when the change matters).

Failure modes and the correct response:

| Error | Meaning | Do |
|---|---|---|
| `oldText not found` | The text differs from what you assumed | Re-read the document, copy `oldText` exactly (whitespace matters) |
| `oldText matches N places` | Ambiguous match | Extend `oldText` with surrounding lines, or set `replaceAll: true` only if every occurrence should change |
| `version conflict: the document is now at version N` | Someone else edited since your read | Re-read, re-derive edits against the new text, retry with `expectedVersion: N` |
| `No changes` | Your edits produce identical content | Skip the call; nothing to save |

## create_brand_source

Input: `identityId`, `filename` (kebab-case, `.md` appended if missing), `markdown`, optional `caption`.

The caption is what future agents see in the source catalog — describe purpose, not contents: "Hook patterns that outperform, with evidence" beats "hooks doc".

## Worked example

1. `read_brand_source` → `currentVersion: 7`, text contains `## Hooks\n\n- Curiosity gap openers work well`
2. `update_brand_source`:

```json
{
  "identityId": "…",
  "sourceId": "…",
  "expectedVersion": 7,
  "editSummary": "Add evidence: contrarian hooks outperform curiosity gaps on Reels",
  "edits": [
    {
      "oldText": "## Hooks\n\n- Curiosity gap openers work well",
      "newText": "## Hooks\n\n- Curiosity gap openers work well\n- Contrarian openers ('Everyone is wrong about…') outperformed curiosity gaps 3.1x on Reels (Aug 2026 top-posts analysis)"
    }
  ]
}
```

3. Verify the returned `diff` shows only the intended `+` line, then report the change and its revision number to the user.
