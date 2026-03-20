# 08 - Process Management

**Difficulty:** Intermediate  
**Time Estimate:** 75–100 minutes  
**Prerequisites:** Lessons 01–07 (especially Terminal Basics and User Management)

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain what a process is and identify its key attributes (PID, PPID, UID)
- Understand all Linux process states and their transitions
- Use `ps`, `top`, `htop`, `pgrep`, and `pstree` to inspect running processes
- Control foreground and background jobs with `&`, `fg`, `bg`, and `jobs`
- Send signals to processes using `kill`, `killall`, and `pkill`
- Adjust process priority using `nice` and `renice`
- Navigate the `/proc` virtual filesystem
- Schedule recurring tasks with `cron` and one-time tasks with `at`

---

## 1. What Is a Process?

A **process** is a running instance of a program. When you execute a command, the kernel creates a process with:

| Attribute | Description                                          |
|-----------|------------------------------------------------------|
| **PID**   | Process ID — unique identifier for this process      |
| **PPID**  | Parent Process ID — the process that spawned this one|
| **UID**   | User ID of the process owner                         |
| **GID**   | Group ID of the process                              |
| **TTY**   | Terminal attached to the process (or `?` if none)    |
| **CMD**   | The command that started the process                 |

Every process except PID 1 (the init/systemd process) has a parent. This creates a **process tree**.

```bash
# See your shell's PID
echo $$

# See the parent PID
echo $PPID

# See PID of a command you just ran
sleep 60 &
echo $!    # PID of the last background process
```

---

## 2. Process States

```
                  ┌─────────────────────────────────────────────┐
                  │           PROCESS STATE DIAGRAM              │
                  └─────────────────────────────────────────────┘

    fork()                                              exit()
  ─────────►  CREATED ──────► RUNNING ◄──────────────► TERMINATED
                                  │    ◄── CPU given back ─┐
                                  │                         │
                    wait for I/O  │    I/O complete /        │
                    or event      ▼    event arrives         │
                             SLEEPING ─────────────────────►┘
                                  │
                    SIGSTOP /      │   SIGCONT
                    Ctrl+Z         ▼
                             STOPPED ──────────────────────►┘
                                  
                         ┌──────────────┐
                         │    ZOMBIE    │  (process exited, parent
                         │   (defunct)  │   hasn't called wait())
                         └──────────────┘
```

| State | Symbol | Description                                                      |
|-------|--------|------------------------------------------------------------------|
| Running / Runnable | R | Actively using CPU or ready and waiting for CPU  |
| Interruptible Sleep | S | Waiting for I/O, signal, or event (can be interrupted)|
| Uninterruptible Sleep | D | Waiting for I/O that cannot be interrupted (disk I/O)|
| Stopped | T | Suspended by SIGSTOP or Ctrl+Z                           |
| Zombie | Z | Exited but parent hasn't called `wait()` yet              |

---

## 3. Viewing Processes

### ps — Process Snapshot

```bash
# Show processes for current terminal
ps

# Show all processes with full details (BSD style)
ps aux

# Show all processes, full format (UNIX style)
ps -ef

# Show process tree (visual hierarchy)
ps -ef --forest

# Show specific columns
ps -eo pid,ppid,uid,user,%cpu,%mem,stat,cmd

# Show only processes owned by alice
ps -u alice

# Show a specific process by PID
ps -p 1234

# Sort by CPU usage (descending)
ps aux --sort=-%cpu | head -20

# Sort by memory usage (descending)
ps aux --sort=-%mem | head -20

# Show threads
ps -eLf
```

### top — Interactive Real-Time View

```bash
# Launch top
top

# Key bindings inside top:
# q        = quit
# k        = kill a process (prompts for PID)
# r        = renice a process
# M        = sort by memory usage
# P        = sort by CPU usage
# T        = sort by cumulative time
# u        = filter by user
# f        = field management (add/remove columns)
# 1        = toggle per-CPU display
# d or s   = change update interval (seconds)
# H        = toggle thread view
# W        = write current settings to ~/.toprc

# Run top non-interactively (5 iterations, 2s interval)
top -b -n 5 -d 2

# Show only a specific user's processes
top -u alice
```

### top Header Explained

```
top - 14:23:01 up 3 days, 6:45,  2 users,  load average: 0.52, 0.48, 0.45
Tasks: 185 total,   1 running, 184 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  0.8 sy,  0.0 ni, 95.5 id,  0.3 wa,  0.0 hi,  0.2 si
MiB Mem :  15987.4 total,   8023.1 free,   5102.4 used,   2861.9 buff/cache
MiB Swap:   2048.0 total,   1987.2 free,     60.8 used.  10407.8 avail Mem

│           │              │            └── load average over 1, 5, 15 min
│           │              └─── uptime
│           └── current time
└── "top" command name
```

