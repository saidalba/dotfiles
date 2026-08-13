### dotfiles

My `chezmoi` source directory — single source of truth for dotfiles across machines.

```
chezmoi apply
```

#### Managed

- `dot_config/bash/rc` — bashrc: aliases, prompt, `$EDITOR=micro`
- `dot_config/foot/foot.ini` — foot terminal config
- `dot_config/readline/inputrc` — readline keybinds
- `dot_config/sway/config` — sway WM config (hardcoded monitor outputs, per-machine)
- `dot_claude/settings.json` — Claude Code settings
- `dot_config/etc/keyd/*.conf`, `dot_config/etc/systemd/logind.conf.d/*.conf` — copies only,
  not live (see caveat below)

Shared between a macOS laptop and this Arch Linux laptop. `.chezmoiignore.tmpl` skips
Linux-only configs (sway, keyd, logind) on macOS.

#### Caveat: `dot_config/etc/...` isn't live

`keyd` and `systemd-logind` read from `/etc/keyd/` and `/etc/systemd/logind.conf.d/`, which
are outside `$HOME` and outside what chezmoi can target here. These files are kept as
reference copies only — chezmoi apply does **not** sync them. Edit `/etc/...` directly, then
manually copy changes back into this repo.
