# Hinglish EMI Collections Voice Agent

A production-shaped collections voice agent for an Indian NBFC — Hinglish, RBI-constrained, deployed on
[Bolna](https://bolna.ai), with the red-team suite written **before** the happy path.

Five adversarial cases verified passing against the live agent, including the two that matter most:
do-not-call honoured with no retention attempt, and no consequence named when the customer asks what
happens if they don't pay. See [RESULTS.md](./RESULTS.md).

---

## Why collections

It is the hardest voice-agent use case in India, and the reason is not technical. The constraints are legal.

An agent that closes 20% more promises by implying legal consequences is not a better agent — it is a
compliance incident under the RBI Fair Practices Code for recovery agents. So the interesting engineering
lives in the refusals, not in the script.

That produces one concrete design decision that drives everything else: **the guardrails are ranked above
the call objective in the prompt**, and the prompt says so explicitly — *"violating one is worse than
failing the call."* Most collections prompts put the objective first and constraints in a footer. The model
then optimises for the objective under pressure, and starts hinting.

---

## What's here

| File | |
|---|---|
| [`bolna-ready/`](./bolna-ready) | Paste-ready config for the Bolna dashboard, tab by tab |
| [`bolna-ready/02-system-prompt-EN.txt`](./bolna-ready/02-system-prompt-EN.txt) | The prompt, in Bolna's Personality / Context / Instructions / Guardrails structure |
| [`bolna-ready/tool-*.json`](./bolna-ready) | Four function-calling tools, OpenAI spec + Bolna's `custom_task` extension |
| [`bolna-ready/04-DEPLOY-STEPS.md`](./bolna-ready/04-DEPLOY-STEPS.md) | Deploy walkthrough, ~20 minutes |
| [`red-team-suite.md`](./red-team-suite.md) | 22 adversarial cases: RBI compliance, truthfulness, robustness |
| [`RESULTS.md`](./RESULTS.md) | What was run, what passed, what is still untested |
| [`call-flow.md`](./call-flow.md) | State diagram + interrupt handlers that outrank every state |

The four tools post to a webhook catcher rather than a real backend, which means **every tool invocation is
inspectable** — when `mark_do_not_call` fires you can read the POST and see the customer's verbatim words
in it. That is how the guardrails were verified rather than assumed.

---

## Design decisions worth arguing about

**The refusals are the product.** Ten guardrails, ordered above the instructions, with an explicit
statement that breaking one is worse than failing the call.

**Language switching was deleted, not added.** The first draft had elaborate Hinglish mirroring rules.
Bolna's docs say plainly: *"Don't include language-switching instructions in it; switching is automatic."*
The prompt was fighting the platform, so the rules came out and Bolna's per-language tabs do the work.
Reading the docs and removing instructions is usually the better move.

**No number is ever spoken unless it came from a tool result or the customer.** The failure that ends
careers is a confident wrong figure. A tool failure must not silently fall back to a prompt variable —
case B4 exists only to catch that, and is still untested.

**Do-not-call fires before persuasion.** The natural model reflex is one more attempt before letting go.
That attempt is the violation. Tested in Hindi (A5, passing); the English-mid-Hindi variant (A6) is next.

**Voice constraints are prompt constraints.** 25-word turns, no markdown, digit-by-digit reference numbers,
amounts spoken as words. A prompt written for chat sounds wrong the instant it is spoken aloud — TTS
reading "Rs 4,200" is the giveaway.

---

## Deploy it yourself

See [`bolna-ready/04-DEPLOY-STEPS.md`](./bolna-ready/04-DEPLOY-STEPS.md). You need a Bolna account
(free tier is enough) and a [webhook.site](https://webhook.site) URL. About 20 minutes.

## Licence

MIT. The prompt is a starting point, not legal advice — if you deploy a collections agent in India, have
someone who knows the RBI Fair Practices Code read your guardrails.
