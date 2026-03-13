---
date: "2026-02-26"
title: "Process management in the terminal with Ctrl + z"
categories: ["software"]
tags: ["bash", "terminal", "linux"]
---

A post to honor what is arguably the most versatile keyboard combination in the Linux terminal: <kbd>Ctrl</kbd> + <kbd>z</kbd>. Whether it is for quickly switching to a different process, or for killing unresponsive processes, <kbd>Ctrl</kbd> + <kbd>z</kbd> is your friend. In this post we will go over what it does and, in turn, what _you_ can do with it.

## <kbd>Ctrl</kbd> + <kbd>z</kbd> suspends a process

When you press the keyboard shortcut, the **currently active process is suspended** in the sense that:

1. Its execution is **paused**. Whatever the process was doing, it will be suspended in that state. It will no longer consume CPU, but it will still take up RAM.
2. Control is **returned to the terminal**.

Technical detail: <kbd>Ctrl</kbd> + <kbd>z</kbd> sends a [`SIGTSTP`](<https://en.wikipedia.org/wiki/Signal_(IPC)#POSIX_signals>) signal to foreground process, which works for Linux as well as any other Unix-based operating system.

For example, when running `python` and pressing <kbd>Ctrl</kbd> + <kbd>z</kbd>, you end up back at the terminal with the Python process being suspended to the background:

<div id="termynal" data-termynal>
    <span data-ty="input">python</span>
    <span data-ty>Python 3.14.3 (main, Feb 13 2026, 15:31:44)</span>
    <span data-ty="input" data-ty-prompt=">>>"> # Pressing Ctrl + z now...</span>
    <span data-ty data-ty-delay="100">[1]+&nbsp;&nbsp;Stopped&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;python</span>
    <span data-ty="input" data-ty-delay="3000">
</div>
<script src="/termynal.js" data-termynal-container="#termynal"></script>

So, how does that help you?

## Switching between processes with `fg`

The first use case is but a single step away: pressing <kbd>Ctrl</kbd> + <kbd>z</kbd> puts you back at the terminal, at which point you can run any command. The next step is to **switch back to the process you suspended** using the command **`fg`** (for foreground).

An example of when this method is useful is when you are writing a script and frequently need to check if it _finally_ works as intended. With the script opened in `vim`:

1. Make changes and save the script
1. Press <kbd>Ctrl</kbd> + <kbd>z</kbd> to go back to the terminal
1. Run your freshly updated script
1. **Type `fg`** and press <kbd>Enter</kbd> to **return to the suspended process** (`vim`)

## Killing an unresponsive process with `kill %1`

When aborting a script, <kbd>Ctrl</kbd> + <kbd>c</kbd> (technical name: `SIGINT`) is by far the most common keystroke used. It's quick and does the job, usually. Except when it does not. When the going gets tough and <kbd>Ctrl</kbd> + <kbd>c</kbd> is unable to terminate a process, its neighbour from two doors down still might.

For this, combine <kbd>Ctrl</kbd> + <kbd>z</kbd> with the command **`kill`** (which by default sends the `SIGTERM` signal) and the shorthand syntax for **"last active job" `%1`**. Usually, `kill` takes as an argument the process ID (PID). Looking that up takes some time, so instead we ask the shell to substitute the last active job.

An example use case is when using the tool Miniterm to communicate with an Arduino. The Miniterm tool might hang indefinitely if there is something wrong with the firmware. When that happens, <kbd>Ctrl</kbd> + <kbd>c</kbd> no longer works. What still works is:

1. Press <kbd>Ctrl</kbd> + <kbd>c</kbd> to suspend the process
1. Type `kill %1` followed by <kbd>Enter</kbd> to tell send the signal to Miniterm to terminate

> [!TIP]
> If `kill %1` does not terminate the process, a terminate signal is still too gentle. Try `kill -9 %1` for the scorched earth approach usinsg the kill signal.

Be aware that so far we have been assuming that the job ID is 1 (hence `%1`). If multiple jobs exist, the index can change as well. For example note the `[2]` in the stopped process below:

```
[2]+  Stopped                    uv run python -m serial.tools.miniterm
```

In this case, use `kill %2` instead.

## Bonus: running a process in the background

This last tip is a bit of a cheat: where `fg` puts a suspended process in the foreground, **`bg`** puts it in the **background**. It will, however, continue to generate output and thereby usually makes a big mess of the terminal, which is the reason I personally never use this.

The real tip: if you want to really run a process in the background, **look into the `screen`** command. That way, it will continue even when the terminal is closed.
