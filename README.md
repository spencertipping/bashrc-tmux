# Auto-`tmux` for SSH logins
A solution to the classic problem: you SSH somewhere, start something, and you
forgot to run it inside `screen` or `tmux` and need to leave (**protip:**
`reptyr` can save your day in a pinch). This script solves that for you,
re-attaching to your session if you later SSH in from someplace else. To use,
make the top of your `.bashrc` look like this:

```sh
[ -z "$PS1" ] && return                 # this still comes first
source path/to/bashrc-tmux

# rest of bashrc below...
```

**Note:** Also works with zsh.

## Behavior
`bashrc-tmux` won't do anything for local shells. It kicks in when:

1. You are logging in via SSH, and
2. You have `tmux` installed on the machine.

If both of these things are true, it will create a special tmux session called
`ssh-$USER` (where `$USER` is your username on the server) and drop you into a
throwaway session linked to `ssh-$USER`. This linkage lets you change windows
independently among multiple logins, but all windows themselves are shared.

There are two ways to logout of this configuration. To leave the shell running,
you can use the `kill-session` command [sic!]. This kills the throwaway session
and leaves the `ssh-$USER` session running in the background. When you want to
get rid of all sessions, you can just `^D` out of all shells in all windows,
which will cause tmux to kill the `ssh-$USER` session and any open throwaways,
forcing a logout.

You can also create isolated sessions; see
[below](#multiple-remote-tmux-sessions) for details.

## Example
Note: this example assumes `^A` is the escape key, not tmux's default of `^B`.

```sh
$ ssh foo               # creates tmux session ssh-spencertipping on foo
foo$ tmux ls            # we are now in tmux
ssh-spencertipping: 1 windows ... (group 0)
ssh-spencertipping-1: 1 windows ... (group 0) (attached)
foo$ du -sh / &         # start something that will take a while
foo$ ^Ak                # kill the throwaway session to logout
$ ssh foo               # ssh-spencertipping already exists, so reuse it
foo$ jobs
[1]+  Running             du -sh / &
foo$ fg                 # reattach to job from last tmux
^C
foo$ ^D                 # exit shell, killing all tmuxen and logging out
```

## Multiple remote tmux sessions
Normally you'll share tabs with existing SSH connections if you log into a
server; but you can specify a session suffix if you want to. This will create a
new set of shared tabs for anyone logging into that session suffix:

```sh
$ ssh -t foo BASHRC_TMUX_SESSION=bar bash
foo$ tmux ls
ssh-spencertipping-bar: 1 windows ... (group 0)
ssh-spencertipping-bar-1: 1 windows ... (group 0) (attached)
```

## xpra extension
`xpra` is to X11 what `tmux` is to ssh, and you can source `bashrc-xpra` to
enable it in similar situations. You can then temporarily attach to the remote
display by saying this:

```sh
$ xpra attach ssh:remote-system:100
```
