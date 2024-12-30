# Tmux

## Source
[https://www.youtube.com/watch?v=DzNmUNvnB04](https://www.youtube.com/watch?v=DzNmUNvnB04)

## Config file
Edit your .tmux.conf file
``` bash
vi ~/.tmux.conf 
```

.tmux.conf contents:
```
# Enable true color
set-option -sa terminal-overrides ",xterm*:Tc"

# Enable mouse support
set -g mouse on

# Start windows and pane numbering at 1, not 0
set -g base-index 1
set -g pane-base-index 1
set-window-option -g pane-base-index 1

# Split windows into current working directory
bind '"' split-window -v -c "#{pane_current_path}"
bind % split-window -h -c "#{pane_current_path}"

# Only display hostname right side of statusbar
set -g status-right '#H'

# Set statusbar colors
set -g status-bg "#33658A"
set -g status-fg white
```

## Cheat-sheet
### Sessions
- tmux to create a session
- Ctrl + b + d to detach from current session
- tmux attach to attach to most recent session
- tmux a shorthand to attach to most recent session
- tmux new -s mysessionname to start a now session named mysessionname
- tmux attach -t mysessionname to attach to a running session named mysessionname
- tmux a -t mysessionname shorthand to attach to a running session named mysessionname
- tmux ls to list sessions

### Windows
- CTRL-b c to create a new window
- CTRL-b n to switch to the next window
- CTRL-b p to switch to previous window
- CTRL-b w to show window list

### Panes
- CTRL-b + % to split the current pane vertically
- CTRL-b " to split the current pane horizontally
- CTRL-b ALT-← to move pane divider to the left
- CTRL-b ALT-→ to move pane divider to the right
- CTRL-b ALT-↑ to move pane divider up
- CTRL-b ALT-↓ to move pane divider down
- CTRL-b x to close the current pane
- CTRL-b o to switch to next pane
- CTRL-b ← → ↑ ↓ to switch panes
- CTRL-b x to close current pane

### Copy-mode
- CTRL-b [ to enter copy mode
- q to exit
