[![Claude Code View Marketplace](https://img.shields.io/badge/Claude%20Code-View%20Marketplace-blue?style=for-the-badge&logo=github)](https://github.com/danielrosehill/Claude-Code-Plugins)

## Linux Desktop Management Plugin

This Claude Code plugin is part of my public marketplace.

**To install the marketplace:**

```bash
/plugin marketplace add https://github.com/danielrosehill/Claude-Code-Plugins
```

**Plugin Installation:**

To install this plugin:

```bash
/plugin install linux-desktop-mgmt@danielrosehill
```

---

Claude Code plugin for linux-desktop slash commands and agents.

## Pairs with: desktop-ops-workspace-template

The plugin runs operations. [`desktop-ops-workspace-template`](https://github.com/danielrosehill/desktop-ops-workspace-template) remembers them — a per-machine, local-first SQLite log of every install, fix, reorg, and incident, so future sessions can query prior history before acting.

The plugin works standalone. When Claude is running inside a workspace (`DESKTOP_OPS_WORKSPACE` set), the workspace's `CLAUDE.md` instructs Claude to log each op into `ops.db`. No plugin-side configuration required.

```bash
gh repo create my-desktop-ops \
  --template danielrosehill/desktop-ops-workspace-template \
  --private --clone
cd my-desktop-ops
./scripts/init-db.sh && ./scripts/register-machine.sh
# Now open Claude Code here and use plugin commands as normal.
```


## Installation

```bash
/plugin marketplace add danielrosehill/CC-Plugin-linux-desktop
/plugin install linux-desktop@danielrosehill
```

## Author

**Daniel Rosehill**
- Website: [danielrosehill.com](https://danielrosehill.com)
- Email: public@danielrosehill.com
- GitHub: [@danielrosehill](https://github.com/danielrosehill)

## License

This plugin is licensed under the MIT License.
