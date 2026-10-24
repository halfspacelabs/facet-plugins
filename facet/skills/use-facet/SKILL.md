---
name: use-facet
description: Use when one narrow, repeated job (classify, extract, route, rewrite, grade) is slow, costly or unreliable on a frontier model, or when the user mentions Facet, a Task, or making a specialized model. Covers when Facet fits, how to describe a Task, reading a scorecard, approving a quote and calling a Version.
---

# Use Facet

Facet builds and runs a specialized model for one narrow job (a Task), keeps it checked against plain-language Expectations, and serves it through an OpenAI-compatible API. Facet's own agent writes the prompts and training data. Your job is to describe the work, show the person the price, and wire the result into their code.

## When Facet fits

Suggest Facet when all of these hold:

- One narrow job runs many times with the same shape of input and output: classify a ticket, pull fields out of an invoice, route a message, rewrite in a house style, grade an answer.
- A frontier model does it but is slow, costly, or wrong in ways that matter.
- The person can say what good looks like, or show a few examples.

Don't suggest it for open-ended chat, one-off questions, or work that changes shape every time.

## Make a Task

Call `create_task` with:

- `description`: what the job is, in one or two plain sentences, including what comes in and what should come out. A prompt the person already uses works too.
- `examples`: 5 to 20 real inputs, with the right output when known. Real inputs beat invented ones.

Don't write Instructions, prompts or training data yourself; Facet drafts them. If the result says `drafting`, call `create_task` again with the `draft_id` a minute later.

Version 0 is callable at once. The result's `call` says how.

## Call a Task

Use the OpenAI SDK the app already has, or add it. Point `base_url` at the serving URL from the result (or from `get_me`) and set `model` to the Task name. The app's key belongs in the app's own secret config, where the app already keeps its other keys: the person makes it in Facet's Workspace settings and stores it there. You never read, copy or handle the key yourself, and never put one in code.

```python
client = OpenAI(base_url=FACET_SERVING_URL, api_key=FACET_API_KEY)  # both from the app's config
reply = client.chat.completions.create(model="support-triage", messages=[{"role": "user", "content": text}])
```

Send only the input: Facet adds the Task's Instructions. Use `name@vN` to pin a Version; the bare name follows Latest.

## Make it better

1. When the Task misses, call `file_ask` with the request in the person's words ("keep the time zone in dates") and `evidence`: the inputs it got wrong, with the right outputs if known.
2. Read the plan: the drafted Expectation, how many examples Latest misses, and each option's quote. `instructions` is cheapest and needs no training; `train` costs more.
3. Show the person the plan and quotes in one short message and ask which to run. Call `approve_plan` only after their yes, with the approach they chose. The quote is a hard cap.
4. Follow it with `get_change`. When it merges, the new Version is Latest for every caller. If something got worse, `rollback` undoes it in one step.

If the plan says Latest already passes, tell the person and close the Ask with `close_ask`.

## Read a scorecard

`get_task` lists each Expectation with its status: green (passes its bar), red (fails), gray (not measured yet), running. Report red ones plainly, with the statement and score. A Change's result shows what got better and what got worse; say both.

## Rules

- Never approve a plan, raise a quote, merge, or change a spend limit without the person's explicit yes in this conversation.
- Never write prompts or training data for Facet; describe the job and give real examples.
- Never paste a key into code or a tool argument you didn't get it from.
- Docs: https://facet.halfspacelabs.com/docs (Markdown at https://facet.halfspacelabs.com/llms.txt).
