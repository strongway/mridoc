---
title: tmux
draft: false
tags:
  - linux
  - server
  - workflow
---

# tmux - Terminal Multiplexer

A comprehensive guide to tmux for managing multiple terminal sessions efficiently, with notes for fMRI workflows in the MSense Lab.

## Why tmux for fMRI work?

If you SSH into a lab workstation and run `fmriprep` in the foreground, the moment your laptop sleeps, your VPN drops, or the WiFi blinks, the SSH connection dies and your job dies with it. tmux solves this by keeping your shell alive on the server: you detach, close the laptop, walk home, and re-attach the next morning with everything still running.

Typical lab use cases:

- Running `fmriprep`, `mriqc`, or `fitlins` on a workstation overnight or across a weekend.
- Watching log files (`tail -f`) of submitted SLURM jobs without keeping a laptop awake.
- Running interactive `nilearn` Jupyter sessions on a remote server with port forwarding.
- Keeping a long `rsync` or `datalad get` of BIDS data running in the background.

When **not** to use tmux: real cluster batch jobs belong in SLURM `sbatch` scripts, not tmux + a foreground process on a login node. tmux is for *interactive* long-running things on a workstation or for monitoring jobs on the cluster head node.

> [!info] Related tutorials
> For true cluster batch submission, see [[LRZ Cloud Computing]]. For non-interactive container runs of fMRIPrep, see [[2.3 fmriPrep with Docker]] and [[2.4 Quality Control with MRIQC]]. Basic shell knowledge is covered in [[Basic Linux Commands]].

## Quick-start for fMRI workflows

The bare minimum a student needs to know:

```bash
# On the server: start a named session for your analysis
tmux new -s fmriprep_sub01

# Inside tmux, launch your job
apptainer run --cleanenv ... fmriprep ... &

# Detach (job keeps running): Ctrl-b then d
# Reconnect from anywhere: tmux attach -t fmriprep_sub01
# List sessions:           tmux ls
# Kill a finished session: tmux kill-session -t fmriprep_sub01
```

> [!tip] Name your sessions descriptively
> Use `fmriprep_sub01`, `mriqc_pilot`, `fitlins_glm1` — not `mywork1` or `test`. After a week with eight sessions running, future-you will be grateful. `tmux ls` only shows names; it can't tell you what's inside.

> [!warning] tmux is NOT a substitute for SLURM on the cluster
> On the LRZ cluster, login nodes are not for compute. Use `sbatch` (see [[LRZ Cloud Computing]]). tmux is fine on the head node only for editing config and watching `squeue`. Long compute belongs in a SLURM batch script.

## Table of Contents

