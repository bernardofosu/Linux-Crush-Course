# 📝 Linux Process Management & Lifecycle

## 🔹 Process States

-   **R (Running)** → Actively running on CPU or ready to run.\
-   **Sleeping** → Waiting for an event (e.g., I/O).\
-   **Waiting** → Longer-term wait, typically for specific resources.\
-   **Stopped** → Suspended, usually with `Ctrl+Z` or `kill -STOP`.\
-   **Zombie** → Finished execution, but parent process hasn't collected
    its exit status.

Check states with:

``` bash
ps -ef
ps aux
```

------------------------------------------------------------------------

## 🔹 Process Lifecycle

-   **Created** → When a program is launched (fork/exec).\
-   **Ready/Running** → Process is scheduled on CPU.\
-   **Waiting/Sleeping** → Process waits for input/output.\
-   **Stopped** → Suspended manually.\
-   **Terminated/Zombie** → Process finished (cleaned by parent).

------------------------------------------------------------------------

## 🔹 Commands to View Processes

-   `ps` → snapshot of processes.\
-   `ps -ef` → full format with UID, PID, PPID, CMD.\
-   `top` → real-time process monitor (CPU, MEM, PID).\
-   `htop` → interactive version of top (scroll, sort, kill).

------------------------------------------------------------------------

## 🔹 Foreground vs Background

**Foreground** → runs in terminal, blocks shell until finished.\
Example:

``` bash
sleep 30
```

**Background** → append `&` to run without blocking shell.\
Example:

``` bash
sleep 30 &
```

Check jobs:

``` bash
jobs
```

Bring background job to foreground:

``` bash
fg %1
```

Send job to background again:

``` bash
bg %1
```

------------------------------------------------------------------------

## 🔹 Signals & Killing Processes

-   `Ctrl + C` → send **SIGINT** (terminate immediately).\
-   `Ctrl + Z` → send **SIGTSTP** (stop/suspend).

Kill manually:

``` bash
kill -15 <PID>   # Gracefully terminate (SIGTERM)
kill -9 <PID>    # Force kill (SIGKILL)
```

View jobs and PIDs:

``` bash
ps -ef | grep <process>
jobs
```

------------------------------------------------------------------------

## 🔹 Job Number vs PID

When you background tasks:

    [1] 30890

-   `[1]` → Job number (shell's tracking).\
-   `30890` → PID (system-wide unique process ID).

------------------------------------------------------------------------

## 🔹 Priorities & Niceness

-   **Priority** = how soon the scheduler picks a process.\
-   **Niceness (nice value)** = user-assigned "politeness" (`–20` =
    highest priority, `19` = lowest).

Check nice value:

``` bash
ps -o pid,comm,nice -p <PID>
```

Run a process with custom nice value:

``` bash
nice -n 10 sleep 200 &
```

Change niceness of a running process:

``` bash
renice -n 5 -p <PID>
```

------------------------------------------------------------------------

## 🔹 Real-Time Monitoring

-   `top` → dynamic, shows PID, CPU%, MEM%.\
-   `htop` → user-friendly with colors and navigation.\
-   `watch -n 1 ps -ef` → refresh ps output every second.

------------------------------------------------------------------------

## ✅ Summary

-   Process states: Running, Sleeping, Waiting, Stopped, Zombie.\
-   Foreground vs Background → `fg`, `bg`, `jobs`.\
-   Signals: `Ctrl+C` (terminate), `Ctrl+Z` (stop).\
-   Kill: `kill -15` (graceful), `kill -9` (force).\
-   Priorities → controlled by `nice` and `renice`.\
-   Monitoring → `ps`, `top`, `htop`.

------------------------------------------------------------------------

# 📝 Foreground & Background Process Control

## 🔹 Run a Process in Foreground

By default, when you start a command, it runs in the **foreground**
(ties up the terminal).

``` bash
sleep 30
```

This will block your shell for 30 seconds until it finishes.

------------------------------------------------------------------------

## 🔹 Run a Process in Background

Add an ampersand `&` at the end:

``` bash
sleep 30 &
```

-   Runs in the background.\
-   Terminal is free for other commands.\
-   Shell shows:\

```{=html}
<!-- -->
```
    [1] 30890

-   `[1]` → job number.\
-   `30890` → PID.

------------------------------------------------------------------------

## 🔹 Check Background Jobs

``` bash
jobs
```

Output example:

    [1]  Running   sleep 200 &
    [2]  Running   sleep 500 &

------------------------------------------------------------------------

## 🔹 Bring a Job to Foreground

``` bash
fg %1
```

Brings job 1 back to foreground.\
If only one job → just `fg`.

------------------------------------------------------------------------

## 🔹 Send a Job Back to Background

If a process is in foreground, press:\
`Ctrl + Z` → suspends it (**stopped state**).

Then run:

``` bash
bg %1
```

➡️ Resumes job 1 in background.

------------------------------------------------------------------------

## 🔹 Switch Between Jobs

-   Start multiple jobs with `&`.\
-   Use `jobs` to list them.\
-   Bring one forward with `fg %<job_number>`.\
-   Move it back with `Ctrl+Z + bg %<job_number>`.

------------------------------------------------------------------------

## ✅ In short:

-   **Background** → run with `&` or use `bg`.\
-   **Foreground** → use `fg`.\
-   **Suspend** → `Ctrl+Z`.\
-   **List jobs** → `jobs`.
