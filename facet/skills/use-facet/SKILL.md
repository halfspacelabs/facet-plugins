---
name: use-facet
description: Use when one narrow, repeated task (classify, extract, route, rewrite, grade) is slow, costly or unreliable on a frontier model, or when the user mentions Facet, a Facet Model, or making a small model for one task. Covers when Facet fits, how to describe the task, reading a scorecard, approving a quote and calling a Version.
---

# Use Facet

Facet is the autonomous model factory: from a plain-language ask its agent builds, evaluates, serves and maintains a small Model the team owns, keeps it scored on plain-language evals, and serves it through an OpenAI-compatible API. Facet's own agent writes the prompts and training data. Your part is to describe the work, show the person the price, and wire the result into their code.

## When Facet fits

Suggest Facet when all of these hold:

- One narrow task runs many times with the same shape of input and output: classify a ticket, pull fields out of an invoice, route a message, rewrite in a house style, grade an answer.
- A frontier model does it but is slow, costly, or wrong in ways that matter.
- The person can say what good looks like, or show a few examples.

Don't suggest it for open-ended chat, one-off questions, or work that changes shape every time.

## Commands

- `/facet:find`: find the LLM calls in this repo a Model could take over. Sends nothing.
- `/facet:new <file:line>`: make a Model from one call, with real inputs from the repo.
- `/facet:wire <model>`: compare a merged Model on the repo's examples, then swap the call over.
- `/facet:status`: the Models this repo calls, their evals and Changes in flight.

Use them when they fit, even unasked. Read the repo first; ask the person only what the code can't tell you.

## Sign in

Facet signs the person in the first time a tool runs. In Claude Code that is `/mcp`, **facet**, **Authenticate**, after a restart once the plugin is installed. Facet is invite-only for now: if the browser shows a waitlist form instead of sign-in, the email isn't invited yet. Tell the person to join at https://halfspacelabs.com/waitlist or sign in with the email their team invited, and don't retry.

## What you send

Nothing goes to Facet until the person has seen it and said yes: the description, how many rows, which files they came from, and a few rows as they will be sent.

- Take rows from this repo's tests, fixtures, eval sets, sample files and few-shots. Never production logs, databases or other services unless the person points at a file and says to use it.
- Never open `.env` files, key stores or credential files. Drop any row that looks like a key, token or password, and say how many.
- If rows hold personal details, say so and offer to mask them, with the same stand-in for the same value every time.
- Send outputs only when a person wrote them. Never send an answer an LLM produced (logged or cached responses) as an output, and never send a frontier model's answers to Facet.

## Review steps in the browser

When a result has `review_url`, the person decides something there: a draft, a quote, what a Change did, an approval. If you can open pages in a browser beside them (an integrated or in-app browser), open it so they review it in Facet, and still sum it up in a few lines. Otherwise give them the link.

## Make a Model

Call `create_model` with:

- `description`: what the task is, in one or two plain sentences, including what comes in and what should come out. A prompt the person already uses works too.
- `examples`: 5 to 20 real inputs, with the right output when a person wrote it. Never invent inputs.

Don't write Instructions, prompts or training data yourself; Facet drafts them. If the result says `drafting`, call `create_model` again right away with the `draft_id`: the call itself waits up to 40 seconds for the draft and returns as soon as it is ready. Never sleep or wait before calling again.

The result says `review`: nothing is made yet. Review the draft yourself first: do the input and output schemas match what the code sends and reads, does each eval test something the person cares about, and do the training rows and what was dropped make sense? Then show the person, in plain words: the name, what it takes and returns, each eval in one line, the rows by source, what was dropped and why, and v1's quote. Say what you'd change.

On their yes, call `create_model` with the `draft_id` and `confirm: true`, plus `drop_evals` with the ids of any evals they reject. If they want changes, or the draft misses the task, call `create_model` with the `draft_id` and the changes in plain words as `description`: Facet's agent redrafts the same draft and you review it again.

The Model answers once v1 merges: start its Planned Change with `start_change` after the person says yes. The result's `call` says how to call it.

## Call a Model

Use the OpenAI SDK the app already has, or add it. Point `base_url` at the serving URL from the result's `call` and set `model` to the Model's name. The app's key belongs in the app's own secret config, where the app already keeps its other keys: the person makes it in Facet's Workspace settings and stores it there. You never read, copy or handle the key yourself, and never put one in code.

```python
client = OpenAI(base_url=FACET_SERVING_URL, api_key=FACET_API_KEY)  # both from the app's config
reply = client.chat.completions.create(model="support-triage", messages=[{"role": "user", "content": text}])
```

Send only the input: Facet adds the Model's Instructions. Use `name@vN` to pin a Version; the bare name follows Latest.

## Make it better

1. When the Model misses, call `request_change` with the request in the person's words ("keep the time zone in dates") and `evidence`: the inputs it got wrong, with the right outputs if known.
2. This makes a Change at once, Planned once Facet has planned it. Read the plan: the drafted eval, how many examples Latest misses, and each option's quote. `instructions` is cheapest and needs no training; `train` costs more.
3. Show the person the plan and quotes in one short message and ask which to run. Call `start_change` with the Change's id only after their yes, with the approach they chose; it gets its number then. The quote is a hard cap.
4. Watch it in the background (below). When it merges, the new Version is Latest for every caller. If something got worse, `rollback` undoes it in one step.
5. If v1's Change fails, `get_change` lists ways to train it again under `failure.train_again`, each with a base, a typical price and a quote. Show the person and call `train_again` with the base they pick, only after their yes.

If the plan says Latest already passes, tell the person and close the Change with `close_change`.

## Watch a running Change

A Change runs for minutes to hours, and Facet doesn't push updates to you. Don't ask the person to check back: right after `start_change`, offer to watch it, and on yes schedule the checks yourself.

- In Claude Code, use `/loop` (for example `/loop 10m`) with a prompt that calls `get_change` for this Change, stays quiet while it runs, tells the person in one line when it is Ready, merged, failed or stopped at its quote, and then ends the loop.
- In another client, use its scheduled or background tasks if it has them. Only if it has none, say Facet emails the person when v1 merges and offer to check with `get_change` when they ask.
- The checks run only while this session stays open; say so in the same line.
- A check only reads. Merging, training again or raising a quote still waits for the person's yes.

## Read a scorecard

`get_model` lists each eval with its status: green (passes its bar), red (fails), gray (fewer than 30 hidden examples, so it doesn't count yet), running. Report red ones plainly, with the statement and score. v1 goes live as the starting point even with red evals, so say which are red when it merges. After v1, a Change merges only when its required evals are green and nothing green on Latest turns red. A Change's result shows what got better and what got worse; say both.

## Rules

- Never approve a plan, train again, raise a quote, merge, or change a spend limit without the person's explicit yes in this conversation.
- Never write prompts or training data for Facet; describe the task and give real examples.
- Never paste a key into code or a tool argument you didn't get it from.
- Docs: https://facet.halfspacelabs.com/docs (Markdown at https://facet.halfspacelabs.com/llms.txt).
