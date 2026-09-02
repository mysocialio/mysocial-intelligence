# Product tool contracts

## list_brand_products

Input: optional `identityId`. Omit it to list across every identity.

Output: `products` (cheapest-first) and `counts` (`total`, plus `byStatus`).

Each product carries `productId`, `identityId`, `name`, `kind`, `status`, `tagline`, `priceLine`, `audience`, `funnelRole`, and `relations` resolved to names.

`priceLine` is rendered by the server (`$149 one-time`, `$99/mo · $990/yr`, `$7,500–$15,000`, `Free`). Quote it as-is. Do not reformat prices or recompute them from the raw fields — the rendered line is what the creator sees in their own UI, and divergence between the two is a bug you would be introducing.

## get_brand_product

Input: `productId`.

Output: `product` (the full spine plus `document`, `version`, `updatedAt`), `sources`, `media`.

`document` is the living body: ordered blocks of `text`, `image`, `carousel`, and `source`. `version` is what you pass back as `expectedVersion` when updating.

## create_brand_product

Input: `identityId`, `name`, and any of `kind`, `status`, `tagline`, `pricing`, `audience`, `funnelRole`, `links`, `relations`, `body`, `document`, `idempotencyKey`.

`body` is shorthand for an opening text block. Use it unless you need images or several blocks.

Output: `productId`, `name`, `kind`, `status`, `priceLine`.

Reuse the same `idempotencyKey` when retrying so a repeat does not file a duplicate offer.

## update_brand_product

Input: `productId`, `expectedVersion`, and only the fields you mean to change.

Output: `productId`, `name`, `kind`, `status`, `priceLine`, `version`.

Partial by design: an omitted field is left alone. `links`, `relations`, and `document` are the exception — each replaces its whole list, so send the full set you want, including the entries you are not changing.

Failure modes and the correct response:

| Error | Meaning | Do |
|---|---|---|
| `version conflict: this product is now at version N` | The creator or another agent edited since your read | Re-read with `get_brand_product`, re-derive your change against the new state, retry with `expectedVersion: N` |
| `Brand product not found` | Wrong id, or not this creator's product | Re-list with `list_brand_products` |
| `Identity not found` | Wrong `identityId` on create | Re-list with `list_brand_identities` |

Never blind-retry a conflict with the same payload.

## Field shapes

### pricing

One object covers every presentation. Pick the `model` that matches reality:

| `model` | Use for | Fields that matter |
|---|---|---|
| "free" | Lead magnets, free tiers | none |
| "one_time" | Digital products, one-off purchases | `amount` |
| "subscription" | Recurring plans | `amount`, `interval`, and `variants` for an annual alongside the monthly |
| "range" | Quoted engagements | `range.min`, `range.max` |
| "custom" | Negotiated or partnership-shaped | `note` |

`currency` defaults to `USD`. `note` is a qualifier shown with the price, e.g. `launch price; standard $199`. Amounts are never negative; `0` means free.

```json
{
  "model": "subscription",
  "amount": 99,
  "currency": "USD",
  "interval": "month",
  "variants": [{ "label": "Annual", "amount": 990, "interval": "year" }]
}
```

### relations

`[{ type, productId, note }]`, targets under the same identity. Types:

| Type | Reads as |
|---|---|
| "upsell" | buyers of this move up to that |
| "cross_sell" | these sell alongside each other |
| "next_step" | the natural thing after this |
| "included_in" | this is bundled inside that |
| "self_serve_version" | the cheap do-it-yourself counterpart |
| "done_with_you_version" | the high-touch counterpart |

Dangling targets are dropped on read, so a relation to a deleted product disappears rather than breaking.

### links

`[{ label, url, kind }]` where `kind` is "sales_page", "checkout", "billing", or "other". URLs must be absolute `http(s)`; anything else is silently dropped.

### document

`{ blocks: [...] }`, replacing the whole body. Block shapes:

- `{ type: "text", text }` — prose
- `{ type: "image", assetIds: [id] }` with optional `caption`
- `{ type: "carousel", assetIds: [id, id], caption }`
- `{ type: "source", assetId }` — an attached source document

Keep a block's `id` when editing it in place. Image, carousel, and source blocks may only reference assets that already exist on that product — there is no upload path here, so never invent an asset id. To add images, ask the creator to drop them into the product in the web app.
