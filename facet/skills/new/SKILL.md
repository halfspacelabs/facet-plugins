---
name: new
description: Make a Facet Model from one LLM call in this repo, using the real inputs and output shape the code already has. Use when the person wants Facet to take over a call, or runs /facet:new with a file and line.
argument-hint: "[file:line of the LLM call]"
---

# Make a Model from a call in this repo

The person's argument names the call site: $ARGUMENTS. If it is empty, ask which call, or offer /facet:find.

## 1. Read the call

- What goes in: the variables the prompt is built from.
- What should come out: the schema, type, enum or parser the code reads the answer into. Name the exact fields and allowed values.
- The rules the current prompt gives. The description you send can be that prompt; Facet writes the Model's own Instructions, so never write or tune a prompt for it.

## 2. Collect real inputs

- Take 10 to 20 real inputs from this repo only: tests, fixtures, eval sets, sample files and few-shot examples in the prompt. Never invent one; if there are fewer, use what there is and say so.
- Don't open production logs, databases or other services unless the person points at a file and says to use it. Don't open `.env` files, key stores or credential files.
- Add an output only when a person wrote it: an expected value in a test, a labeled fixture, a few-shot answer. Never send an answer an LLM produced (logged or cached responses, snapshots of model output) as an output; send that row as an input only.
- Drop any row that holds something that looks like a key, token or password, and count them.
- If rows hold personal details (names, emails, phone numbers, addresses, account numbers), say so and offer to mask them before anything is sent, with the same stand-in for the same value every time.

## 3. Show it before it leaves

Nothing goes to Facet until the person says yes. Show in one short message:

- the one or two sentence description,
- how many rows, which files they came from, how many were dropped and why,
- three of the rows as they will be sent.

Ask: "Send these to Facet to draft the Model?"

## 4. Draft it

- On yes, call `create_model` with the description and the rows as examples. If the result says drafting, call `create_model` again with only its `draft_id` a minute later.
- The result says `review`: Facet has drafted the Model and nothing is made yet.

## 5. Review the draft

Check it against the call site from step 1 before the person sees it:

- The input and output schemas: every field and allowed value the code reads, with the same names and types. Name any mismatch.
- Each eval: does it test something the current prompt or the code cares about? Flag ones that test nothing the code reads, and rules the code needs that no eval covers.
- The data: how many rows are theirs and how many the teacher wrote, and the teacher's sample rows.
- What was dropped and why: their rows that don't fit the schemas often mean the schema is wrong, not the row.

Show the person, in one short message: the name, what it takes and returns, each eval in one line, the rows by source, what was dropped and why, v1's quote, and what you'd change. Ask: "Create this Model?"

- On yes, call `create_model` with the `draft_id` and `confirm: true`, plus `drop_evals` with the ids of evals they reject, or `name` to rename it.
- If they want changes, or the draft misses the task, call `create_model` with the `draft_id` and the changes in plain words as `description`. Facet's agent redrafts the same draft; review the new one as above. Don't confirm a draft whose output schema doesn't match what the code reads.

## 6. Start v1

- The confirm result has a Planned Change to train v1 and its quote. Ask whether to start.
- Call `start_change` only after their yes in this conversation. Training takes a while; say so, and that Facet emails them when v1 merges. Offer to watch it in the background with `/loop` and `get_change`, as the use-facet Skill says under Watch a running Change.
- Once v1 merges, offer /facet:wire.