**Load average:** The average number of processes waiting for or using the CPU over 1, 5, and 15 minutes. A value equal to your CPU count means 100% utilization; above that means overloaded.

### htop — Enhanced Interactive View

```bash
# Install htop
sudo apt install htop

# Launch htop
htop

# Key bindings inside htop:
# F2 / S   = Setup (configure display)
# F3 / /   = Search by process name
# F4       = Filter (show only matching processes)
# F5 / t   = Toggle tree view
# F6       = Sort column selection
# F9 / k   = Kill process
# F10 / q  = Quit
# u        = Filter by user
# Space    = Tag a process (for bulk operations)
# I        = Invert sort order
```

### pgrep and pstree

```bash
# Find PIDs by name
pgrep firefox
pgrep -u alice bash   # bash processes owned by alice

# Find PIDs and show process name
pgrep -a nginx

# Find PID of the oldest matching process
pgrep -o sshd

# Find PID of the newest matching process
pgrep -n python

# Count matching processes
pgrep -c apache2

# Show process tree (text art)
pstree

# Show process tree with PIDs
pstree -p

# Show process tree for a specific user
pstree -u alice

# Show process tree starting from a PID
pstree 1234
```

---

## 4. Foreground and Background Jobs

### Job Control

```bash
# Run a command in the background with &
sleep 300 &           # Returns: [1] 12345 (job number, PID)

# List background jobs
jobs                  # [1]+ Running    sleep 300 &
jobs -l               # Include PIDs

# Bring job to foreground (by job number)
fg %1
fg %sleep             # Can also use process name

# Suspend foreground process (send SIGSTOP)
# Press Ctrl+Z while process is running

# Send suspended process to background
bg %1

# Run and immediately disown (survives terminal close)
sleep 300 &
disown %1
disown -a   # disown all jobs

# nohup: immune to SIGHUP (terminal hangup)
nohup long_running_script.sh &
nohup long_running_script.sh > output.log 2>&1 &
```

### Job Number vs PID

```bash
# Job number (shell-level, use %)
fg %1
kill %1

# PID (system-level, no %)
kill 12345

# Current job: %% or %+
# Previous job: %-
fg %%
```

---

## 5. Signals and Killing Processes

### Common Signals

| Signal   | Number | Default Action | Description                            |
|----------|--------|----------------|----------------------------------------|
| SIGHUP   | 1      | Terminate      | Hangup (terminal closed / reload config)|
| SIGINT   | 2      | Terminate      | Interrupt from keyboard (Ctrl+C)       |
| SIGQUIT  | 3      | Core dump      | Quit from keyboard (Ctrl+\\)           |
| SIGKILL  | 9      | Terminate      | Cannot be caught or ignored — instant kill|
| SIGTERM  | 15     | Terminate      | Graceful termination (default for kill)|
| SIGSTOP  | 19     | Stop           | Cannot be caught or ignored — pause    |
| SIGCONT  | 18     | Continue       | Resume a stopped process               |
| SIGUSR1  | 10     | Terminate      | User-defined signal 1                  |
| SIGUSR2  | 12     | Terminate      | User-defined signal 2                  |
| SIGCHLD  | 17     | Ignore         | Child process stopped or terminated    |

### kill, killall, pkill

```bash
# Send SIGTERM (graceful) to a PID
kill 12345
kill -15 12345
kill -SIGTERM 12345

# Force kill (cannot be ignored)
kill -9 12345
kill -SIGKILL 12345

# Send SIGHUP (often triggers config reload)
kill -1 12345
kill -HUP 12345

# Kill all processes named "firefox"
killall firefox
killall -9 firefox

# Kill by name pattern (like pgrep but sends signal)
pkill python
pkill -9 -u alice python   # Kill alice's python processes
pkill -STOP nginx           # Pause all nginx processes
pkill -CONT nginx           # Resume all nginx processes

# Kill all processes in a process group
kill -- -12345    # Negative PID = process group

# List all available signals
kill -l
```

**Best practice:** Always try `SIGTERM` first, wait a moment, then use `SIGKILL` only if necessary. `SIGKILL` gives the process no chance to clean up (close files, release locks, etc.).

---

## 6. Process Priority

Linux schedules processes using a priority system. The **nice value** ranges from **-20** (highest priority) to **+19** (lowest priority). The default is **0**.

```
Nice value scale:
-20 ──────────────────────── 0 ──────────────────────── +19
 │                            │                            │
Highest priority         Default               Lowest priority
(needs root to set)                         (nice to others)
```

