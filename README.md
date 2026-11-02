# Pane History

A [Herdr](https://herdr.dev) plugin for navigating back through previously
focused panes. One key opens a popup dialog listing every pane across every
workspace, **most recently focused first**, with a live preview of the pane you
would land in. Enter jumps to the exact pane.

Like a browser's back history, but for your panes.

## Requirements

- herdr 0.7.4 or newer (`herdr -V`)
- `fzf`, `jq`, and `python3` on `PATH`

## Install

```sh
herdr plugin install riogesulgon/herdr-history
```

Then bind keys in `~/.config/herdr/config.toml` (any free keys work):

```toml
[[keys.command]]
key = "super+u"
type = "shell"
command = "herdr plugin pane open --plugin pane-history --entrypoint picker"
description = "pane history dialog"

[[keys.command]]
key = "super+ctrl+j"
type = "plugin_action"
command = "pane-history.previous"
description = "previous pane"

[[keys.command]]
key = "super+ctrl+k"
type = "plugin_action"
command = "pane-history.next"
description = "next pane"
```

Reload with `herdr server reload-config`.

## Keys

Global:

| Key | Action |
|---|---|
| `super+u` | Open the pane-history dialog |
| `super+ctrl+j` | Previous pane — step back through the focus order |
| `super+ctrl+k` | Next pane — undo the step |

Dialog:

| Key | Action |
|---|---|
| type / backspace | Fuzzy-filter the list |
| `enter` | Focus the selected pane (its exact pane, not just its tab) |
| `esc` / `ctrl-c` | Close the dialog |

The pane you are on stays in the list at its place in the order, marked ➡️ —
after walking with previous/next that is your position in the history. On
top the ➡️ row is a non-selectable fzf header, so `enter` on the top row is
still "back to where I just was"; mid-list `enter` on the ➡️ row closes the
dialog without jumping. Previous/next walk the recorded order without
opening the dialog; their jumps are not recorded, so walking never rewrites
the order being walked and `super+ctrl+k` always returns exactly along the
path `super+ctrl+j` took.

## How the history is tracked

herdr exposes no last-focused timestamp, so the plugin records one: a
`pane.focused` hook touches one empty file per pane in the plugin's state
directory, and the picker reads them back in mtime order. The same pattern as
[sushidesu/herdr-tabpick](https://github.com/sushidesu/herdr-tabpick) uses for
tabs.

Two halves that only make sense together, which is why they ship as one
plugin: the popup pane runs the picker, and the event hook records the focus
order the picker sorts by.

Notes:

- The dialog is a `placement = "popup"` plugin pane. A popup has no pane id and
  emits no pane lifecycle events, so opening the dialog never pollutes the
  history it displays.
- Focus changes from any client (sidebar click, keybinding, CLI) are recorded
  — except this plugin's own previous/next jumps.
- Right after installing, the order is cold — panes you have not focused since
  installation sort by tab number.
- Records for closed or moved panes are pruned automatically whenever the
  picker opens.

## Development

```sh
git clone https://github.com/riogesulgon/herdr-history
cd herdr-history
herdr plugin link "$PWD"
herdr plugin list --plugin pane-history
herdr plugin log list --plugin pane-history   # event hook log
bin/pane-history list                          # needs the plugin env, or run
                                               # it from a plugin pane
```

`bin/herdr-rpc` is the one piece that does not go through the Herdr CLI: the
CLI has no pane-focus-by-id, so the exact jump uses the raw socket method
`pane.focus` over `$HERDR_SOCKET_PATH` (Unix socket — Linux/macOS only).

## Troubleshooting

**`fzf not found` / `jq not found`** — herdr starts the plugin with the `PATH`
the herdr server was started with, not a login shell's. Start herdr from a
shell where `fzf`, `jq`, and `python3` already resolve.

**The order never updates** — check that the hook is registered and enabled:

```sh
herdr plugin list --plugin pane-history
```

**Nothing jumps** — check `herdr plugin log list --plugin pane-history` for
hook errors; the picker reports socket errors on stderr in the popup.

## License

MIT
