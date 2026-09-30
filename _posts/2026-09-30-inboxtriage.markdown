---
layout: post
title: Talking My Way Out of a SaaS
comments: true
---

My inbox has the same problem everyone's inbox has: real mail buried under a rising tide of
newsletters, marketing blasts, and "digest" mail I never asked to be digested. The obvious fix —
filter out anything containing the word "unsubscribe" — is also a terrible one. Receipts, shipping
notices, security alerts, and bank statements are all legally required to carry an
unsubscribe/manage-preferences link too. Filter on that text and you bury the mail that actually
matters right alongside the mail that doesn't.

I've been leaning on [Claude][learning] more and more for side projects like this, and I've learned
not to open with "build me a thing." I open a conversation and think out loud instead. This one
started with no code at all — just describing the mess and asking what shape a fix could even take.

## "Just use Gmail's filters"

The obvious answer, before any of this, is that Gmail already has filters built in. No new code, no
Apps Script, no problem to solve. That's true right up until you've actually tried to build
something nontrivial out of them. Have you? It's awkward and cryptic — a UI built for "if from
contains X, do Y," not for header-based scoring, not for "classify this once and remember the
decision," not for anything that has to reason about a message instead of just pattern-matching it.
I already had a pile of hand-built filters from years of doing exactly this by hand, and the honest
state of that pile was: I couldn't tell you anymore which ones still made sense. That pile of
filters wasn't a solution I'd overlooked. It was the mess I was trying to get out from under.

## Option one: make it a SaaS

The first idea either of us reached for was the biggest one: a real product. Sign up, connect your
Gmail, get a dashboard, subscription tiers, the works. It's the default shape my brain jumps to for
"tool that solves a problem I clearly share with other people."

Talking it through is what killed it, not the idea itself. Anything that inspects
`gmail.settings` or `gmail.modify` on someone else's account needs Google's OAuth app
verification, and for those scopes specifically, that means a CASA security assessment — an
outside audit, a real cost, a real delay, before a single other user could touch it. Add a billing
layer and multi-tenant infrastructure on top and I wasn't looking at "a version of my tool" anymore.
I was looking at a different piece of software that happened to share a classifier with it. Worth
knowing, maybe, if this ever proves itself. Not worth building on day one, when I don't even know
yet if the classification is any good.

## Option two: a browser extension

Next idea: skip the backend entirely, run it client-side as a Chrome extension watching the Gmail
tab. No server to host, no OAuth client of my own to register.

That one didn't survive contact with Gmail's actual feature set either. Google already ships
Workspace Add-ons — a first-party way to put a sidebar next to any open message, `CardService` UI
and all, with none of the plumbing a Chrome extension drags in: no separate manifest, no content
scripts fighting Gmail's DOM, no OAuth client to stand up and maintain. Everything an extension
would buy me, the platform already had, minus the parts I'd have had to build and keep working
myself.

## What was left

Once a SaaS and an extension were both off the table, what remained was smaller than either: a
Google Apps Script project, container-bound to a single Sheet, running under my own Google
account. No separate OAuth client, no store listing, nothing distributed to anyone else's account —
which means the entire verification requirement that killed option one simply doesn't apply. It
runs as me, on a timer, and the Sheet is both the control surface and the audit trail.

It's the least impressive-sounding architecture of the three. It's also the only one I could
actually ship that week.

## Building the thing we'd decided on

With the shape settled, the conversation shifted from "what should this be" to "how does
classification actually work." The signal we landed on was the `List-Unsubscribe` header (RFC
8058) — mail clients already use it to separate bulk mail from transactional mail, and unlike body
text, it can't be spoofed by an innocent word choice in a receipt. It's not perfect alone, though —
Nextdoor digests turned out to have no `List-Unsubscribe` header at all, which sent us back to add
weaker corroborating signals: a `no-reply`-style sender, `Precedence: bulk`, other RFC 2369
`List-*` headers, mass-mailer fingerprints like `X-SFMC-*` (Salesforce Marketing Cloud). One strong
signal is enough on its own; otherwise it takes two weak ones agreeing, so a `no-reply` 2FA code
doesn't get swept away with the actual noise.

## The path a message actually takes

Every incoming message gets tagged `Unscanned` the instant it arrives — that part is a plain
native Gmail filter, no script involved, so nothing can slip in ahead of it. Every ten minutes, the
script works through whatever's sitting in `Unscanned` and scores it against the header signals
above. Clean mail just has the tag removed and goes on with its life in the Inbox, no trace left
behind. Mail that fails the check gets the `Unscanned` tag swapped for `Suspect` and gets archived
out of the Inbox — held, not hidden, since it's still sitting right there in `All Mail`.

From there it's on me. I open a message tagged `Suspect`, open the panel on the right that the
add-on puts next to it, and pick Allow, Block, or File. Allow clears the sender and puts the
message — and every other `Suspect` message from them — back in the Inbox. Block and File both do
their work through Gmail's own filters rather than anything the script has to keep enforcing:
picking one creates or updates a real Gmail filter for that sender, skip-the-inbox plus a label, so
every future message from them is routed straight there by Gmail itself — the script just has to
recognize the sender is already decided and leave it alone. File just points that filter at a label
of my choosing instead of `Blocked`. I'd assumed Allow and
Block would be the whole model here. GitHub notifications broke that within the first day of real
use — I don't want them in my Inbox, but I don't want them blocked either, I want them organized.
They also carry no `List-Unsubscribe` header at all, so they'd never even reach `Suspect` on their
own. That's why the panel isn't gated to flagged mail — it offers all three options on *any* open
message, because the classifier missing a sender entirely turned out to be a normal case, not an
edge one.

## Bugs worth remembering

A few things went sideways in ways I'll probably repeat if I don't write them down. The GitHub
Actions deploy step corrupted a credential secret by interpolating it directly into a quoted shell
string — GitHub substitutes secrets as raw text before bash ever parses the line, so the JSON's own
embedded quotes broke it. Passing the secret through `env:` instead fixed it. Separately, adding a
new OAuth scope to the manifest doesn't take effect on the next script push — it requires manually
re-running Test Deployment ▸ Install in the Apps Script editor, a step I discovered by hitting the
same permissions error twice before it stuck.

## The part still deliberately unbuilt

The SaaS question didn't fully go away — it just moved to the end of the conversation instead of
the start. Once the tool was live and working, the same three options came back up: sell the
template as-is, gate copies behind a license and a small backend, or go full multi-tenant with its
own OAuth client and billing. The recommendation I landed on with Claude was to not decide yet —
let classification run against real mail for a few weeks first, because accuracy is the actual
risk here, not the business model. Deciding a monetization strategy for a filter that might still
misfile someone's bank statement would be solving the wrong problem first.

There's a migration tool for my existing hand-built filters, fully designed and still unwritten,
for the same reason. Some things are more honest half-built and clearly labeled that way than
finished and premature.

---

*This post was drafted with the help of Claude.*

[learning]: /2026/03/22/learning-to-use-claude.html
