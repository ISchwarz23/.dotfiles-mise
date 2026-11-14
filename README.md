# .dotfiles-mise

Personal dotfiles managed with [mise](https://mise.jdx.dev) — a polyglot runtime manager that also handles dotfiles, system packages, and machine bootstrapping.

## Quick Start (New Machine)

### One-liner: Install mise + Bootstrap from GitHub

```bash
# HTTPS (with -y to skip all prompts)
curl -fsSL https://mise.jdx.dev/install.sh | bash && ~/.local/bin/mise bootstrap --from git@github.com:ISchwarz23/.dotfiles-mise.git --yes

# Or SSH (if you have SSH keys set up)
curl -fsSL https://mise.jdx.dev/install.sh | bash && ~/.local/bin/mise bootstrap --from git@github.com:ISchwarz23/.dotfiles-mise.git --yes
```

This single command:
1. Installs mise to `~/.local/bin/mise`
2. Clones this repo to `~/.dotfiles-mise`
3. Links `~/.config/mise` → `~/.dotfiles-mise/.config/mise`
4. Runs full bootstrap (packages, repos, dotfiles, shell activation, services)

---

### Step-by-Step (Manual)

#### 1. Install mise

```bash
# Linux / macOS / WSL
curl -fsSL https://mise.jdx.dev/install.sh | bash

# Or via package manager
# Arch: sudo pacman -S mise
# Homebrew: brew install mise
# Fedora: sudo dnf install mise
```

Restart your shell or run:
```bash
eval "$(~/.local/bin/mise activate zsh)"  # or bash
```

#### 2. Bootstrap from this repository

```bash
# From GitHub (HTTPS) - with -y to skip prompts
mise bootstrap --from https://github.com/ISchwarz23/.dotfiles-mise.git --yes

# From GitHub (SSH)
mise bootstrap --from git@github.com:ISchwarz23/.dotfiles-mise.git --yes

# From local clone (if already cloned)
cd ~/.dotfiles && mise bootstrap --yes
```

#### 3. Verify

```bash
# Check mise is active
echo $MISE_SHELL  # should show "zsh" or "bash"

# Check dotfiles applied
ls -la ~/.gitconfig ~/.config/nvim ~/.config/tmux ~/.config/opencode

# Check tools installed
mise list
```

---

## What Gets Configured

| Category | Tools / Files |
|----------|---------------|
| **Shell** | zsh + Oh My Zsh, aliases, custom themes |
| **Editor** | Neovim (Lua config), opencode theme |
| **Terminal** | tmux + TPM, kitty, Hyprland/Waybar configs |
| **Git** | Config with aliases, fetch all branches, prune |
| **AI Tools** | opencode (with Matt Pocock skills plugin) |
| **Mobile** | Termux config (opt-in via `mise run setup-termux`) |
| **Packages** | curl, git, jq, neovim, tmux, podman, kubectl, opencode |
| **Services** | mise history watcher (auto-commits dotfile changes) |

---

## Daily Usage

### Update dotfiles + re-apply

```bash
cd ~/.dotfiles && git pull && mise bootstrap --update --yes
```

### Apply only dotfiles (skip packages)

```bash
mise dot apply --force
```

### Run specific setup tasks

```bash
mise run basic-setup       # Core tools + shell + editor + tmux
mise run web-dev-setup     # + Node.js via nvm
mise run dev-ops-setup     # + podman + kubectl
mise run full-setup        # Everything
mise run setup-termux      # Apply Termux config (Android only)
```

### Install additional tools

```bash
mise use -g node@lts python@latest go@latest
mise install  # installs all tools from mise.toml
```

---

## Repository Structure

```
.dotfiles/
├── mise.toml                    # Main bootstrap config
├── git/
│   ├── .gitconfig               # Main git config (copied, machine-specific)
│   └── .gitconfig-general       # Shared git config (symlinked, synced)
├── nvim/.config/nvim/           # Neovim config
├── tmux/.config/tmux/           # tmux config
├── opencode/.config/opencode/   # opencode theme + plugin config
├── termux/.termux/              # Termux (Android) config
├── zsh/
│   ├── .zshrc                   # zsh config
│   └── .alias                   # Shared aliases (fallback)
├── bash/.alias                  # Shared aliases for bash
└── README.md                    # This file
```

---

## Key Concepts

### Mise Activation
Adds `eval "$(mise activate zsh)"` to `~/.zshrc` and `eval "$(mise activate bash)"` to `~/.bashrc`. This enables:
- Automatic tool version switching per project
- Project-local environment variables
- Shell aliases from `[shell_alias]` in mise.toml

### Dotfiles Modes
- **copy** — Machine-specific files (`.gitconfig`, `.zshrc`) — changes don't sync back
- **symlink** — Shared configs (`.gitconfig-general`, nvim, tmux) — always in sync with repo

### Git Config Strategy
```
~/.gitconfig              # copy — machine-specific, includes general
~/.gitconfig-general      # symlink → repo — shared aliases, settings, fetch config
```
Run `git config user.name "Name"` → writes to `~/.gitconfig` (local only).

---

## Multi-OS Support

Packages declared for multiple managers — mise uses first available:

```toml
"pacman:curl" = "latest"
"apt:curl" = "latest"
"dnf:curl" = "latest"
"brew:curl" = "latest"
# ...
```

Works on: **Arch**, **Ubuntu/Debian**, **Fedora/RHEL**, **macOS**, **Alpine**, **openSUSE**.

---

## Opencode Skills

Matt Pocock's engineering skills auto-installed via plugin:

```json
// ~/.config/opencode/opencode.jsonc
{
  "plugin": ["github:kevbob1/opencode-mattpocock-skills"]
}
```

Skills: `grilling`, `tdd`, `domain-modeling`, `codebase-design`, `diagnosing-bugs`, `write-a-skill`, and more.

---

## Troubleshooting

### "Config not trusted"
```bash
mise trust ~/.dotfiles/mise.toml
```

### Dotfiles conflict (already exist)
```bash
mise dot apply --force
# or
mise bootstrap --force-dotfiles
```

### Homebrew permission errors (Linuxbrew)
```bash
sudo chown -R $(whoami) /home/linuxbrew/.linuxbrew
```

### Termux config not applied
```bash
mise run setup-termux  # Run manually on Android/Termux
```

### "unknown bootstrap package manager 'yay'" warning (Arch users)
This is harmless — mise doesn't have the yay plugin installed by default. AUR packages still work.
```bash
# To enable yay support (optional)
mise plugin install package:yay

# Or ignore the warning — pacman handles all core packages
```
**Why:** The config lists `yay:tmux-xpanes` as an AUR-only package. If you don't need tmux-xpanes, you can remove that line.

---

## Customization

### Machine-specific overrides
Create `~/.config/mise/config.local.toml` (gitignored):
```toml
[bootstrap.packages]
"apt:docker-ce" = "latest"  # only on this machine

[env]
WORK_PROXY = "http://proxy:8080"
```

### Add project-specific tools
Create `mise.toml` in any project:
```toml
[tools]
node = "20"
python = "3.12"

[env]
DATABASE_URL = "postgres://..."
```

---

## Resources

- [mise documentation](https://mise.jdx.dev)
- [mise bootstrap guide](https://mise.jdx.dev/bootstrap/)
- [mise dotfiles guide](https://mise.jdx.dev/dotfiles.html)
- [Matt Pocock skills](https://github.com/mattpocock/skills)
- [opencode documentation](https://opencode.ai/docs)
