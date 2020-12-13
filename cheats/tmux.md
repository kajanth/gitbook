# Tmux

## Install 

```text
apt update
apt install tmux
```

## Sessions

### Create new

```text
tmux new -s my-session
```

### Use existing session

```text
tmux new -A -s my-session
```

Or you can list all sessions:

```text
tmux ls
```

And attache to an existing one:

```text
tmux attach-session -t my-session
```

### Delete/Kill session

#### Kill all sessions

```bash
tmux kill-server
```

#### Kill specific session

```text
tmux kill-session -t my-session
```

#### Kill other sessions

If you are inside a tmux session you would like to keep and kill all others, run:

```text
tmux kill-session -a
```

## Windows and Panes

* `Ctrl+b` `c` Create a new window \(with shell\)
* `Ctrl+b` `w` Choose window from a list
* `Ctrl+b` `0` Switch to window 0 \(by number \)
* `Ctrl+b` `,` Rename the current window
* `Ctrl+b` `%` Split current pane vertically into two panes
* `Ctrl+b` `"` Split current pane horizontally into two panes
* `Ctrl+b` `o` Go to the next pane
* `Ctrl+b` `;` Toggle between the current and previous pane
* `Ctrl+b` `x` Close the current pane
* Enable `synchronize-panes`: `ctrl+b` then `shift :`. Then type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.
* `Ctrl+b` `[` To enable copy mode. Use arrows to scroll.
* `Ctrl+b` `Alt + 1` All panels on vertical with same width.
  * Or run `Ctrl+b` `Shift + :` `select-layout even-vertical` 
* `Ctrl+b` `Alt + 2` All panels on horizontal with same height.
  * Or run `Ctrl+b` `Shift + :` `select-layout even-horizontal`

