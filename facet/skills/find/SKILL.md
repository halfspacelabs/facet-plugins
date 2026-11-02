---
name: find
description: Find the LLM calls in this repo that a small Facet Model could take over, ranked by what each would save. Use when the person asks which calls Facet could take, or runs /facet:find. Sends nothing to Facet.
---

# Find a call for Facet

Look through this repo for one LLM call a small Model could take over. This command only reads the repo on this machine; it sends nothing to Facet and calls no Facet tool.

1. Find calls to a hosted LLM (OpenAI, Anthropic, Gemini, Bedrock, OpenRouter and similar SDKs, or raw HTTP to them) whose job is narrow: they classify, extract, route, score or rewrite text, with the same shape of input and output every time. Skip open-ended chat and agent loops.
2. For each, note the file and line, what goes in, what comes out (the parser, schema or type the code reads the answer into), the model it calls and roughly how many tokens go in and out.
3. Rank them by what a small model would save: how often the call runs times what it costs. When the code doesn't show how often a call runs, ask the person instead of guessing.
4. Show the ranked list in a short table: call site, what it does, model, why it fits. Then name the top pick and how many real examples the repo holds for it (step 5).
5. Count real inputs for the top pick in this repo only: tests, fixtures, eval sets, sample files and few-shot examples in its prompt. Don't open production logs, databases or other services, and don't open `.env` files, key stores or credential files.
6. End with one line: "Make it with /facet:new <file:line>?"

Don't write prompts or examples for Facet, and don't invent inputs.
