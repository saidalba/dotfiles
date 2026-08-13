# chezmoi dotfiles repo

This is my `chezmoi` source directory (`~/.local/share/chezmoi`), which doubles as this
git repo. It is the single source of truth for my dotfiles across two machines — a macOS
laptop and this Arch Linux laptop. `chezmoi apply` reads from here and writes into `$HOME`.
`CLAUDE.md` and `README.md` are excluded from being applied (see `.chezmoiignore.tmpl`).

## Layout

chezmoi maps source paths to target paths by stripping the `dot_` prefix and translating it
to a literal `.`. Everything here targets `$HOME`, since this repo uses no `.chezmoiroot` and
no root/system-level target support.

| Source path | Applies to | Purpose |
|---|---|---|
| `dot_claude/settings.json` | `~/.claude/settings.json` | Claude Code settings |
| `dot_config/bash/rc` | `~/.config/bash/rc` | bashrc: aliases, prompt, `$EDITOR=micro` |
| `dot_config/foot/foot.ini` | `~/.config/foot/foot.ini` | foot terminal config |
| `dot_config/readline/inputrc` | `~/.config/readline/inputrc` | readline keybind (`C-h` = backward-kill-word) |
| `dot_config/sway/config` | `~/.config/sway/config` | sway WM config — borders, gaps, keybinds, **hardcoded monitor outputs** |
| `dot_config/etc/keyd/*.conf` | `~/.config/etc/keyd/*.conf` | **not** `/etc/keyd/` — see caveat below |
| `dot_config/etc/systemd/logind.conf.d/*.conf` | `~/.config/etc/systemd/logind.conf.d/*.conf` | **not** `/etc/systemd/` — see caveat below |

## Known caveat: the `dot_config/etc/...` files are not live

`keyd` and `systemd-logind` read their config from `/etc/keyd/` and
`/etc/systemd/logind.conf.d/` respectively — real system paths, not under `$HOME`. Because
chezmoi (without root/external-path support configured) can only ever target paths under
`$HOME`, `chezmoi apply` writes these into `~/.config/etc/...` instead, which nothing on the
system reads.

In practice this means:
- The copies in this repo and the live files in `/etc/keyd/` and `/etc/systemd/logind.conf.d/`
  are two independent trees that only match by manual, one-off copying (`sudo cp` /
  `diff`) — chezmoi will never sync them for you, on this machine or a new one.
- After `chezmoi apply` on a fresh machine, keyd/logind will **not** pick up these configs
  until someone manually copies `~/.config/etc/keyd/*.conf` → `/etc/keyd/` and
  `~/.config/etc/systemd/logind.conf.d/*.conf` → `/etc/systemd/logind.conf.d/`, then restarts
  `keyd`/re-execs `systemd-logind`.
- When editing keyd/logind behavior, edit `/etc/...` first (that's what's actually active),
  verify it works, then port the change back into this repo's `dot_config/etc/...` copy.

If this repo is meant to fully bootstrap a new machine unattended, the durable fix is either a
chezmoi `run_onchange_` script that `sudo cp`s these into place, or a separate root-targeted
chezmoi source state applied with `sudo chezmoi apply --source <dir> --destination /`
(`destDir` defaults to `$HOME`, so reaching `/etc` requires overriding it explicitly — this is
unrelated to `.chezmoiroot`, which only relocates where *this* source state lives within the
git repo and never changes the destination). Flag this if asked to touch keyd or logind config.

## Other things worth knowing

- `dot_config/sway/config` hardcodes two specific monitor outputs (a Lenovo P24QD-40 and an
  `eDP-1` laptop panel at a specific position/resolution). This config will misbehave on a
  machine with different outputs — the `output` lines need adjusting per-machine.
- `.gitignore` excludes `.env` preemptively; no `.env` currently exists in the repo.

## Cross-OS handling (macOS + Arch Linux)

`.chezmoiignore.tmpl` is templated (Go text/template, using chezmoi's built-in
`.chezmoi.os` variable — `"darwin"` on macOS, `"linux"` here) to skip Linux-only configs
on macOS: `.config/sway`, `.config/etc/keyd`, `.config/etc/systemd/logind.conf.d`. This is
the *only* templating in the repo so far — every other managed file is still applied
identically on both machines, byte-for-byte, with no `.tmpl` extension or `.chezmoidata`.

If a specific file's *content* needs to differ per-OS (not just whether it's applied at
all — e.g. package manager paths in `dot_config/bash/rc`), convert that individual file to
a `.tmpl` and branch inside it with `{{ if eq .chezmoi.os "darwin" }}...{{ else }}...{{ end }}`,
following the same pattern as `.chezmoiignore.tmpl`. Don't add this speculatively — only
when a real per-machine difference shows up.
