# Plugin repository rules

This repo owns one Claude plugin; the Yellowtail application and MCP server live separately in
`perryjsteward/investor-signals`. Read README.md and INSTALL.md before changing it.

- Preserve the plugin ID and command prefix. Bump `.claude-plugin/plugin.json` and CHANGELOG.md
  together for every plugin change; update affected guides, SOPs and assets in the same commit.
- Run `pnpm check`. Every skill description must be 41–1024 characters, with no angle brackets.
- Keep `${CLAUDE_PLUGIN_ROOT}` references inside the shipped plugin. Never depend on an app checkout.
- MCP tool names must match the app registry. `tests/mcp-tools.json` records the verified tool names.
  For an MCP rename or help article change, coordinate with the application repo and refresh its
  contract snapshots with `pnpm plugins:contracts` from that repo. Keep shared house rules and display
  guidance consistent between the yt and yt-raise repositories.
- Cards and approval precede writes; never send email or publish assets without the user's instruction.
- Keep `.yt-raise/`, practice.md, CRM files and asset registries compatible for existing practices.
- Do not make this private repo public without an explicit instruction.
