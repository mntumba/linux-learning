# 08 — Process Management

> **Module 2 · Lesson 2** | Difficulty: ★★★☆☆ Intermediate | Time: ~75 min

---

## Learning Objectives

- Understand what a process is (PID, PPID, state)
- Monitor processes with ps, top, and htop
- Control processes with kill, signals, and job control
- Manage background and foreground jobs
- Schedule tasks with cron and at
- Understand systemd service management

---

## Table of Contents

1. [What Is a Process?](#1-what-is-a-process)
2. [Viewing Processes](#2-viewing-processes)
3. [Process Signals](#3-process-signals)
4. [Job Control](#4-job-control)
5. [Process Priority (nice/renice)](#5-process-priority-nicerenice)
6. [The /proc Filesystem](#6-the-proc-filesystem)
7. [systemd Service Management](#7-systemd-service-management)
8. [Scheduled Tasks with cron](#8-scheduled-tasks-with-cron)
9. [The `at` Command](#9-the-at-command)
10. [Practice Exercises](#10-practice-exercises)
11. [Key Takeaways](#11-key-takeaways)

---

## 1. What Is a Process?

A **process** is a running instance of a program. Every time you run a command, the kernel creates a new process.

```
Process Attributes:
┌────────────────────────────────────────────┐
│ PID    — Process ID (unique integer)       │
│ PPID   — Parent Process ID                 │
│ UID    — User ID running the process       │
│ GID    — Group ID                          │
│ State  — Running/Sleeping/Stopped/Zombie   │
│ Nice   — Priority (-20 to 19)              │
│ CPU %  — CPU usage                         │
│ MEM %  — Memory usage                      │
│ CMD    — Command name                      │
└────────────────────────────────────────────┘
```

### Process States

| State | Code | Description |
|-------|------|-------------|
| Running | R | Currently executing on CPU |
| Sleeping (interruptible) | S | Waiting for event (most processes) |
| Sleeping (uninterruptible) | D | Waiting for I/O (cannot be interrupted) |
| Stopped | T | Stopped by signal (Ctrl+Z) |
| Zombie | Z | Finished but parent hasn't acknowledged |

### Process Tree

Every process has a parent. The first process (PID 1) is `systemd` (or `init`):

```
PID 1: systemd
├── PID 234: sshd
│   └── PID 891: sshd (user session)
│       └── PID 892: bash
│           └── PID 1024: vim
├── PID 410: cron
├── PID 511: nginx
│   ├── PID 512: nginx (worker)
│   └── PID 513: nginx (worker)
└── PID 614: postgres
```

```bash
pstree              # visualize process tree
pstree -p           # with PIDs
pstree -u           # with usernames
pstree alice        # tree for specific user
```

---

## 2. Viewing Processes

### `ps` — Process Status

```bash
# Basic usage
ps                      # processes in current terminal
ps -e                   # every process
ps -ef                  # every process, full format
ps aux                  # BSD style, all users, details

# ps aux output explained:
$ ps aux | head -5
USER    PID  %CPU %MEM    VSZ   RSS TTY    STAT START   TIME COMMAND
root      1   0.0  0.1 168544 12824 ?       Ss  09:00   0:01 /sbin/init
root      2   0.0  0.0      0     0 ?       S   09:00   0:00 [kthreadd]
alice  1234   0.1  0.5 123456 45678 pts/0   Ss  10:15   0:00 bash
alice  1235   0.0  0.0  18992  3420 pts/0   R+  10:16   0:00 ps aux

# Column meanings:
# USER = owner
# PID = process ID
# %CPU = CPU usage
# %MEM = memory usage
# VSZ = virtual memory size (KB)
# RSS = resident set size (physical RAM, KB)
# TTY = terminal (? = no terminal/daemon)
# STAT = process state
# START = when started
# TIME = CPU time consumed
# COMMAND = command

# Useful ps variations
ps -ef --forest             # with ASCII tree
ps -u alice                 # processes owned by alice
ps -p 1234                  # specific PID
ps -C nginx                 # processes named nginx
ps aux --sort=-%cpu | head  # sorted by CPU usage
ps aux --sort=-%mem | head  # sorted by memory usage
ps -eo pid,ppid,user,stat,comm  # custom columns
```

### `top` — Interactive Process Monitor

```bash
top                     # launch interactive monitor

# top keyboard controls:
# q     — quit
# k     — kill process (enter PID)
# r     — renice (change priority)
# s     — change refresh interval
# f     — select fields to display
# o     — filter processes
# 1     — toggle per-CPU display
# M     — sort by memory
# P     — sort by CPU (default)
# T     — sort by time
# u     — filter by user
# H     — toggle threads
# Z     — colorize display
# h     — help

# top header explained:
top - 10:15:23 up 2 days, 3:45,  2 users,  load average: 0.12, 0.18, 0.22
Tasks: 184 total,   1 running, 183 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 96.8 id,  0.4 wa,  0.0 hi,  0.0 si
MiB Mem :   7841.9 total,   3421.2 free,   2103.5 used,   2317.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5291.4 avail Mem

# CPU states: us=user, sy=system, ni=nice, id=idle, wa=I/O wait
# Load average: last 1, 5, 15 minutes (rough rule: < #CPUs = fine)
```

### `htop` — Enhanced Interactive Monitor

```bash
sudo apt install htop
htop                    # much nicer than top

# htop features:
# - Color-coded CPU/memory bars
# - Mouse support
# - F3/F4 to search/filter
# - F5 for tree view
# - F6 to sort
# - F9 to kill (choose signal)
# - Space to tag processes
```

### Other Monitoring Tools

```bash
# atop — historical process monitor
sudo apt install atop
atop

# glances — comprehensive system overview
sudo apt install glances
glances

# vmstat — virtual memory statistics
vmstat 1 5          # show 5 readings, 1 second apart

# iostat — disk I/O statistics
sudo apt install sysstat
iostat -x 1 3       # extended, 3 readings

# free — memory usage
free -h             # human-readable
free -s 2           # update every 2 seconds
watch -n 1 free -h  # continuously update

# uptime — system uptime and load
uptime
```

---

## 3. Process Signals

Signals are software interrupts sent to processes to notify them of events.

### Common Signals

| Signal | Number | Default Action | Description |
|--------|--------|----------------|-------------|
| SIGHUP | 1 | Terminate | Hangup; often used to reload config |
| SIGINT | 2 | Terminate | Interrupt from keyboard (Ctrl+C) |
| SIGQUIT | 3 | Core dump | Quit from keyboard (Ctrl+\) |
| SIGKILL | 9 | Terminate | Kill immediately — cannot be caught! |
| SIGTERM | 15 | Terminate | Graceful termination (default kill) |
| SIGSTOP | 19 | Stop | Pause process — cannot be caught! |
| SIGCONT | 18 | Continue | Resume stopped process |
| SIGUSR1 | 10 | Terminate | User-defined signal 1 |
| SIGUSR2 | 12 | Terminate | User-defined signal 2 |
| SIGCHLD | 17 | Ignore | Child process stopped/terminated |

### Sending Signals

```bash
# kill — send signal to PID
kill 1234              # SIGTERM (15) to PID 1234
kill -15 1234          # explicitly SIGTERM
kill -9 1234           # SIGKILL (immediate, no cleanup)
kill -TERM 1234        # by name
kill -HUP 1234         # reload config (nginx, etc.)

# killall — kill by process name
killall firefox        # SIGTERM all firefox processes
killall -9 firefox     # SIGKILL all firefox processes

# pkill — kill by pattern
pkill -f "python script.py"   # kill by full command line
pkill -u alice                # kill all of alice's processes
pkill -9 -t pts/1            # kill all processes on terminal pts/1

# Send signal to multiple PIDs
kill -9 1234 5678 9012

# Send to process group
kill -TERM -1234       # negative PID = process group

# Graceful vs Forceful:
kill -15 nginx    # Graceful: nginx finishes requests, saves state
kill -9 nginx     # Forceful: immediate death, no cleanup (last resort!)
```

### Signals and `trap` in Scripts

```bash
#!/bin/bash
# Handle signals in scripts
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/lockfile
    exit 0
}

trap cleanup SIGINT SIGTERM  # call cleanup on Ctrl+C or kill
echo "Running... Press Ctrl+C to stop"
while true; do
    sleep 1
done
```

---

## 4. Job Control

Job control allows you to manage multiple processes in a single terminal.

```bash
# Start a background job
sleep 100 &            # & = run in background
# [1] 12345  ← job number, PID

# List jobs
jobs                   # list background jobs
jobs -l                # with PIDs

# Suspend current foreground process
# Press Ctrl+Z  → [1]+  Stopped    sleep 100

# Resume in background
bg %1                  # resume job 1 in background
bg                     # resume most recent job

# Bring to foreground
fg %1                  # bring job 1 to foreground
fg                     # bring most recent job to foreground

# Disown (detach from terminal)
sleep 100 &
disown %1              # job continues even if terminal closes
disown -a              # disown all jobs

# nohup — run command immune to hangup
nohup long_script.sh &        # output → nohup.out
nohup long_script.sh > /var/log/script.log 2>&1 &
```

### Job States

```
[1]+  Running    sleep 100 &      (currently running in bg)
[2]-  Stopped    vim file.txt     (suspended with Ctrl+Z)
[3]   Done       ls -la           (completed)
```

---

## 5. Process Priority (nice/renice)

### CPU Scheduling Priority

Linux uses **nice values** to determine CPU scheduling priority:

```
-20 ←────────────────────────────────────→ +19
Highest priority              Lowest priority
(needs root to set negative values)

Default nice value = 0
```

```bash
# Start process with specific nice value
nice -n 10 long_compilation &    # lower priority (nice)
nice -n -5 important_task &      # higher priority (requires root)
sudo nice -n -15 critical.sh &   # very high priority

# Change priority of running process
renice 10 1234             # lower priority of PID 1234
renice -5 1234             # higher priority (requires root)
sudo renice -15 1234       # very high priority

# Renice by name
sudo renice 10 -p $(pidof firefox)  # lower Firefox priority

# View priority in ps
ps aux | awk '{print $1,$2,$18,$11}' | head -10
# USER  PID  NI  COMMAND

# View in top:
# NI column = nice value
# PR column = real priority (nice + 20, so 0 nice = 20 PR)
```

---

## 6. The /proc Filesystem

`/proc` exposes kernel and process information as files:

```bash
# Per-process directories
ls /proc/1/                   # init/systemd process
cat /proc/1/status            # process status
cat /proc/1/cmdline | tr '\0' ' '  # command line
cat /proc/1/environ | tr '\0' '\n' # environment variables
ls /proc/1/fd/                # open file descriptors
cat /proc/1/maps              # memory map

# Your current process
cat /proc/$$/status           # $$ = current shell PID

# System information
cat /proc/cpuinfo             # CPU details
cat /proc/meminfo             # memory details
cat /proc/version             # kernel version
cat /proc/uptime              # uptime in seconds
cat /proc/loadavg             # load averages
cat /proc/mounts              # mounted filesystems
cat /proc/partitions          # disk partitions

# Network information
cat /proc/net/tcp             # TCP connections
cat /proc/net/udp             # UDP connections
cat /proc/net/if_inet6        # IPv6 interfaces

# Kernel parameters (tunable!)
cat /proc/sys/net/ipv4/ip_forward  # 0 or 1
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward  # enable IP forwarding
```

---

## 7. systemd Service Management

Modern Linux uses **systemd** as the init system. It manages services, logging, networking, etc.

### systemctl — Service Control

```bash
# Service status
systemctl status nginx         # service status
systemctl is-active nginx      # check if running
systemctl is-enabled nginx     # check if starts on boot

# Start/stop/restart
sudo systemctl start nginx     # start service
sudo systemctl stop nginx      # stop service
sudo systemctl restart nginx   # stop then start
sudo systemctl reload nginx    # reload config without restart

# Enable/disable at boot
sudo systemctl enable nginx    # start on boot
sudo systemctl disable nginx   # don't start on boot
sudo systemctl enable --now nginx  # enable AND start now

# List services
systemctl list-units --type=service         # running services
systemctl list-units --type=service --all   # all services
systemctl list-unit-files --type=service    # all service files

# System control
sudo systemctl poweroff        # shutdown
sudo systemctl reboot          # reboot
sudo systemctl suspend         # suspend
sudo systemctl hibernate       # hibernate
```

### Creating a Custom Service

```bash
# Create service file
sudo tee /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=alice
WorkingDirectory=/home/alice/myapp
ExecStart=/usr/bin/python3 /home/alice/myapp/app.py
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd, enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
sudo systemctl status myapp.service
```

### journalctl — View Logs

```bash
journalctl                          # all logs
journalctl -f                       # follow (tail -f equivalent)
journalctl -u nginx                 # logs for nginx service
journalctl -u nginx -f              # follow nginx logs
journalctl -n 50                    # last 50 lines
journalctl --since "1 hour ago"     # last hour
journalctl --since "2024-01-01" --until "2024-01-02"
journalctl -p err                   # errors only
journalctl -p err..warning          # errors and warnings
journalctl -b                       # since last boot
journalctl -b -1                    # previous boot
```

---

## 8. Scheduled Tasks with cron

**cron** is the classic task scheduler for recurring jobs.

### Cron Syntax

```
┌─────────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌─────────── day of month (1-31)
│ │ │ ┌───────── month (1-12 or Jan-Dec)
│ │ │ │ ┌─────── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * *   command to execute

Special characters:
*   = any value
,   = list of values (1,3,5)
-   = range (1-5)
/   = step values (*/5 = every 5)
```

### Common Cron Examples

```bash
# Edit your crontab
crontab -e

# List your crontab
crontab -l

# Remove your crontab
crontab -r

# Crontab examples:
# Run every minute
* * * * * /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run at 2:30 AM daily
30 2 * * * /usr/bin/backup.sh

# Run at midnight on weekdays
0 0 * * 1-5 /path/to/cleanup.sh

# Run at 9 AM every Monday
0 9 * * 1 /path/to/weekly_report.sh

# Run on the 1st of every month
0 0 1 * * /path/to/monthly.sh

# Run at 8:00 and 20:00 daily
0 8,20 * * * /path/to/script.sh

# Capture output to log
*/10 * * * * /path/to/monitor.sh >> /var/log/monitor.log 2>&1

# Environment variables in crontab
MAILTO=admin@example.com
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
0 6 * * * /path/to/daily.sh
```

### Cron Special Strings

```bash
@reboot   — run once at startup
@yearly   — 0 0 1 1 * (once a year)
@annually — same as @yearly
@monthly  — 0 0 1 * * (once a month)
@weekly   — 0 0 * * 0 (once a week, Sunday)
@daily    — 0 0 * * * (once a day)
@midnight — same as @daily
@hourly   — 0 * * * * (once an hour)

# Example
@reboot /path/to/startup_script.sh
@daily  /usr/bin/logrotate /etc/logrotate.conf
```

### System Cron Directories

```bash
ls /etc/cron.d/         # custom cron files
ls /etc/cron.daily/     # scripts run daily
ls /etc/cron.hourly/    # scripts run hourly
ls /etc/cron.monthly/   # scripts run monthly
ls /etc/cron.weekly/    # scripts run weekly
cat /etc/crontab        # system crontab
```

---

## 9. The `at` Command

`at` schedules a **one-time** task:

```bash
sudo apt install at
sudo systemctl start atd

# Schedule commands
at 2pm            # enter commands, press Ctrl+D when done
at 14:30
at midnight
at now + 2 hours
at now + 30 minutes
at 10:00 tomorrow
at 10:00 2024-12-31

# Example:
$ at now + 5 minutes
at> echo "5 minutes have passed" | mail admin@example.com
at> <Ctrl+D>
job 1 at Mon Jan 15 10:20:00 2024

# List scheduled jobs
atq
at -l             # same as atq

# Remove a scheduled job
atrm 1            # remove job 1
at -d 1           # same
```

---

## 10. Practice Exercises

### Exercise 8.1 — Process Investigation

```bash
# 1. Find the PID of the systemd process
ps -ef | grep systemd | head -1
# or: pidof systemd

# 2. Find all processes owned by root
ps -u root | wc -l

# 3. Find the 5 most CPU-hungry processes
ps aux --sort=-%cpu | head -6

# 4. Find the 5 most memory-hungry processes  
ps aux --sort=-%mem | head -6

# 5. Display a process tree
pstree -p | head -20
```

### Exercise 8.2 — Job Control

```bash
# 1. Start a background job
sleep 300 &

# 2. Start another background job
sleep 400 &

# 3. List all background jobs
jobs -l

# 4. Bring first job to foreground
fg %1

# 5. Suspend it (Ctrl+Z)
# 6. Resume in background
bg %1

# 7. Kill all sleep processes
killall sleep
```

### Exercise 8.3 — Cron Setup

```bash
# Create a log rotation script
cat << 'EOF' > ~/rotate_log.sh
#!/bin/bash
LOG=/tmp/test.log
echo "$(date): Log entry" >> $LOG
# Keep only last 10 lines
tail -10 $LOG > $LOG.tmp && mv $LOG.tmp $LOG
EOF
chmod +x ~/rotate_log.sh

# Schedule it to run every minute
crontab -e
# Add: * * * * * /home/youruser/rotate_log.sh

# After 2 minutes, check
cat /tmp/test.log

# Remove the cron job
crontab -e  # delete the line
```

### Exercise 8.4 — Service Management

```bash
# 1. Check status of SSH service
sudo systemctl status ssh

# 2. Install and start nginx
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl status nginx

# 3. Verify nginx is running
curl -s http://localhost | head -10
ps aux | grep nginx

# 4. Reload nginx config without restart
sudo systemctl reload nginx

# 5. Stop and disable nginx
sudo systemctl stop nginx
sudo systemctl disable nginx
```

---

## 11. Key Takeaways

- A **process** is a running program with a PID, state, owner, and resource usage
- **`ps aux`** shows all processes; **`top`** and **`htop`** give interactive monitoring
- **Signals**: SIGTERM (15) = graceful kill; SIGKILL (9) = immediate kill (last resort)
- **Job control**: `&` (background), `Ctrl+Z` (suspend), `fg`/`bg` (resume)
- **`nice`** sets initial priority (-20 to 19); **`renice`** changes running process priority
- **systemctl** manages services: start/stop/enable/disable/status/restart
- **cron** handles recurring tasks; **at** handles one-time scheduled tasks
- **`journalctl -u service -f`** for real-time service logs

---

## Next Lesson

➡️ [09 — Package Management](09_Package_Management.md)

---

*Module 2 · Lesson 2 of 4 | [Course Index](../INDEX.md)*
