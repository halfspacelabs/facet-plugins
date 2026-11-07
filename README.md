# Facet plugins

[Facet](https://facet.halfspacelabs.com) takes a task from one plain-language ask to a live model: its agent builds a model for one narrow task, runs plain-language evals on every change, and serves it through an OpenAI-compatible API. This repository is its agent plugin: the remote MCP server and its Skills, for every agent that takes plugins.

In Claude Code the Skills are commands:

- `/facet:find`: find the LLM calls in your repo a Model could take over. Sends nothing to Facet.
- `/facet:new <file:line>`: make a Model from one call, with real inputs from your repo.
- `/facet:wire <model>`: compare a Model on your repo's examples, then swap the call over.
- `/facet:status`: the Models your repo calls, their evals and any Change in flight.

Nothing leaves your machine until you've seen it and said yes. Rows come only from your repo's tests, fixtures and samples; the agent never opens `.env` or credential files, drops rows that look like keys, offers to mask personal details, and never sends an LLM's answers as training data. The plugin runs no code on your machine.

- **Claude Code:** `/plugin marketplace add halfspacelabs/facet-plugins`, then `/plugin install facet@facet`
- **Cursor:** install Facet from the Cursor marketplace
- **Codex and ChatGPT:** install Facet from the plugin directory
- **Any MCP client:** add `https://facet.halfspacelabs.com/mcp`; it signs you in with OAuth

Facet is invite-only for now: [join the waitlist](https://halfspacelabs.com/waitlist) first, then sign in with the email we invite. If sign-in shows a waitlist form, that email isn't invited yet.

Sign in once in Claude Code:

1. Restart Claude Code so it loads the plugin.
2. Run `/mcp`, choose **facet**, then **Authenticate**.
3. Sign in in the browser tab that opens.

Docs: [facet.halfspacelabs.com/docs/agents](https://facet.halfspacelabs.com/docs/agents)

This repository is published from `plugin/` in Facet's own repository; change it there.
