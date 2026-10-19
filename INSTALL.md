# Install and release Yellowtail Club

Repository: `perryjsteward/yt` (private). Marketplace: `yellowtail-club`. Plugin: `yt`.

## Claude Code

You need read access to this repository and working GitHub git credentials (SSH or an HTTPS
credential helper, for example `gh auth login` followed by `gh auth setup-git`). GitHub MCP is
not required: it provides tools to the model, not authentication for the plugin installer.

```sh
claude plugin marketplace add perryjsteward/yt
claude plugin install yt@yellowtail-club
```

From a local clone, `claude plugin marketplace add .` works too. Each repository contains its
own `.claude-plugin/marketplace.json`, listing its plugin from `./`. Installing either plugin
does not install the other. Core commands stay `/yt:…`; managed service commands stay `/yt-raise:…`.

For the other plugin, add its repository (`perryjsteward/yt-raise`) and install
`yt-raise@yellowtail-raise` separately.

## Claude organization GitHub sync

An organization Owner connects the private repo under **Organization settings → Plugins & skills →
Marketplaces → Add plugins → Sync from GitHub**. Give the Claude GitHub App access to this repository.
This GitHub App authorization is separate from installing a GitHub MCP server. Members can then
install the plugin according to the organization's access policy.

Source: [Claude's organization plugin setup](https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization).
A plain unauthenticated URL importer cannot clone a private repo; use authenticated GitHub sync,
Claude Code git credentials, or upload the plugin as a zip.

## Move from the app marketplace

Install `yt@yellowtail-club` first, then uninstall the old `yt@yellowtail` installation, so the
same commands are not loaded twice. For the service, remove an old `club@yellowtail` or
`raise@yellowtail` install if that is what you have. The old `yellowtail` catalog can continue to
point at the new GitHub repositories during migration. Existing practice files stay where they are.

## Release

1. Edit this repo, bump the plugin version and update CHANGELOG.md and affected documentation.
2. Run `pnpm check` and, with Claude Code installed, `claude plugin validate --strict .`.
3. For changed tool references or help links, run `pnpm plugins:contracts` in the app repo with
   these plugin repos cloned as siblings; review and commit the updated compatibility snapshots there.
4. Commit and push this repo. Its changes do not deploy the Yellowtail application.
5. Users run `claude plugin marketplace update yellowtail-club` and
   `claude plugin update yt@yellowtail-club`. Re-sync organization installations as needed.

The validation uses a checked snapshot of MCP names in `tests/mcp-tools.json`; the application
also checks the exported plugin contracts against its actual MCP registry and help articles.
When MCP tools change, update the tool snapshot here alongside the app change.
