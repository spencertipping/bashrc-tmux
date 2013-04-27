# Auto-`tmux` for SSH logins
A solution to the classic problem: you SSH somewhere, start something, and you
forgot to run it inside `screen` or `tmux` and need to leave. This script
solves that for you, re-attaching to your session if you later SSH in from
someplace else. To use, make the top of your `.bashrc` look like this:

```sh
[ -z "$PS1" ] && return                 # this still comes first
source path/to/bashrc-tmux

# rest of bashrc below...
```

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
