# pi-tmux-status

A [pi](https://github.com/badlogic/pi-mono) extension that shows the agent's state
in your tmux window list, so you can tell at a glance whether it needs you.

| badge | state | meaning |
| --- | --- | --- |
| cog | `working` | the agent is running, or a subagent is still busy |
| hourglass | `waiting` | a subagent is stuck on a question and needs an answer |
| check | `done` | the turn finished and nothing is running |

`waiting` beats `working`, and `done` appears only when every subagent has
finished. `waiting` stays until the subagent is answered, so a glance at the
window never clears it. `done` clears when you focus the window, or after 5s if
you are already looking at it. Requires a Nerd Font.

## Install

1. Copy the extension into pi's global extensions directory:

   ```sh
   cp pi-tmux-status.ts ~/.pi/agent/extensions/
   ```

2. Add three lines to `~/.tmux.conf`, then run `tmux source-file ~/.tmux.conf`:

```tmux
# badge: working=cog, waiting=hourglass, done=check (Nerd Font glyphs)
set -g @tmux2k-window-list-format '#I:#W #{?@pi_status,#{?#{==:#{@pi_status},waiting},,#{?#{==:#{@pi_status},done},,}},}'
# clear "done" when the window is focused or selected; working/waiting stay
set-hook -g pane-focus-in 'if-shell -F "#{==:#{@pi_status},done}" "set-option -uw @pi_status"'
set-hook -g after-select-window 'if-shell -F "#{==:#{@pi_status},done}" "set-option -uw @pi_status"'
```

If you do not use the tmux2k status bar, put the badge expression into whatever
`window-status-format` your setup defines instead of `@tmux2k-window-list-format`.

## How it works

The extension writes one tmux window option, `@pi_status`, holding the state word
(`working`, `waiting`, `done`); your status bar format turns that word into a
glyph. Subagents run as panes in the same window, so the badge is an aggregate of
the session and all of its children, written by the parent process.
