# Call flow

```
                          ┌──────────────┐
                          │  CALL START  │
                          └──────┬───────┘
                                 │
                     ┌───────────▼────────────┐
                     │  IDENTIFY              │
                     │ "Kya main {name} se    │
                     │  baat kar rahi hoon?"  │
                     └───┬────┬─────┬─────┬───┘
           yes ──────────┘    │     │     └────────── voicemail
                              │     │                      │
              "kaun hai?" ────┘     └──── "wrong number"   │
              (third party)                    │           │
                    │                          │           │
                    ▼                          ▼           ▼
          ┌──────────────────┐      mark_wrong_number   end_call
          │ NO DISCLOSURE    │           + end        (no details left)
          │ "personal matter"│
          │ ask availability │
          └────────┬─────────┘
                   └──► schedule_callback OR end_call
    
    (identity confirmed)
           │
           ▼
    ┌──────────────────────────────┐
    │  STATE THE POSITION          │
    │  amount + due date, once     │
    └──────────────┬───────────────┘
                   │
                   ▼
    ┌──────────────────────────────────────────────────────┐
    │  ASK: "Kab tak payment kar paayenge?"                │
    └──┬─────┬─────┬──────┬──────┬──────┬──────┬──────┬────┘
       │     │     │      │      │      │      │      │
       │     │     │      │      │      │      │      └── abusive
       │     │     │      │      │      │      │            → 1 empathy line
       │     │     │      │      │      │      │            → end_call(abusive)
       │     │     │      │      │      │      │
       │     │     │      │      │      │      └── "stop calling"
       │     │     │      │      │      │            → mark_do_not_call
       │     │     │      │      │      │            → end IMMEDIATELY
       │     │     │      │      │      │              (no retention attempt)
       │     │     │      │      │      │
       │     │     │      │      │      └── "already paid"
       │     │     │      │      │            → mark_dispute(claims_paid)
       │     │     │      │      │            → end politely, no argument
       │     │     │      │      │
       │     │     │      │      └── disputes amount
       │     │     │      │            → mark_dispute(amount)
       │     │     │      │            → offer transfer
       │     │     │      │
       │     │     │      └── wants settlement / restructure
       │     │     │            → transfer_to_human(settlement_request)
       │     │     │              NEVER offer a number yourself
       │     │     │
       │     │     └── "can't talk now" / driving
       │     │           → schedule_callback (08:00–19:00 IST only)
       │     │
       │     └── vague: "jaldi", "is week"
       │           → ask ONCE for a specific date
       │           → still vague: record promise_date=null
       │                          + verbatim notes
       │
       └── specific date given
             │
             ▼
       ┌─────────────────────────┐
       │  CONFIRM AMOUNT         │
       │  full or partial?       │
       └───────────┬─────────────┘
                   ▼
       ┌─────────────────────────────────────┐
       │  record_payment_promise             │
       │  then READ IT BACK:                 │
       │  "{date} ko {amount} rupaye. Sahi?" │
       └───────────┬─────────────────────────┘
                   │
          no ──────┴────── yes
           │                │
           ▼                ▼
     re-ask once      end_call(promise_recorded)
```

## Interrupt handlers — active in every state

These outrank whatever state the call is in:

| Trigger | Action |
|---|---|
| "stop calling" / "don't call" / equivalent, any language | `mark_do_not_call` → end. No persuasion. |
| "are you a bot/AI/recording?" | Answer yes immediately, then resume the current state. |
| Request for a human | `transfer_to_human(customer_requested)`. No retention attempt. |
| Asks for a number the agent does not hold | "Mere paas abhi wo detail nahi hai" + transfer offer. Never estimate. |
| Silence 5s | One "Hello, aap sun paa rahe hain?" → second silence → `end_call(no_response)` |
| Customer interrupts | Stop mid-word, listen. |
| Language the agent cannot hold | `transfer_to_human(language_mismatch)` |

## Exit codes

`promise_recorded` · `callback_scheduled` · `dispute_raised` · `do_not_call` · `wrong_number` ·
`no_response` · `abusive` · `completed`

Success metric is **not** promise rate. It is promise rate with zero Block-A red-team failures —
a promise obtained by implying a consequence is a negative outcome, not a positive one.
