# This fork

This is John Randall's fork of [riogesulgon/herdr-history](https://github.com/riogesulgon/herdr-history), a herdr plugin (id `pane-history`) that adds browser-style back and forward through the panes you have focused, plus a most-recent-first picker. It is the watchbill system `herdr-history` (`watchbill systems show herdr-history`): the plugin does the work, and its lead designer maintains the fork, its upstream pull requests, and the keys that call it.

## Start here

- **herdr** is the terminal multiplexer John runs his Claude sessions in. A **pane** is one terminal, a **tab** holds panes, and a **workspace** (shown in herdr as a space) holds tabs.
- herdr's own `last_pane` key (`⌃b ;`) remembers one previous pane only, so pressing it again swaps you back. This plugin keeps the whole order.
- **Upstream** is the original repository; **the fork** is `johntrandall/herdr-history` on GitHub, mirrored to Forgejo on Umbridge.

## Branches

The layout follows the `vendor-fork-deploy` convention.

| Branch | What it is | Upstream PR |
|---|---|---|
| `main` | Tracks `upstream/main`, untouched | — |
| `fix/one-call-listing` | Snapshot every pane in three herdr calls instead of two per workspace (2.8 s → 0.27 s a press with 103 workspaces) | opened 2026-11-12 |
| `local/watchbill-system` | This file and the watchbill system files (`intake/`, `dev-docs/`, `_types/`, `mdbase.yaml`); never sent upstream | — |
| `deployed` | `main` merged with every branch above; the checkout herdr runs | — |

herdr runs the plugin from this folder (`herdr plugin link ~/dev/herdr-history`), so the plugin is whatever `deployed` holds here. Keep `deployed` checked out. Work on a topic branch made from `main`, merge it into `deployed` with `--no-ff`, and never rebase `deployed`.

## Keys

The keys live in chezmoi's `dot_config/herdr/config.toml`, never in this repository: `⌃b [` back, `⌃b ]` forward, `⌃b u` the picker. Change them there, run `chezmoi apply ~/.config/herdr/config.toml` and `herdr server reload-config`, and update the herdr section of `~/admin-technical/inventories/Keyboard-Shortcuts-Inventory.md`.

## When upstream moves

Fetch `upstream`, fast-forward `main`, and merge `main` into `deployed`. A pull request of ours that upstream merged then needs nothing more; delete its branch. A request goes in `intake/`, one note each, from `intake/TEMPLATE.md`.
