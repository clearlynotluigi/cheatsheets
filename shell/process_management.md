# Linux Process Management Cheatsheet ⚙️

> A practical guide to finding, inspecting, and terminating processes on the command line. Master the `ps`, `pgrep`, `pkill`, and `kill` commands.

---
### 🔍 Finding Processes
---
*Before you can manage a process, you need to find it.*

#### `ps aux`
*Lists all running processes on the system.*
> This is the classic, powerful command to see everything. It produces a lot of output, so it's almost always used with `grep`.
```shell
# Find all processes related to 'nginx'
ps aux | grep nginx
```

#### `pgrep`
*Searches for processes based on name and other attributes, and prints their PIDs.*
> This is a modern, more direct way to find a process ID (`PID`) for scripting.

##### `pgrep -l `
*Finds processes and lists their PID and name.*
```shell
# Find the PID and name of the ssh-agent process
pgrep -l ssh-agent
```

##### `pgrep -f `
*Searches the **full command line** for the pattern, not just the process name.*
> 🌟 **This is extremely powerful!** Use this when you need to find a process based on the arguments it was launched with.
```shell
# Find the process for a specific python script
pgrep -f "my_long_running_script.py"
```

---
### 🔬 The "Surgical Strike" Method: Chaining Commands
---
*This section expands on the powerful chain you provided. It's the most precise way to find a very specific process.*

Let's analyze your command:
```shell
pgrep -l -f ssh | grep -- "-o ExitOnForwardFailure=yes" | grep "9103:vpc-whalar" | awk '{print $1}'
```
> This chain is brilliant because it filters in stages.
> 1.  `pgrep -l -f ssh`: Find all processes whose command line contains `ssh` and list their PID and full command.
> 2.  `grep -- "-o ..."`: From that list, keep only the lines that *also* contain the specific SSH option.
> 3.  `grep "9103:vpc-whalar"`: From the remaining lines, keep only the one that *also* contains the specific port forward string.
> 4.  `awk '{print $1}'`: From the final, single line of output, print only the first column, which is the PID.

Here are the variations based on that logic:

#### To Find and Verify the Process
*This command will show you the full line of the matching process so you can confirm it's the correct one. This is the safest first step.*
```shell
pgrep -l -f ssh | grep -- "-o ExitOnForwardFailure=yes" | grep "9103:vpc-whalar"
```

#### To Get ONLY the Process ID (PID)
*This command will output only the number of the Process ID, which is useful for scripting or using with `kill`.*
```shell
pgrep -l -f ssh | grep -- "-o ExitOnForwardFailure=yes" | grep "9103:vpc-whalar" | awk '{print $1}'
```

---
### 🛑 Terminating Processes
---
*How to stop a running process.*

#### `kill `
*Sends a signal to a process to terminate it.*
> By default, `kill` sends the `TERM` (terminate) signal, which asks the process to shut down gracefully.
```shell
# Gracefully stop the process with PID 1234
kill 1234
```

#### `kill -9 ` or `kill -SIGKILL `
*Sends the `KILL` signal.*
> This is the "force quit" command. The process is killed immediately by the kernel and has no chance to clean up. Use this if a graceful `kill` doesn't work.
```shell
# Forcibly kill process 1234
kill -9 1234
```

#### `pkill`
*Kills processes based on their name or other attributes.*
> `pkill` is a shortcut that combines `pgrep` and `kill`. It's very convenient but be careful not to kill the wrong process!

##### `pkill `
*Kills all processes whose name matches the pattern.*
```shell
# Gracefully kill all processes named 'slack'
pkill slack
```

##### `pkill -f `
*Kills all processes whose **full command line** matches the pattern.*
> 🌟 This is very useful but potentially dangerous. Always do a `pgrep -f` first to see what you *would* kill.
```shell
# Forcibly kill the specific python script
pkill -9 -f "my_long_running_script.py"
```

#### To Kill the Process Directly (Using Your Chain)
*This command finds the precise PID and immediately sends it to the `kill` command. Use this once you are confident the filters are correct.*
```shell
# The $() syntax executes the command inside it and uses its output as an argument
kill $(pgrep -l -f ssh | grep -- "-o ExitOnForwardFailure=yes" | grep "9103:vpc-whalar" | awk '{print $1}')
```