- [What is tmux?](#what-is-tmux)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Basic Commands](#basic-commands)
- [Key Bindings](#key-bindings)
- [Configuration](#configuration)
- [Common Workflows](#common-workflows)
- [Advanced Features](#advanced-features)
- [Troubleshooting](#troubleshooting)
- [Quick reference for the lab](#quick-reference-for-the-lab)

---

## What is tmux?

**tmux** (terminal multiplexer) enables:
- Multiple terminal sessions in one window
- Sessions that persist when you disconnect
- Split panes (horizontal/vertical)
- Multiple windows per session
- Session sharing between users
- Detaching and reattaching to running sessions

**Perfect for:**
- Remote server work (SSH sessions that survive disconnection)
- Long-running processes
- Organizing multiple projects
- Pairing/collaboration

---

## Installation

### macOS
```bash
brew install tmux
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install tmux
```

### Linux (Fedora/RHEL)
```bash
sudo dnf install tmux
```

### Verify installation
```bash
tmux -V  # Should show version (e.g., tmux 3.3a)
```

---

## Core Concepts

### Hierarchy
```text
Session (entire tmux instance)
  └── Window (like tabs in a browser)
        └── Pane (split sections within a window)
```

### Sessions
- Named containers for your work
- Persist even when you disconnect
- Can have multiple windows

### Windows
- Like tabs in a browser
- Each window fills the entire terminal
- Navigate between them quickly

### Panes
- Split a window into multiple sections
- Each pane runs its own shell
- View multiple things simultaneously

### Prefix Key
- Default: `Ctrl+b` (written as `C-b`)
- Press prefix, then the command key
- Example: `C-b c` means press `Ctrl+b`, release, then press `c`

---

## Basic Commands

### Session Management

| Command | Description |
|---------|-------------|
| `tmux` | Start new session |
| `tmux new -s name` | Create named session |
| `tmux ls` | List sessions |
| `tmux attach` | Attach to last session |
| `tmux attach -t name` | Attach to named session |
| `tmux kill-session -t name` | Kill named session |
| `tmux kill-server` | Kill all sessions |

### Inside tmux (prefix + key)

**Session commands:**
- `C-b d` - Detach from session
- `C-b $` - Rename session
- `C-b s` - List sessions (interactive)
- `C-b (` - Previous session
- `C-b )` - Next session

**Window commands:**
- `C-b c` - Create new window
- `C-b ,` - Rename window
- `C-b w` - List windows (interactive)
- `C-b n` - Next window
- `C-b p` - Previous window
- `C-b 0-9` - Jump to window number
- `C-b &` - Kill window (with confirmation)

**Pane commands:**
- `C-b %` - Split vertically (left/right)
- `C-b "` - Split horizontally (top/bottom)
- `C-b o` - Cycle through panes
- `C-b arrow` - Navigate panes with arrow keys
- `C-b x` - Kill current pane (with confirmation)
- `C-b z` - Toggle pane zoom (full screen/back)
- `C-b {` - Move pane left
- `C-b }` - Move pane right
- `C-b !` - Convert pane to window
- `C-b C-o` - Rotate panes

**Other useful commands:**
- `C-b ?` - Show all key bindings
- `C-b :` - Enter command mode
- `C-b [` - Enter copy mode (scroll/search)
- `C-b ]` - Paste buffer
- `C-b t` - Show clock

---

## Key Bindings

### Copy Mode (scrolling and copying)

Enter copy mode: `C-b [`

**Navigation (default):**
- Arrow keys - Move cursor
- `q` - Quit copy mode
- `g` - Go to top
- `G` - Go to bottom
- `/` - Search forward
- `?` - Search backward
- `n` - Next search result
- `Space` - Start selection
- `Enter` - Copy selection
- `Esc` - Cancel

**With vi mode (see configuration):**
- `h/j/k/l` - Move cursor (left/down/up/right)
- `w/b` - Word forward/backward
- `v` - Begin selection
- `y` - Copy selection

### Pane Resizing

Enter command mode: `C-b :`

```bash
resize-pane -D 5  # Down 5 lines
resize-pane -U 5  # Up 5 lines
resize-pane -L 5  # Left 5 columns
resize-pane -R 5  # Right 5 columns
```

Or hold `C-b` and press arrow keys repeatedly.

---

## Configuration

Create `~/.tmux.conf` for custom settings:

```bash
# Basic Settings
set -g default-terminal "screen-256color"  # Enable 256 colors
set -g history-limit 10000                 # Increase history
set -g base-index 1                        # Start window numbering at 1
setw -g pane-base-index 1                  # Start pane numbering at 1
set -g renumber-windows on                 # Renumber windows on close

# Enable mouse support
set -g mouse on

# Change prefix from C-b to C-a (optional, like screen)
# unbind C-b
# set -g prefix C-a
# bind C-a send-prefix

# Split panes using | and - (more intuitive)
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Reload config easily
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Vi mode for copy
setw -g mode-keys vi
bind-key -T copy-mode-vi 'v' send -X begin-selection
bind-key -T copy-mode-vi 'y' send -X copy-selection-and-cancel

# Pane navigation with vim-like keys
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Pane resizing with vim-like keys
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# Status bar customization
set -g status-position bottom
set -g status-style 'bg=colour234 fg=colour137'
set -g status-left '[#S] '
set -g status-right '#[fg=colour233,bg=colour241] %d/%m #[fg=colour233,bg=colour245] %H:%M '
set -g status-right-length 50
set -g status-left-length 20

# Window status
setw -g window-status-current-style 'fg=colour1 bg=colour19 bold'
setw -g window-status-current-format ' #I#[fg=colour249]:#[fg=colour255]#W#[fg=colour249]#F '
setw -g window-status-style 'fg=colour9 bg=colour18'
setw -g window-status-format ' #I#[fg=colour237]:#[fg=colour250]#W#[fg=colour244]#F '

# Pane borders
set -g pane-border-style 'fg=colour238'
set -g pane-active-border-style 'fg=colour51'

# No delay for escape key press
set -sg escape-time 0
```

**Apply configuration:**
```bash
tmux source-file ~/.tmux.conf
```

Or inside tmux: `C-b :` then type `source-file ~/.tmux.conf`

---

## Common Workflows

### Remote Development
```bash
# Connect to server
ssh user@server

# Create/attach to session
tmux new -s work
# or
tmux attach -t work

# Do your work...
# Disconnect safely: C-b d

# Later, reconnect
ssh user@server
tmux attach -t work  # Everything still running!
```

### Multi-Project Setup
```bash
# Create sessions for different projects
tmux new -s frontend
tmux new -s backend
tmux new -s database

# Switch between them
tmux attach -t frontend
# Inside tmux: C-b s (interactive session list)
```

### Split Terminal Workflow
```bash
# Start session
tmux new -s dev

# Split horizontally for editor + terminal
C-b "

# Split the bottom pane vertically for logs + tests
C-b %

# Navigate: C-b arrow keys
# Resize: C-b : then resize-pane commands
```

### Typical Development Layout
```text
┌──────────────────────────────────┐
│  Editor (vim/emacs/etc)          │
│                                  │
│                                  │
├──────────────────┬───────────────┤
│  Build/Run       │  Git/Tests    │
│                  │               │
└──────────────────┴───────────────┘
```

Create with:
```bash
tmux new -s dev
C-b "          # Split horizontal
C-b %          # Split bottom pane vertical
```

---

## Advanced Features

### Session Persistence with tmux-resurrect

Install TPM (Tmux Plugin Manager):
```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Add to `~/.tmux.conf`:
```bash
# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'

# Initialize TMUX plugin manager (keep at bottom)
run '~/.tmux/plugins/tpm/tpm'
```

- `C-b I` - Install plugins
- `C-b C-s` - Save session
- `C-b C-r` - Restore session

### Scripted Session Creation

Create `~/tmux-dev.sh`:
```bash
#!/bin/bash
SESSION="dev"

tmux has-session -t $SESSION 2>/dev/null

if [ $? != 0 ]; then
  # Create session
  tmux new-session -d -s $SESSION -n editor

  # Editor window
  tmux send-keys -t $SESSION:editor "cd ~/projects && vim" C-m

  # Create terminal window
  tmux new-window -t $SESSION -n terminal
  tmux send-keys -t $SESSION:terminal "cd ~/projects" C-m

  # Create server window with splits
  tmux new-window -t $SESSION -n server
  tmux split-window -h -t $SESSION:server
  tmux send-keys -t $SESSION:server.0 "npm run dev" C-m
  tmux send-keys -t $SESSION:server.1 "npm run test:watch" C-m
fi

tmux attach -t $SESSION
```

Usage:
```bash
chmod +x ~/tmux-dev.sh
~/tmux-dev.sh
```

### Command Mode Examples

Enter command mode: `C-b :`

```bash
# Join panes from other windows
join-pane -s :2.0           # Bring pane 0 from window 2

# Break pane into new window
break-pane -t :3            # Move current pane to window 3

# Swap windows
swap-window -s 2 -t 1       # Swap window 2 with window 1

# Synchronize panes (type in all at once)
setw synchronize-panes on   # Turn on
setw synchronize-panes off  # Turn off
```

### Copy to System Clipboard (macOS)

Add to `~/.tmux.conf`:
```bash
# macOS clipboard integration
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "pbcopy"
```

### Copy to System Clipboard (Linux)

```bash
# Install xclip
sudo apt install xclip

# Add to ~/.tmux.conf
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -in -selection clipboard"
```

---

## Troubleshooting

### Colors Look Wrong
```bash
# Add to ~/.bashrc or ~/.zshrc
export TERM=screen-256color

# Or in ~/.tmux.conf
set -g default-terminal "screen-256color"
```

### Can't Scroll with Mouse
```bash
# Enable in ~/.tmux.conf
set -g mouse on
```

### Prefix Key Not Working
```bash
# Check current prefix
tmux show-option -g prefix

# Reset to default (in ~/.tmux.conf)
set -g prefix C-b
bind C-b send-prefix
```

### Session Won't Attach
```bash
# List all sessions
tmux ls

# Kill stuck session
tmux kill-session -t session-name

# Nuclear option (kill all)
tmux kill-server
```

### Panes Not Splitting Correctly
```bash
# Ensure no conflicting bindings in ~/.tmux.conf
# Test with default keys first: C-b % and C-b "
```

### Copy Mode Not Working
```bash
# Enter copy mode: C-b [
# If vi keys don't work, check ~/.tmux.conf:
setw -g mode-keys vi  # For vi mode
# or
setw -g mode-keys emacs  # For emacs mode (default)
```

### Detached Session Won't Die
```bash
# Force kill
tmux kill-session -t session-name

# Or find and kill process
ps aux | grep tmux
kill -9 <PID>
```

---

## Quick Reference Card

```text
SESSION MANAGEMENT
  tmux                  Start new session
  tmux new -s name      Named session
  tmux ls               List sessions
  tmux attach           Attach last
  tmux attach -t name   Attach named
  C-b d                 Detach
  C-b $                 Rename session
  C-b s                 Session list

WINDOW MANAGEMENT
  C-b c                 New window
  C-b ,                 Rename window
  C-b w                 Window list
  C-b n/p               Next/previous
  C-b 0-9               Jump to window #
  C-b &                 Kill window

PANE MANAGEMENT
  C-b %                 Split vertical
  C-b "                 Split horizontal
  C-b arrow             Navigate
  C-b o                 Cycle panes
  C-b z                 Toggle zoom
  C-b x                 Kill pane
  C-b !                 Pane → window

COPY MODE
  C-b [                 Enter copy mode
  q                     Quit
  Space                 Start selection
  Enter                 Copy
  C-b ]                 Paste

HELP
  C-b ?                 Show all bindings
  C-b :                 Command mode
```

---

## Resources

- **Official Documentation**: https://github.com/tmux/tmux/wiki
- **Man Page**: `man tmux`
- **Cheat Sheet**: https://tmuxcheatsheet.com/
- **Book**: "tmux 2: Productive Mouse-Free Development" by Brian P. Hogan
- **Community**: r/tmux on Reddit

---

## Tips & Best Practices

1. **Name your sessions** - Easier to remember than numbers
2. **Use mouse mode** - Great for beginners (can disable later)
3. **Learn copy mode** - Essential for working with output
4. **Customize your prefix** - Some prefer `C-a` (like screen)
5. **Use plugins sparingly** - Start simple, add as needed
6. **Script common layouts** - Save time on repetitive setups
7. **Keep sessions focused** - One per project/task
8. **Practice key bindings** - Muscle memory makes you fast
9. **Read logs in copy mode** - Don't pipe to less/more
10. **Detach, don't kill** - Preserve your work

---

## Quick reference for the lab

The 90% of tmux you will actually use for fMRI analyses:

| Command / keybinding         | What it does                                            |
| ---------------------------- | ------------------------------------------------------- |
| `tmux new -s <name>`         | Start a new named session (e.g. `fmriprep_sub01`)       |
| `tmux ls`                    | List all running sessions on this server                |
| `tmux attach -t <name>`      | Re-attach to a session after SSH'ing back in            |
| `C-b d`                      | Detach (your job keeps running)                         |
| `C-b "`                      | Split pane horizontally — useful for `htop` + job logs  |
| `C-b %`                      | Split pane vertically                                   |
| `C-b [`                      | Enter scroll/copy mode — read back long fMRIPrep output |
| `tmux kill-session -t <name>` | Kill a finished session and free its memory             |

> [!tip] A common lab layout
> One pane running `fmriprep` (or `mriqc`), one pane with `htop` or `nvidia-smi` to watch resources, one pane with `tail -f` on the fMRIPrep log. Detach overnight; come back, attach, scroll up in copy mode (`C-b [`) to inspect what happened.

---

**Last updated**: 2026-01-22
