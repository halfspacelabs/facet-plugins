# Facet plugins

[Facet](https://facet.halfspacelabs.com) builds a specialized model for one narrow task, checks every change against plain-language Expectations, and serves it through an OpenAI-compatible API. This repository is its agent plugin: the remote MCP server and the `use-facet` Skill, for every agent that takes plugins.

- **Claude Code:** `/plugin marketplace add halfspacelabs/facet-plugins`, then `/plugin install facet@facet`
- **Cursor:** install Facet from the Cursor marketplace
- **Codex and ChatGPT:** install Facet from the plugin directory
- **Any MCP client:** add `https://facet.halfspacelabs.com/mcp`; it signs you in with OAuth

Docs: [facet.halfspacelabs.com/docs/agents](https://facet.halfspacelabs.com/docs/agents)

This repository is published from `plugin/` in Facet's own repository; change it there.
