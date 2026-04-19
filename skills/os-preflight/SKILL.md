---
name: os-preflight
description: Detect the host OS, distribution, desktop environment, init system, and tool availability before running commands from this plugin. Use at the start of any plugin command that assumes a specific distro (Debian/Ubuntu apt), desktop environment (KDE/Plasma, GNOME), or tool (kwin, systemctl, snapper, nvidia-smi). Fail clearly with actionable guidance instead of running commands that will error obscurely on an unsupported system.
---

# os-preflight

Many commands in this plugin were originally written for Daniel's Ubuntu + KDE Plasma workstation. When a user on Fedora, Arch, GNOME, Wayland-without-KWin, or WSL invokes one of those commands, the command should detect the mismatch early and either:

- adapt (use the right package manager, the right window-manager tool), or
- decline cleanly with a message explaining what's needed.

This skill defines the preflight contract that other commands in the plugin rely on.

## What to detect

Before running environment-sensitive logic, gather:

| Facet | Source | Example values |
|---|---|---|
| Distro family | `/etc/os-release` → `ID`, `ID_LIKE` | `ubuntu`, `debian`, `fedora`, `arch`, `opensuse` |
| Distro version | `/etc/os-release` → `VERSION_ID` | `25.10`, `40`, `rolling` |
| Package manager | Derived from distro family | `apt`, `dnf`, `pacman`, `zypper` |
| Desktop environment | `$XDG_CURRENT_DESKTOP` (lowercased) | `kde`, `gnome`, `xfce`, `hyprland` |
| Session type | `$XDG_SESSION_TYPE` | `wayland`, `x11`, `tty` |
| Window manager | `$KDE_SESSION_VERSION` / `wmctrl -m` / process list | `kwin`, `mutter`, `sway` |
| Init system | `ps -p 1 -o comm=` | `systemd`, `openrc`, `runit` |
| Tool availability | `command -v <tool>` | boolean per tool the command needs |

Don't query everything every time — only the facets the calling command actually depends on.

## Preflight contract for commands

A command using this skill should, as its first step:

1. Declare its requirements as a short list: distro family (or "any"), DE (or "any"), required tools.
2. Run the relevant detections.
3. If requirements are met: proceed silently.
4. If unmet: emit a one-paragraph explanation naming the exact facet that failed and the closest equivalent on the detected system, then stop. Do **not** blindly substitute — `apt install` → `dnf install` sounds easy but package names differ.

Example preflight block a command should produce:

```
Preflight:
  - needs: debian-family (got: debian ✓)
  - needs: KDE/Plasma (got: KDE ✓)
  - needs: kwriteconfig6 (got: /usr/bin/kwriteconfig6 ✓)
  → proceeding.
```

Or on mismatch:

```
Preflight failed:
  - needs: debian-family with apt
  - got: fedora (uses dnf)
  This command uses `apt` to manage packages. On Fedora the equivalent is `dnf`, but
  package names differ (e.g. `libkf6-something` vs `kf6-something-devel`). Rather than
  guess, this command declines to run. If you'd like a Fedora port, open an issue.
```

## Detection snippets

These are the canonical one-liners commands should use. Keep them consistent so output is predictable.

```bash
# Distro family
. /etc/os-release 2>/dev/null
distro_id="${ID:-unknown}"
distro_like="${ID_LIKE:-}"
distro_version="${VERSION_ID:-unknown}"

# Package manager
if   command -v apt    >/dev/null 2>&1; then pm=apt
elif command -v dnf    >/dev/null 2>&1; then pm=dnf
elif command -v pacman >/dev/null 2>&1; then pm=pacman
elif command -v zypper >/dev/null 2>&1; then pm=zypper
else pm=unknown
fi

# Desktop environment
de="$(echo "${XDG_CURRENT_DESKTOP:-}" | tr '[:upper:]' '[:lower:]')"
session="${XDG_SESSION_TYPE:-unknown}"

# Init
init="$(ps -p 1 -o comm= 2>/dev/null)"

# Tool presence
have() { command -v "$1" >/dev/null 2>&1; }
```

## What commands should do with the result

- **Hard match required** (e.g. a `kwin`-specific fix): declare it, preflight, bail cleanly if mismatch.
- **Soft match** (e.g. "install a package"): preflight to pick the right package manager, then proceed with the adapted command.
- **Universal** (e.g. read a file): skip preflight entirely.

## Migration note

This skill is the target for an incremental refactor of the plugin's commands. Existing commands that hardcode `apt`, `kwin`, or `dbus-send` to the KDE session bus should gradually adopt this preflight. New commands should follow the contract from day one.

Mark migrated commands by including this line near the top of the command body:

```
Uses os-preflight: <comma-separated facet list, e.g. "distro=debian-family, de=kde, tools=kwriteconfig6">
```

So progress is greppable.
