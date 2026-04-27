<p align="center">
    <a href="https://kamy.dev">
      <img alt="Kamy" src="https://kamy.dev/icon.png" width="96">
    </a>
  </p>

  <h1 align="center">Kamy Plugin</h1>

  <p align="center">
    <a href="https://www.npmjs.com/package/@kamydev/sdk"><img alt="SDK" src="https://img.shields.io/npm/v/@kamydev/sdk?label=%40kamydev%2Fsdk&color=1863dc"></a>
    <a href="https://www.npmjs.com/package/@kamydev/cli"><img alt="CLI" src="https://img.shields.io/npm/v/@kamydev/cli?label=%40kamydev%2Fcli&color=1863dc"></a>
    <a href="https://kamy.dev"><img alt="kamy.dev" src="https://img.shields.io/badge/website-kamy.dev-1863dc.svg"></a>
    <img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue.svg">
  </p>

  > [Open Plugins](https://open-plugins.com) distribution for **Kamy** — install Kamy PDF generation
  > in Cursor, Claude Code, and any other Open-Plugins-compatible AI coding agent.

  This repository ships **only** the plugin manifest and the agent-facing assets
  (skill, rules, sub-agent, MCP config). The Kamy API, dashboard, SDK source, and
  template engine live in private infrastructure; this repo is the install
  surface for AI agents.

  ## What's in this plugin

  | Path | Component | Purpose |
  |---|---|---|
  | [`.plugin/plugin.json`](.plugin/plugin.json) | Manifest | Name, version, metadata, declared components |
  | [`skills/kamy/SKILL.md`](skills/kamy/SKILL.md) | Skill | Teaches the agent when and how to render PDFs |
  | [`rules/kamy-sdk.mdc`](rules/kamy-sdk.mdc) | Rule | Coding conventions for `@kamydev/sdk` and the REST API |
  | [`agents/pdf-template-author.md`](agents/pdf-template-author.md) | Sub-agent | Specialised template-design agent (Handlebars + print CSS) |
  | [`.mcp.json`](.mcp.json) | MCP server | Hosted MCP at `mcp.kamy.dev` exposing render tools |

  ## Install

  Point your Open-Plugins-compatible agent at this repository and follow the
  host's plugin-install flow:

  ```
  https://github.com/Kamy-Development/kamy-plugin
  ```

  The agent reads `.plugin/plugin.json` and loads the components automatically.

  You will also need a Kamy API key. Grab one from
  [kamy.dev/dashboard/keys](https://kamy.dev/dashboard/keys) and expose it as
  `KAMY_API_KEY` in your environment so the MCP server picks it up.

  ## Use Kamy without the plugin

  If you don't use an Open-Plugins-compatible agent, install the SDK or CLI
  directly:

  ```bash
  # SDK (Node, Bun, Deno, Workers, Edge, browser)
  npm i @kamydev/sdk

  # CLI (scripts, CI, previews)
  npm i -g @kamydev/cli
  ```

  Full docs at [kamy.dev/docs](https://kamy.dev/docs).

  ## Support

  - Docs — [kamy.dev/docs](https://kamy.dev/docs)
  - Status — [kamy.dev/status](https://kamy.dev/status)
  - Changelog — [kamy.dev/changelog](https://kamy.dev/changelog)
  - Email — [support@kamy.dev](mailto:support@kamy.dev)
  - Security — [security@kamy.dev](mailto:security@kamy.dev)

  ## License

  MIT — see [LICENSE](./LICENSE).
  