```bash
# Start a command with a specific nice value
nice -n 10 ./cpu_intensive_task.sh
nice -n -5 ./important_task.sh   # Requires root for negative values
sudo nice -n -10 ./critical.sh

# Change priority of a running process (renice)
renice -n 10 -p 12345             # By PID
renice -n 5 -u alice              # All of alice's processes
renice -n -5 -g developers        # All processes in group

# View nice values with ps
ps -eo pid,ni,comm | head -20

# In top: press 'r' to renice a process interactively
```

---

## 7. The /proc Filesystem

`/proc` is a **virtual filesystem** — it has no files on disk. The kernel exposes process and system information as readable files.

```bash
# Each PID has its own directory
ls /proc/1/           # init/systemd process

# Key files per process:
cat /proc/1/status    # Human-readable process info
cat /proc/1/cmdline   # Command line (null-separated)
cat /proc/1/environ   # Environment variables
cat /proc/1/maps      # Memory map
ls -la /proc/1/fd/    # Open file descriptors
cat /proc/1/io        # I/O statistics
cat /proc/1/net/tcp   # TCP connections

# System-wide info:
cat /proc/cpuinfo     # CPU details
cat /proc/meminfo     # Memory usage
cat /proc/uptime      # Uptime in seconds
cat /proc/loadavg     # Load averages
cat /proc/version     # Kernel version
cat /proc/mounts      # Mounted filesystems
cat /proc/net/dev     # Network interface statistics
cat /proc/sys/        # Kernel parameters (sysctl)

# Your own process
cat /proc/self/status
```

```bash
# Practical: find what files a process has open
ls -la /proc/$(pgrep nginx)/fd

# Practical: read command line of a process
tr '\0' ' ' < /proc/$(pgrep sshd)/cmdline
echo

# Practical: check kernel parameters
cat /proc/sys/net/ipv4/ip_forward
cat /proc/sys/vm/swappiness
```

---

## 8. Job Scheduling with cron

### cron Expression Syntax

```
┌───────── minute (0–59)
│ ┌─────── hour (0–23)
│ │ ┌───── day of month (1–31)
│ │ │ ┌─── month (1–12 or Jan–Dec)
│ │ │ │ ┌─ day of week (0–7, 0 and 7 = Sunday, or Mon–Sun)
│ │ │ │ │
* * * * *  command to execute
```

**Special characters:**
- `*` — any value
- `,` — list (e.g., `1,3,5`)
- `-` — range (e.g., `1-5`)
- `/` — step (e.g., `*/5` = every 5)

```bash
# Edit your personal crontab
crontab -e

# View your crontab
crontab -l

# Remove your crontab
crontab -r

# Edit another user's crontab (root only)
sudo crontab -u alice -e
```

### Cron Expression Examples

```bash
# Every minute
* * * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/script.sh

# At 2:30 AM every day
30 2 * * * /path/to/backup.sh

# At midnight on the 1st of every month
0 0 1 * * /path/to/monthly_report.sh

# At 9 AM every weekday (Mon–Fri)
0 9 * * 1-5 /path/to/morning_task.sh

# Every Sunday at 3 AM
0 3 * * 0 /path/to/weekly_cleanup.sh

# Every 15 minutes between 9 AM and 5 PM on weekdays
*/15 9-17 * * 1-5 /path/to/check.sh

# On January 1st at midnight
0 0 1 1 * /path/to/new_year.sh

# Every 6 hours
0 */6 * * * /path/to/sync.sh
```

### Cron Special Strings

```bash
@reboot     /path/to/startup_script.sh
@yearly     /path/to/annual_task.sh     # same as: 0 0 1 1 *
@monthly    /path/to/monthly_task.sh    # same as: 0 0 1 * *
@weekly     /path/to/weekly_task.sh     # same as: 0 0 * * 0
@daily      /path/to/daily_task.sh      # same as: 0 0 * * *
@hourly     /path/to/hourly_task.sh     # same as: 0 * * * *
```

### System Cron Directories

```bash
# Drop scripts directly into these directories:
ls /etc/cron.hourly/
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

# System-wide crontab (has an extra 'user' field)
cat /etc/crontab

# Additional system crontabs
ls /etc/cron.d/
```

---

## 9. One-Time Scheduling with at and batch

```bash
# Install at
sudo apt install at

# Schedule a command for a specific time
at 14:30
at> /path/to/script.sh
at> Ctrl+D     # Press Ctrl+D to submit

# Schedule using natural language
echo "/path/to/backup.sh" | at midnight
echo "/path/to/report.sh" | at "next monday 9am"
echo "/path/to/alert.sh"  | at "now + 2 hours"
echo "/path/to/task.sh"   | at "tomorrow 8:00"

# List pending at jobs
atq
at -l

# View a specific job's commands
at -c 3      # Show job number 3

# Remove a pending job
atrm 3
at -d 3

# batch: run when system load drops below 0.8
echo "/path/to/heavy_task.sh" | batch
```

---

## Common Mistakes

