# The sales motion

How Mysocial runs a relationship from first touch to won, whatever the
motion: a creator pitching a brand, a brand courting a creator, an agency
or consultant chasing a client, a founder selling software. Learned from a
founder selling to CMOs off an early alpha, and kept because it is simple.
Segment framings for the message itself are in segment-pitches.md.

## Four stages, and they are the progress bar

`saved (prospecting) -> contacted -> meeting -> working (won)`

| Stage | Means | You record it with |
|---|---|---|
| saved | prospecting: chosen, being researched, brief and route being built | `save_leads`, `add_brand`, `add_creator` |
| contacted | we are in contact, whichever side opened | `log_touch` (moves it for you, outbound or inbound) |
| meeting | a call or demo is booked or was held | `log_touch` or `update_lead` status `meeting`, `nextActionAt` = the slot |
| working | won: they paid, signed, or the collab is on | `update_lead` status `working` |

A reply is a touch, not a stage - `log_touch` inbound stamps `lastReplyAt`
and the row shows it. Qualification is context (`add_note` with kind `context`), not a
stage. A trial, a gifted post, a pilot project on the way to won is a note
and a reminder, not a stage. Never move a lead backwards. Never move
`archived` / `rejected` without the user saying so.

## The light CRM: four things, nothing more

1. **Who and how to reach them** - `find_contacts`, routes on the entity.
2. **When we last heard from each other, and on which channel** - every
   call, email, DM, reply and demo goes through `log_touch`. That is what
   stamps `lastContactedAt` / `lastReplyAt` / the channel. A bare `add_note`
   does not, and then the guard below cannot protect anyone.
3. **A reminder** - "call back in a week" is `nextAction` + `nextActionAt`.
   `list_leads due:true` is the day's queue. Logging a touch retires a
   reminder that was due; set the next one in the same call.
4. **The guard** - `get_lead.touch.warning` and `log_touch.guard` name a
   recent outbound with no reply since. Read them before drafting. The rule
   the whole system exists for: nobody gets contacted twice by accident.

## Sourcing and the first call

Pick the ICP first: who gets the most out of it? The company or founder
whose content is a real acquisition channel and who has a decision maker
who posts. One market first, then out. The user calls; you
prepare. The call is two to three minutes to a CMO / VP Growth / founder,
so the opener is four beats and nothing else:

1. who I am and where I am calling from
2. what we build, in one plain sentence - what it does for them, not the category
3. why now - the concrete reason (a cohort closing, a round closing, a launch)
4. why them - one true thing about their content, and the invite

Then answer questions; the ask is a demo, not a deal. Expect the first
five or six calls to be rough - log every objection verbatim with
`log_touch`; they are the cheapest product research there is. Never open
with "AI social media analytics".

## Proof compounds: the cohort

Once a few customers exist, the list is the pitch. `list_leads status
"working"` is the proof, and only stored names may be used - never invent
a logo. The frame: *these names in your niche are already in; the terms
are small; should you not be too?* Draft it with the `pitch` preset and
say it is a proof-led invite.

## The demo

Forty minutes, no longer, **with their material**. Before the demo ask for
their channels (or briefs and content, if that is what they work with) and
attach them - `add_creator` / `link_channel` / `add_brand` - so the demo
shows their own outlier post, their own pattern, their own gap. People
relate to their own content and to nothing else. Close with zero friction:
the workspace is already created, their content is in, they can start
today. The `prep` preset writes this plan.

## The trial

Whatever the motion calls it - a pilot, a gifted post, a test collab, a
scoped project - the terms are the same: short, free or nearly, and the
only price is feedback and a one-liner to quote. Log it as a touch with a
reminder for the last week (`log_touch` ... `nextAction: "paid
conversation"`), stay close, build what they ask for. Record the one-liner
verbatim with `add_note` kind `context` when it lands. Every trial should
end at `working`.

## Build with them

The product is built with customers. What they use stays; what nobody
uses goes. Keep the actionable, cut the nice-to-have. Their feedback from
the pilot is the roadmap - put it on the lead's timeline, and when it is
durable, in Brand Studio.
