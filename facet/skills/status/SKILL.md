---
name: status
description: Show the Facet Models this repo calls, how each scores on its evals, and any Change in flight. Use when the person asks how their Facet Models are doing, or runs /facet:status.
---

# Facet status for this repo

1. Call `list_models`, then search this repo on this machine for each Model's name (as `model="name"` or `name@vN`, in code or config). Send nothing about the repo to Facet.
2. For each Model this repo calls, call `get_model` and `list_changes`, then show one short table: Model, the Version the code calls (Latest or pinned `name@vN`), each eval with its status (green passes its bar, red fails, gray not measured yet, running), and any Change that is planned, queued or running with its quote and spend.
3. If a Change is queued or running, offer to watch it in the background with `/loop` and `get_change`, as the use-facet Skill says under Watch a running Change.
4. Report red evals plainly, with the statement and score. List other Models in the workspace in one line, without detail.
5. If an eval is red, or the person says the Model gets something wrong, offer `request_change` in the person's words, with the failing inputs as evidence. Take those inputs from this repo or from what the person gives you, with the same rules as /facet:new: no secrets, no production data unless the person points at it, and offer to mask personal details. Show the plan and quote; call `start_change` only after their yes.