1. **Using `kill -9` immediately** — Always try `kill -15` first and give the process a few seconds to clean up before escalating to `-9`.
2. **Confusing job numbers and PIDs** — Job numbers use `%` prefix (`fg %1`); PIDs do not (`kill 1234`). Mixing them up sends signals to the wrong process.
3. **Cron jobs silently failing** — Cron runs with a minimal environment (`PATH`, `HOME`, etc. differ from your login shell). Always use absolute paths in cron scripts and redirect stderr: `command >> /var/log/myjob.log 2>&1`.
4. **Forgetting `&` when expecting background execution** — Without `&`, the terminal blocks until the command finishes.
5. **`killall` killing unintended processes** — On some systems, `killall firefox` kills *any* process with "firefox" in its name. Prefer `pkill -x firefox` for exact-name matching.
6. **Zombie processes** — A zombie isn't a problem by itself (it uses almost no resources). The real issue is the parent not reaping it. Fix or restart the parent process.
7. **Not redirecting cron output** — By default, cron emails output to the system user. Always redirect: `* * * * * command > /dev/null 2>&1`.

---

## Pro Tips

- **`watch` command** — Run any command repeatedly: `watch -n 2 'ps aux | grep nginx'` updates every 2 seconds.
- **`lsof -p PID`** — List all open files (including sockets) for a process. Invaluable for debugging.
- **`strace -p PID`** — Attach to a running process and see every system call in real time.
- **`/proc/PID/fd/`** — Even without `lsof`, you can see open file descriptors directly.
- **Process substitution in scripts**: Check if a process is running: `if pgrep -x nginx > /dev/null; then echo "nginx is running"; fi`
- **`systemd-cgtop`** — Like top but organized by cgroup/service (great for systemd systems).
- **`ionice`** — Like `nice` but for I/O scheduling: `ionice -c 3 -p 12345` (idle I/O class).
- **`timeout`** — Automatically kill a command after N seconds: `timeout 30s ./long_script.sh`.

---

## Practice Exercises

**Exercise 1 — Process Inspection**  
Run `ps aux --sort=-%cpu | head -15` and identify the top 5 CPU-consuming processes. Note their PIDs, owners, and state codes.

**Exercise 2 — Job Control**  
Run `sleep 200` in the terminal. Suspend it with Ctrl+Z. Send it to the background with `bg`. Bring it back to the foreground. Finally kill it. Verify it's gone with `jobs`.

**Exercise 3 — Process Priority**  
Start `sha256sum /dev/urandom` in the background with a nice value of 15. Check its nice value with `ps -o pid,ni,comm -p <PID>`. Renice it to 5. Verify the change.

**Exercise 4 — Signal Handling**  
Start `sleep 500 &`. Use three different methods to kill it: first by PID with `kill`, then start a new one and use `killall`, then start another and use `pkill`. Confirm each terminates.

**Exercise 5 — /proc Exploration**  
Find the PID of your bash shell with `echo $$`. Explore `/proc/$$`: read `status`, decode `cmdline` (replace null bytes), and list the open file descriptors in `fd/`.

**Exercise 6 — Cron Setup**  
Write a cron job that appends a timestamp to `/tmp/heartbeat.log` every minute. Wait 3 minutes. Check the log. Then remove the cron job.

**Exercise 7 — at Scheduling**  
Schedule a command to create a file `/tmp/at_test.txt` with the content "at works!" in 2 minutes. Use `atq` to verify the job is queued. After it runs, confirm the file exists.

**Exercise 8 — Load Average Interpretation**  
Run `uptime` and note the load averages. Run `cat /proc/loadavg`. Look up how many CPU cores your system has (`nproc`). Calculate whether the system is under, at, or over its capacity.

---

## Key Takeaways

- Every process has a **PID**, **PPID**, and **UID**; processes form a tree rooted at PID 1
- Process states: **R** (running), **S** (sleeping), **D** (uninterruptible), **T** (stopped), **Z** (zombie)
- **`ps aux`** for a snapshot; **`top`/`htop`** for real-time monitoring; **`pgrep`** for finding PIDs by name
- Job control: `&` (background), Ctrl+Z (suspend), `fg`/`bg` (foreground/background), `jobs` (list)
- Always try **`SIGTERM` (15)** before **`SIGKILL` (9)** — give processes a chance to clean up
- **nice** (-20 to +19): negative = higher priority (needs root); positive = lower priority
- **cron** for recurring tasks; **at** for one-time future tasks
- `/proc` is a virtual filesystem exposing live kernel and process data — no disk I/O involved

---

## Next Lesson Preview

**Lesson 09 — Package Management**  
Now that you can manage processes, it's time to install software like a pro. We'll cover `apt`, `dpkg`, PPAs, Snap, Flatpak, and how to compile software from source. You'll never struggle with installing or removing packages again.
