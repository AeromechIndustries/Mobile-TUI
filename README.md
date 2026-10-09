<div align="center">
  <img src="logo/logo.svg" width="128" alt="remobi logo"/>
</div>

# Mobile-TUI

[![CI](https://github.com/AeromechIndustries/Mobile-TUI/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/AeromechIndustries/Mobile-TUI/actions/workflows/ci.yml)
[![upstream npm](https://img.shields.io/npm/v/remobi?label=upstream%20npm)](https://www.npmjs.com/package/remobi)
[![licence](https://img.shields.io/npm/l/remobi)](LICENSE)

Aeromech Industries' mobile terminal reference and adaptation fork for
[TessarAct](https://github.com/AeromechIndustries/TessarAct), derived from
[Connor's remobi](https://github.com/connorads/remobi). The upstream MIT licence
and copyright are retained in [LICENSE](LICENSE).

## How we use this repository

We study and reuse upstream's mobile terminal interaction code for TessarAct
Code: software-keyboard handling, responsive sizing, touch keys, scrolling and
gestures. The intended product is TessarAct's existing Code panel with better
mobile interaction, not another standalone terminal service or coding agent.

**Current status:** this is a reference/adaptation fork. It has not yet been
integrated into TessarAct or accepted on an actual iPhone. The initial upstream
baseline is [`2eb301e`](https://github.com/connorads/remobi/commit/2eb301e47e78409602cdfe60d1d59a654ee88306).
Upstream code is a useful starting point; its tests do not establish our own
browser, authentication or panel-lifetime acceptance.

| Area to reuse or learn from | Source | TessarAct adaptation |
|---|---|---|
| Keyboard-aware sizing and orientation | `src/viewport/`, `src/startup-resize.ts` | Measure the Code panel; keep the rest of the app scrollable. |
| Keyboard focus and touch handling | `src/util/keyboard.ts`, `src/util/tap.ts` | Preserve typing focus without repeatedly opening the phone keyboard. |
| Special-key toolbar and command drawer | `src/toolbar/`, `src/drawer/`, `src/actions/` | Configure controls for Mastra Code, not stock tmux bindings. |
| Scrolling, pinch and swipe | `src/gestures/`, `src/controls/` | Respect TUI mouse mode, selection and application navigation. |
| Terminal appearance and safe areas | `styles/base.css`, `src/theme/` | Scope styles to the terminal, retaining touch targets and iPhone safe areas. |
| Mobile regression coverage | `tests/playwright/`, `playwright.config.ts` | Add our mounted `/dev` journeys and real-device testing. |

### Integration boundary

- Native Mastra Code remains the coding controller and TUI inside the workspace
  sandbox. This repository supplies browser interaction, not an agent loop.
- TessarAct retains authentication, workspace access, attachment tickets,
  provider credentials and persistent-process transport. The upstream Hono/PTY
  server is available for standalone reference, not a replacement for that path.
- The current overlay uses `window.term`, appends controls to `document.body`,
  locks document height and can reload the page on reconnect. Embedding requires
  an explicit terminal/container adapter, panel-scoped sizing and cleanup on
  unmount. Its existing `init()` is not a drop-in React component.
- Keep reusable upstream fixes separate from TessarAct-specific adapters. Pin
  the reviewed source revision and retain attribution when copying or bundling
  code. There is no predetermined percentage of code to reuse.

## Fork maintenance

`origin` is `AeromechIndustries/Mobile-TUI`; its default integration branch is
`master`. `upstream` is `connorads/remobi`, whose default branch remains `main`.
Open focused PRs against this fork's `master`; inspect upstream changes before
merging or cherry-picking them. Do not overwrite our changes with a blind sync.

Track fork work in [this repository's issues](https://github.com/AeromechIndustries/Mobile-TUI/issues).
TessarAct integration and acceptance stay in the application repository,
currently [issue #534](https://github.com/AeromechIndustries/TessarAct/issues/534).
An upstream npm install installs upstream remobi, not this fork.

Automatic publishing is disabled, and `package.json` is private to prevent
publishing to upstream's `remobi` package. The existing package/import names are
retained for source compatibility. A separately named distributable and release
process require a deliberate later decision. CI validation remains enabled.

## Upstream standalone application reference

The remaining sections describe remobi's standalone operation. Its installers,
npm packages, website and tmux defaults belong to upstream; they do not install
or configure TessarAct Code. Use the fork development commands below when
working on our source.

**Your terminal. Everywhere.**

Your tmux session, on your phone. Same panes, same windows, same bindings — nothing changes on your computer. Swipe between windows, pinch to zoom, tap to send commands. You just get a remote control.

It's a terminal on a 6-inch screen. It won't win design awards. But you can do *everything* — monitor coding agents, intervene when they're stuck, scroll through output, switch contexts. Full power.

```bash
/bin/bash -c "$(curl -fsSL http://remobi.app/install.sh)"
```

To upgrade stable: `npm install -g remobi@latest`

To try the experimental channel: `npm install -g remobi@dev`

Your coding agent handles the rest. It installs remobi, inspects your tmux config, generates a config, and suggests tweaks to make your tmux more mobile-friendly — one conversation. Works with Claude Code and Codex.

## Why remobi

- **Zero workflow changes** — your existing tmux setup, untouched
- **Swipe between windows** — gesture navigation, no prefix key fumbling on a phone screen
- **Pinch to zoom** — resize text like every other app on your phone
- **Install to your home screen** — standalone PWA, looks and feels native
- **Config-driven** — your buttons, your gestures, your layout
- **Self-hosted** — local-first by default. Bring your own access layer (Tailscale, Cloudflare, ngrok)

<div align="center">
  <video src="https://github.com/user-attachments/assets/952bdb34-4b73-4210-815a-b2b60f99f87f" />
</div>

## Requirements

- [Node.js](https://nodejs.org/) ≥ 22
- [tmux](https://github.com/tmux/tmux) (the default target multiplexer) — or [zellij](https://github.com/zellij-org/zellij) ([Using with zellij](#using-with-zellij)) or [herdr](https://github.com/ogulcancelik/herdr) ([Using with herdr](#using-with-herdr))

## Manual setup

```bash
# 1. Install
npm install -g remobi

# 2. Enable mouse mode in tmux (required for touch scroll and tap-to-focus)
#    Add to ~/.config/tmux/tmux.conf or ~/.tmux.conf:
#    set -g mouse on

# 3. Start (spawns your command, serves remobi on 127.0.0.1:7681)
remobi serve
```

**Essential tmux settings for mobile** — add to your `tmux.conf` if not already present:

| Setting | Command | Why |
|---------|---------|-----|
| Mouse mode | `set -g mouse on` | Enables touch scroll, tap-to-focus, drag resize. **The single highest-value setting for mobile use.** |
| Status position | `set -g status-position top` | Keeps status bar away from remobi's touch toolbar at the bottom |
| Renumber windows | `set -g renumber-windows on` | Keeps window list tidy after closing windows |

See [Mobile-friendly tmux config](.agents/skills/remobi-setup/references/mobile-tmux.md) for responsive status bars, popup sizing, and more.

For local development, see the [Development](#development) section below.

Open `http://localhost:7681` on the same machine to verify it works. For phone access, put a trusted proxy/tunnel in front of it, for example [Tailscale Serve](.agents/skills/remobi-setup/references/tailscale-serve.md). If your proxy mounts remobi under a URL prefix, start remobi with `--base-path /that-prefix` so the HTML, PWA links, and WebSocket all use the same external path.

## Using with zellij

[zellij](https://github.com/zellij-org/zellij) is a batteries-included tmux alternative with a discoverable, modal UI. remobi serves it the same way it serves tmux:

```bash
remobi serve -- zellij attach --create main
```

`zellij attach --create <name>` attaches or creates, like `tmux new-session -A`. Sessions persist across remobi restarts (zellij detaches rather than dying when the client goes away).

zellij needs no extra setup for mobile:

- Mouse mode is on by default, so touch scroll and tap-to-focus just work — no `set -g mouse on` equivalent to remember
- Stock zellij ships a tmux-compat mode on `Ctrl-B`, so remobi's Prefix button and the default swipe gestures (`prefix+n`/`prefix+p` — next/previous tab) work unchanged, as do the New Window (`prefix+c`), Split (`prefix+%`/`prefix+"`), Zoom (`prefix+z`), Copy/scrollback (`prefix+[`), and Kill (`prefix+x`) drawer buttons
- Its native modal shortcuts (`Ctrl+t` for tabs, `Ctrl+p` for panes, …) also pass through fine

Only three drawer buttons send sequences zellij doesn't bind. A `remobi.config.ts` with zellij equivalents:

```typescript
export default {
  name: 'zellij',
  drawer: {
    buttons: (defaults) => [
      ...defaults.filter((b) => !['tmux-sessions', 'tmux-windows', 'tmux-help'].includes(b.id)),
      { id: 'zellij-sessions', label: 'Sessions', description: 'Open session manager', action: { type: 'send', data: '\x0fw' } },
      { id: 'zellij-lock', label: 'Lock', description: 'Toggle locked mode (pass Ctrl keys through)', action: { type: 'send', data: '\x07' } },
    ],
  },
}
```

`zellij-lock` toggles zellij's locked mode, which passes `Ctrl` keys straight through — handy when a TUI inside the session wants shortcuts zellij would otherwise capture. If you also stay attached from your desktop while remobi is connected, set `mirror_session true` in `~/.config/zellij/config.kdl` so both clients see the same view. For interactive onboarding, point an AI agent at the [remobi-setup skill](.agents/skills/remobi-setup/SKILL.md), which covers the zellij path.

## Using with herdr

[herdr](https://github.com/ogulcancelik/herdr) is an agent multiplexer — a tmux alternative purpose-built for supervising AI coding agents, with per-pane agent status detection. remobi serves it the same way it serves tmux:

```bash
remobi serve -- herdr --session main
```

`herdr --session <name>` attaches or creates, like `tmux new-session -A`. Sessions persist across remobi restarts.

herdr needs no extra setup for mobile:

- Mouse capture is on by default, so touch scroll and tap-to-focus just work — no `set -g mouse on` equivalent to remember
- Its default prefix is `Ctrl-B`, the same as tmux, so remobi's Prefix button and the default swipe gestures (`prefix+n`/`prefix+p` — next/previous tab) work unchanged, as do the New Window (`prefix+c`), Zoom (`prefix+z`), Kill (`prefix+x`), and Help (`prefix+?`) drawer buttons
- It has a built-in single-column mobile layout for narrow terminals (`ui.mobile_width_threshold` in herdr's config)

Only a few drawer buttons send sequences herdr doesn't bind. A `remobi.config.ts` with herdr equivalents:

```typescript
export default {
  name: 'herdr',
  drawer: {
    buttons: (defaults) => [
      ...defaults.filter(
        (b) => !['tmux-split-vertical', 'tmux-split-horizontal', 'tmux-sessions', 'tmux-windows', 'tmux-copy'].includes(b.id),
      ),
      { id: 'herdr-split-v', label: 'Split |', description: 'Split pane side-by-side', action: { type: 'send', data: '\x02v' } },
      { id: 'herdr-split-h', label: 'Split —', description: 'Split pane stacked', action: { type: 'send', data: '\x02-' } },
      { id: 'herdr-workspaces', label: 'Spaces', description: 'Workspace picker', action: { type: 'send', data: '\x02w' } },
      { id: 'herdr-sidebar', label: 'Sidebar', description: 'Toggle agent sidebar', action: { type: 'send', data: '\x02b' } },
      { id: 'herdr-scrollback', label: 'Scroll', description: 'Edit scrollback', action: { type: 'send', data: '\x02e' } },
    ],
  },
}
```

`tmux-windows` is filtered out because in herdr `prefix+w` opens the workspace picker — `herdr-workspaces` re-adds the same sequence with an accurate label. For interactive onboarding, point an AI agent at the [remobi-setup skill](.agents/skills/remobi-setup/SKILL.md), which covers the herdr path.

## Publishing policy

This fork does not publish npm packages or automated GitHub releases. Its
inherited semantic-release configuration names `master` and `dev`, but is
inactive: CI has no release job and the package is private. No `dev` branch is
required for reference/adaptation work.

Upstream's release channels and installation instructions are maintained in
[its README](https://github.com/connorads/remobi#readme). Continue using
conventional commits so reusable changes are easy to review upstream.

## Set up with AI

The setup skill checks your environment, inspects your tmux config, generates a `remobi.config.ts`, and suggests tmux mobile optimisations — one conversation. Three ways to use it, from simplest to most manual:

**Option 1: One-liner** — installs the skill and launches an interactive session with your coding agent:

```bash
/bin/bash -c "$(curl -fsSL http://remobi.app/install.sh)"
```

**Option 2: Install the skill** — if you prefer to start the conversation yourself:

```bash
npx skills add connorads/remobi
```

Then tell your coding agent: "Use the remobi-setup skill to onboard me."

**Option 3: Already in a session?** — if an agent is already looking at this repo (or you don't want to install a skill), tell it:

> Read `.agents/skills/remobi-setup/SKILL.md` in this repo and follow it to onboard me.

The skill file contains the full setup workflow. The agent can read it directly — no installation needed.

## Security model

`remobi` is a remote-control surface for your terminal. Anyone who can reach it can drive the tmux session with your user privileges.

- `remobi serve` binds to `127.0.0.1` by default.
- The inner PTY-backed terminal session stays local to the remobi process.
- There is no built-in login, password, or ACL in remobi itself.
- Safe default: keep it on localhost and publish it through a trusted layer like Tailscale Serve.
- If you use `remobi serve --host 0.0.0.0`, you are exposing terminal control to your LAN/whatever can route to that port. Do that only if you intentionally want direct network exposure and have separate network controls in place.

To report a vulnerability, see [SECURITY.md](SECURITY.md).

## CLI reference

```text
remobi serve [--config <path>] [--port <n>] [--host <addr>] [--base-path <path>] [-- <command...>]
  Start remobi with its built-in web terminal and PWA support.
  Default host: 127.0.0.1. Default port: 7681. Default command: tmux new-session -A -s main
  Example: remobi serve --host 0.0.0.0 --port 8080
  Example: remobi serve --base-path /random-token
  Example: remobi serve --port 8080 -- tmux new -As dev

remobi build [--config <path>] [--output <path>] [--dry-run]
  Deprecated. remobi no longer patches ttyd HTML.

remobi inject [--config <path>] [--dry-run]
  Deprecated. remobi no longer patches ttyd HTML.

remobi init
  Scaffold a remobi.config.ts with commented defaults.

remobi --version
remobi --help
```

Short flags: `-c` (`--config`), `-p` (`--port`). Legacy deprecated flags: `-o` (`--output`), `-n` (`--dry-run`).

### Config resolution

When `--config` is not specified, remobi searches:

1. `remobi.config.ts` / `.js` in the current directory
2. `~/.config/remobi/remobi.config.ts` / `.js` (XDG fallback)

## Configuration

Create `remobi.config.ts` (or run `remobi init`):

```typescript
export default {
  font: {
    family: 'JetBrainsMono NFM, monospace',
    mobileSizeDefault: 16,
    sizeRange: [8, 32],
  },
  toolbar: {
    row1: [
      { id: 'esc', label: 'Esc', description: 'Send Escape key', action: { type: 'send', data: '\x1b' } },
      { id: 'tmux-prefix', label: 'Prefix', description: 'Send tmux prefix key (Ctrl-B)', action: { type: 'send', data: '\x02' } },
      // ...
    ],
    row2: [
      { id: 'alt-enter', label: 'M-↵', description: 'Send Alt+Enter (ESC + Enter)', action: { type: 'send', data: '\x1b\r' } },
      { id: 'drawer-toggle', label: '☰ More', description: 'Open command drawer', action: { type: 'drawer-toggle' } },
      { id: 'paste', label: 'Paste', description: 'Paste from clipboard', action: { type: 'paste' } },
      { id: 'backspace', label: '⌫', description: 'Send Backspace key', action: { type: 'send', data: '\x7f' } },
      // ...
    ],
  },
  drawer: {
    buttons: [
      { id: 'tmux-new-window', label: '+ Win', description: 'Create tmux window', action: { type: 'send', data: '\x02c' } },
      { id: 'tmux-split-vertical', label: 'Split |', description: 'Split pane vertically', action: { type: 'send', data: '\x02%' } },
      { id: 'combo-picker', label: 'Combo', description: 'Open combo sender (Ctrl/Alt + key)', action: { type: 'combo-picker' } },
      // ...
    ],
  },
  gestures: {
    swipe: {
      enabled: true,
      left: '\x02n',         // data sent on swipe left (default: next tmux window)
      right: '\x02p',        // data sent on swipe right (default: prev tmux window)
      leftLabel: 'Next tmux window',    // shown in help overlay
      rightLabel: 'Previous tmux window',
    },
    scroll: {
      enabled: true,
      strategy: 'wheel',
      sensitivity: 40,
      wheelIntervalMs: 24,
    },
    pinch: { enabled: true },
  },
  mobile: {
    initData: '\x02z',     // send on mobile load when viewport < widthThreshold
    widthThreshold: 768,   // px — default matches common phone/tablet breakpoint
  },
  floatingButtons: [
    {
      position: 'top-left',
      buttons: [
        { id: 'zoom', label: 'Zoom', description: 'Toggle pane zoom', action: { type: 'send', data: '\x02z' } },
      ],
    },
  ],
}
```

All fields are optional — the CLI fills in defaults internally when it loads the config.

Shipped tmux drawer defaults stick to stock tmux bindings (`c`, `%`, `"`, `s`, `w`, `[`, `?`, `x`, `z`) rather than personal popup workflows.

Replace the drawer entirely with a plain array when you want a fully custom setup:

```typescript
import { defineConfig } from 'remobi/config'

export default defineConfig({
  drawer: {
    buttons: [
      { id: 'sessions', label: 'Sessions', description: 'Choose tmux session', action: { type: 'send', data: '\x02s' } },
      { id: 'git', label: 'Git', description: 'Open my tmux git popup', action: { type: 'send', data: '\x02g' } },
    ],
  },
})
```

At runtime, remobi validates the config object shape and rejects unknown keys with clear path-based errors.

`gestures.scroll.strategy` controls touch scroll behaviour:

- `wheel` (default): sends SGR mouse wheel events with touch-mapped terminal coordinates.
- `keys`: sends `PageUp` / `PageDown` for app-level paging when preferred.

### Programmatic API

```typescript
import { defineConfig, serialiseThemeForTtyd } from 'remobi/config'
import type { RemobiConfig, ControlButton } from 'remobi/types'
import { init } from 'remobi'
```

Advanced consumers can use hook registry primitives to observe lifecycle and terminal-send events:

```typescript
import { createHookRegistry, init } from 'remobi'

const hooks = createHookRegistry()
hooks.on('beforeSendData', (ctx) => {
  if (ctx.data.includes('rm -rf /')) return { block: true }
})

init(undefined, hooks)
```

## Guides

- [Mobile-friendly tmux config](.agents/skills/remobi-setup/references/mobile-tmux.md) — responsive status bar, popup sizing, binding ergonomics
- [Mobile pane navigation](.agents/skills/remobi-setup/references/mobile-panes.md) — zoom-aware swipe, auto-zoom on load, floating buttons
- [Tailscale Serve](.agents/skills/remobi-setup/references/tailscale-serve.md) — expose over your tailnet with HTTPS
- [Keeping your Mac awake](.agents/skills/remobi-setup/references/keep-awake.md) — prevent sleep during remote sessions
- [ttyd flags](.agents/skills/remobi-setup/references/ttyd-flags.md) — legacy notes for old ttyd-based setups

## Architecture docs

- [How remobi works](docs/architecture/how-remobi-works.md) — runtime overview, shared session model, and boot path
- [Networking and WebSocket flow](docs/architecture/networking-and-websockets.md) — request lifecycle, protocol, and network boundary

## Architecture

Pure TypeScript + DOM API — no framework. The build bundles the browser client via esbuild, serves it from Node, and bridges browser input/output to a local PTY via `node-pty`. `xterm.js` handles terminal rendering in the browser; remobi layers the mobile controls on top. The docs above walk through the current runtime in more detail, including diagrams for the server, browser, and WebSocket flow.

Key modules:

| Module | Purpose |
|--------|---------|
| `src/toolbar/` | Two-row touch toolbar |
| `src/drawer/` | Command drawer with grid layout |
| `src/gestures/` | Swipe, pinch, scroll detection |
| `src/controls/` | Font size, help overlay, scroll buttons |
| `src/theme/` | Catppuccin Mocha + theme application |
| `src/viewport/` | Height management, landscape detection |
| `src/util/` | DOM helpers, terminal, keyboard, haptics |

## Public API and semver

remobi follows semantic versioning. The public API is defined by the following import paths:

| Import path | Contents | Stability |
|---|---|---|
| `remobi` | `init`, `defineConfig`, `createHookRegistry`, `RemobiConfig`, `ControlButton`, `ButtonAction`, `ButtonArrayPatch`, `ButtonArrayInput`, `RemobiConfigOverrides`, `HookRegistry` | Public — breaking changes are semver-major |
| `remobi/config` | `defineConfig`, `mergeConfig`, `defaultConfig`, `serialiseThemeForTtyd` | Public |
| `remobi/types` | All types in `src/types.ts` | Public |

**Internal modules** (not part of the public API — may change without a major version bump):
`src/toolbar/`, `src/drawer/`, `src/gestures/`, `src/controls/`, `src/theme/`, `src/viewport/`, `src/util/`, `src/serve.ts`, `src/cli/`, `build.ts`

**Semver policy**:
- **Major**: removing or renaming a public export, changing a public function signature incompatibly, removing a config field
- **Minor**: adding new public exports, new optional config fields, new `ButtonArrayInput` operations
- **Patch**: bug fixes, internal refactors, documentation updates

## Development

```bash
git clone https://github.com/AeromechIndustries/Mobile-TUI.git
cd Mobile-TUI
git remote add upstream https://github.com/connorads/remobi.git
mise install                         # use this repo's Node/pnpm/tool versions
pnpm install --frozen-lockfile
git config core.hooksPath .hk-hooks   # enable commit hooks (conventional commits, biome)
```

### Running locally

From source (bundles the browser client on the fly via esbuild — no build step needed):

```bash
pnpm exec tsx cli.ts serve    # localhost:7681, default tmux session
```

Or build first, then run from dist/:

```bash
pnpm run build:dist          # transpile TS → JS + bundle browser client
node dist/cli.mjs serve      # run locally-built version on localhost:7681
```

No watch mode — re-run the build or use `tsx` for automatic source bundling.

### Checks

```bash
pnpm test            # vitest (unit + integration)
pnpm run test:pw     # playwright e2e (needs: pnpm exec playwright install chromium webkit --with-deps)
pnpm run check       # biome lint + format
```

## FAQ

**Is this secure?**
remobi doesn't handle auth — it's a UI overlay. Use a tunnel or VPN you trust. We recommend [Tailscale](.agents/skills/remobi-setup/references/tailscale-serve.md) (deployment guide included) — your session never leaves your tailnet. Cloudflare Tunnel and ngrok also work. Security is your responsibility.

**Why not Termux / Termius / SSH apps?**
They work. But you're managing SSH keys, losing your tmux setup, and fighting a UI that wasn't built for touch. remobi keeps your exact workflow — same panes, same windows, same bindings — and adds touch controls on top.

**Why not [Happy](https://github.com/slopus/happy) / Claude resume / chat-based mobile apps?**
Those tools change your workflow. Chat relays route through third-party servers. Claude's resume has limitations. remobi gives you the raw terminal — full power, self-hosted, works with every agent because it works with tmux.

**Why Node?**
remobi migrated from Bun to Node.js + pnpm for broader compatibility. It transpiles to JS via tsdown for npm distribution and uses esbuild for the browser client bundle.

**Is this production-ready?**
Upstream includes unit and mobile-browser tests. This fork is not yet a shipped
TessarAct integration; real iPhone keyboard, rotation, selection and reconnect
acceptance remain required before claiming that experience is ready.

## Acknowledgements

remobi owns the full web terminal path now: a local PTY on the server, `xterm.js` in the browser, and the mobile touch controls on top.

Earlier versions were built on top of [ttyd](https://github.com/tsl0922/ttyd). It helped remobi launch quickly, prove the workflow, and shape the product before the runtime moved in-house.

## Licence

MIT
