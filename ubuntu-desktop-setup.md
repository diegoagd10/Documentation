# Ubuntu Desktop Setup Guide

Essential configurations for development workflow.

## System Information

- **OS**: Ubuntu 24.04.3 LTS (Noble)
- **User**: diegoagd10

> For comprehensive tool installation guides, see [useful-commands.md](useful-commands.md)

## 1. Keyboard Configuration

### 1.1 Map Caps Lock to Escape

Perfect for Vim users and developers. This mapping persists across reboots.

```bash
gsettings set org.gnome.desktop.input-sources xkb-options "['caps:escape']"
```

**Verify the mapping:**
```bash
gsettings get org.gnome.desktop.input-sources xkb-options
# Should output: ['caps:escape']
```

**Alternative options:**

Swap Caps Lock and Escape:
```bash
gsettings set org.gnome.desktop.input-sources xkb-options "['caps:swapescape']"
```

Make Caps Lock an additional Escape (keep Caps Lock functionality with Shift+Caps):
```bash
gsettings set org.gnome.desktop.input-sources xkb-options "['caps:escape_shifted_capslock']"
```

**Revert to default:**
```bash
gsettings set org.gnome.desktop.input-sources xkb-options "[]"
```

## 2. SSH Keys

Generate SSH key for GitHub/GitLab and server access:

```bash
# Generate Ed25519 key (modern, secure)
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard
cat ~/.ssh/id_ed25519.pub
```

Add the public key to your GitHub/GitLab account and server `~/.ssh/authorized_keys` files.

## 3. tmux Auto-Start Configuration

Automatically start tmux when opening a new terminal. This provides session persistence and powerful terminal multiplexing.

**Install tmux:**
```bash
sudo apt install -y tmux
```

**Configure auto-start in Oh My Zsh:**

Add the following to the end of your `~/.zshrc` file:

```bash
# Auto-start tmux
if command -v tmux &> /dev/null && [ -n "$PS1" ] && [[ ! "$TERM" =~ screen ]] && [[ ! "$TERM" =~ tmux ]] && [ -z "$TMUX" ]; then
  exec tmux
fi
```

**What this does:**
- Automatically starts tmux when you open a new terminal
- Avoids nested tmux sessions (won't start if you're already inside tmux)
- Only runs in interactive shells (checks `$PS1`)
- Uses `exec` to replace the shell process with tmux (cleaner exit behavior)

**Apply the changes:**
```bash
source ~/.zshrc
```

**Alternative: Attach to existing session**

To always attach to a persistent session named "main" (or create it if it doesn't exist), use:

```bash
# Replace the auto-start command with:
exec tmux new-session -A -s main
```

**Alternative: Use oh-my-zsh tmux plugin**

1. Change `plugins=(git)` to `plugins=(git tmux)` in `~/.zshrc`
2. Configure plugin variables before sourcing oh-my-zsh:
   ```bash
   ZSH_TMUX_AUTOSTART=true
   ZSH_TMUX_AUTOCONNECT=true
   ```

**Basic tmux commands:**
- `Ctrl+b d` - Detach from session
- `Ctrl+b c` - Create new window
- `Ctrl+b n` - Next window
- `Ctrl+b p` - Previous window
- `Ctrl+b %` - Split pane vertically
- `Ctrl+b "` - Split pane horizontally
- `tmux ls` - List sessions
- `tmux attach -t <session-name>` - Attach to session

## 4. Useful Shell Aliases

Add these convenience aliases to your `~/.zshrc` file for improved workflow:

```bash
# Aliases
alias pbcopy='xclip -selection clipboard'
alias pbpaste='xclip -selection clipboard -o'
alias bat='batcat'
```

**What these do:**
- `pbcopy` - macOS-style clipboard copy (requires `xclip`)
- `pbpaste` - macOS-style clipboard paste (requires `xclip`)
- `bat` - shorthand for `batcat` (syntax-highlighted cat alternative)

**Install required packages:**
```bash
sudo apt install -y xclip bat
```

**Apply the changes:**
```bash
source ~/.zshrc
```

---

**Last Updated**: November 2025
**System**: Ubuntu 24.04.3 LTS (Noble)
