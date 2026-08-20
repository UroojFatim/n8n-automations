# 03 — Gmail Auto-Triage

Classifies incoming Gmail messages into categories and applies the matching label, using regex rules over the subject line, the sender domain, and Gmail's own category labels.

Labelling only — nothing is deleted, archived, or forwarded. Every action this workflow takes is reversible from the Gmail UI.

## Workflow

![Gmail Auto-Triage workflow in n8n](./screenshot.png)

```
Manual Trigger  →  Get Many Messages  →  Switch  ┬─ Academic ────→ Label Academic ────┐
                        (Gmail)         (Rules)  ├─ Jobs ────────→ Label Jobs ────────┤
                                                 ├─ Newsletters ─→ Label Newsletters ─┼→ Triaged
                                                 └─ Fallback ────→ Label To Review ───┘
```

## Routing rules

| Output | Matches on |
|---|---|
| Academic | Subject contains coursework terms (`assignment`, `exam`, `result`, `semester`, `transcript`, `quiz`, `attendance`, `course`, `lecture`) **or** sender domain ends in `.edu` / `.edu.pk` |
| Jobs | Subject contains `interview`, `application`, `hiring`, `opportunity`, `recruiter` |
| Newsletters | Subject contains `newsletter`, `unsubscribe`, `digest`, `weekly` **or** Gmail has already tagged it `CATEGORY_PROMOTIONS` / `CATEGORY_SOCIAL` / `CATEGORY_UPDATES` |
| Fallback | Everything else → `To Review` |

Rules are evaluated top to bottom and the first match wins, so specific rules sit above general ones. The Newsletters rule is deliberately last of the three — it is broad enough to swallow job alerts from LinkedIn, which arrive tagged `CATEGORY_UPDATES` and belong under Jobs.

## Results on a 50-message sample

| Label | Messages |
|---|---|
| Newsletters | 41 |
| Jobs | 5 |
| To Review | 3 |
| Academic | 1 |

## Concepts demonstrated

**Switch vs. IF.** An IF node gives two branches. A Switch gives as many as you define, each with a named output, which keeps the canvas readable once there are more than two paths.

**Fallback outputs.** Anything matching no rule is routed to an explicit extra output rather than being dropped. Without it, unmatched items disappear with no error and no trace.

**Boolean rules over `contains` rules.** Rather than one `contains` rule per keyword, each rule evaluates a single regex returning a boolean. One line covers a whole category, the `i` flag handles casing, and adding a keyword means adding `|word`.

**Reading the platform's own signals.** Gmail has already classified most mail into Promotions, Social, and Updates. Matching those existing labels is far more reliable than guessing at subject keywords, and it comes free in the message payload.

**Branch convergence.** All four label branches feed a single No-Op node. A Merge node is for joining *different* data streams; one stream that fans out and comes back together just needs the connections pointed at a common node.

**Development triggers vs. production triggers.** A Gmail Trigger fires only on newly arriving mail, which makes iteration painfully slow. Building against a Manual Trigger plus *Get Many Messages* lets you replay the same 50 real messages until the rules are right; the trigger gets swapped in at the end.

## Setup

Requires the **Gmail API** enabled in a Google Cloud project and a Gmail OAuth2 credential in n8n. The same OAuth client used for Google Sheets in Project 02 works here — OAuth clients belong to a project, not to a single API.

1. Create four labels in Gmail: `Academic`, `Jobs`, `Newsletters`, `To Review`

   Gmail reserves its own category names, so `Social`, `Promotions`, `Updates`, `Forums`, and `Primary` cannot be used as user label names.

2. Import `workflow.json`
3. Open each Gmail node, connect your credential, and reselect the label from the dropdown — label IDs are account-specific and the ones in this file are placeholders
4. Run with **Execute workflow**

## Going live

Replace the Manual Trigger and *Get Many Messages* nodes with a **Gmail Trigger** (Poll Times: every 15 minutes, Simplify on), wire it into the Switch, and publish. New mail is then triaged as it arrives.

## Debugging notes

Every failure in this project was silent. Nothing errored, every node showed a green check, and the workflow did nothing — which is a harder class of bug than a stack trace.

**A capitalised field name.** With *Simplify* enabled the Gmail node returns `Subject`, `From`, and `To`, not `subject` and `from`. Reading `$json.subject` yields `undefined`, and `/exam/i.test(undefined)` returns `false` rather than throwing. All 50 messages fell through to the fallback branch with no indication of why. Dragging fields in from the INPUT panel instead of typing them avoids this entirely.

**A No-Op node holding the end of every branch.** All four Switch outputs were connected straight to the convergence node, with the Gmail label nodes never added. The workflow executed perfectly and made no API calls at all. Green checks confirm that a node ran, not that it did anything useful.

**A missing label.** A branch was built for a `Social` category before discovering Gmail refuses to create a user label with that name. n8n cannot create labels — they have to exist first. The branch was folded into Newsletters instead, which was the better design anyway.

## Possible extensions

- Swap keyword rules for an LLM classifier to catch messages that use none of the expected words
- Append every triaged message to a Google Sheet for a weekly volume report
- Auto-archive the Newsletters branch so it skips the inbox entirely
- Send a daily summary of anything landing in `To Review`
