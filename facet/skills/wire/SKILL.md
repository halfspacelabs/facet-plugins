---
name: wire
description: Swap an LLM call in this repo over to its Facet Model through the OpenAI SDK, after comparing both on the repo's own examples. Use when a Model's v1 has merged and the person wants to use it, or runs /facet:wire.
argument-hint: "[model name] [file:line]"
---

# Wire a Model into this repo

The person's argument names the Model and, if given, the call site: $ARGUMENTS. If the Model isn't named, call `list_models` and ask which. If its v1 hasn't merged yet (`get_model`), say so and stop.

## 1. Compare both models on this repo's examples

- Use inputs from this repo's tests, fixtures or sample files, up to 10, with the expected answers the repo already has, and the same rules as /facet:new: no production data unless the person points at a file, no secrets, personal details masked if the person asked for that.
- Call `try_version` with each input. Each call is metered; say how many you'll make before you start.
- Offer to run the current call on the same inputs, the way the app already calls it, with the key the app already uses, on this machine. It sends those inputs to the provider the app already uses and costs what a call costs there, so run it only on the person's yes. Its answers stay on this machine: never send them to Facet, as evidence or anywhere else.
- Show one table, one row per model:
  - correct out of N, against the repo's expected answers,
  - total cost of those N calls (`try_version` results carry cost; for the current model, its token counts at its provider's list price),
  - **cost per correct answer**: total cost divided by correct answers,
  - cost a month at the call's volume, if the code or the person says how often it runs.
- Add the Model's cost per 1,000 correct answers on its hidden examples from `get_version` (`per_1k_correct_micros`), when it has one.
- Say plainly where the answers differ. If the Model misses, offer `request_change` with those inputs as evidence instead of wiring it in.

## 2. Swap the call

On the person's yes:

- Use the OpenAI SDK the app already has, or add it. Set `base_url` to the serving URL from the `call` that `create_model` returned (if it isn't in this conversation, the person copies it from the Model's page in Facet) and keep it in the app's config, and set `model` to the Model's name. Send only the input: Facet adds the Model's Instructions.
- The key lives in the app's own secret config, beside its other keys: the person makes one in Facet's Workspace settings and stores it there. Never read, copy or write a key yourself, and never put one in code.
- Keep the old call behind one config flag, off by default, so going back is one line. Keep the code's parser and checks on the answer as they were.
- Run the repo's tests that cover the call and report the result.

Use `name@vN` to pin a Version; the bare name follows Latest. If a later Version makes things worse, `rollback` undoes it in one step.
