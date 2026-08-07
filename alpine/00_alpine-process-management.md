# Alpine Linux — Process Management Cheatsheet

Alpine uses **BusyBox** utilities by default (lighter, fewer flags than GNU coreutils) and **OpenRC** as its init/service manager (not systemd). Keep that in mind — some familiar flags won't exist.

## Viewing Processes

```sh
ps                    # BusyBox ps — minimal by default
ps aux                # BSD-style flags (works if 'procps' or full busybox applet supports it)
ps -ef                # full-format listing
ps -o pid,ppid,cmd    # custom columns
```

> Alpine's built-in `ps` is BusyBox's stripped-down version. For a fuller `ps`/`top`/`kill` experience, install the `procps` package:
> ```sh
> apk add procps
> ```

```sh
top                    # live process viewer (BusyBox top; press 'q' to quit)
htop                   # nicer interactive viewer (apk add htop)
pstree                 # process tree (apk add psmisc, or busybox has a basic one)
```

## Finding a Specific Process

```sh
pgrep nginx            # list PIDs matching name
pgrep -a nginx         # PIDs + full command line
ps aux | grep nginx    # classic fallback
```

## Killing / Signaling Processes

```sh
kill <PID>             # SIGTERM (graceful)
kill -9 <PID>          # SIGKILL (force)
kill -HUP <PID>        # reload config (common for daemons)

pkill nginx            # kill by name (SIGTERM)
pkill -9 -f "python app.py"   # kill by full matched command line
killall nginx           # kill all processes by exact name (apk add psmisc if missing)
```

Common signals:
| Signal | Number | Meaning |
|---|---|---|
| SIGHUP  | 1 | Reload / hangup |
| SIGINT  | 2 | Interrupt (Ctrl+C) |
| SIGKILL | 9 | Force kill (can't be caught) |
| SIGTERM | 15 | Graceful terminate (default) |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume paused process |

## Job Control (shell-level)

```sh
command &              # run in background
jobs                    # list background jobs
fg %1                   # bring job 1 to foreground
bg %1                   # resume job 1 in background
Ctrl+Z                  # suspend foreground process
disown %1                # detach job from shell (survives shell exit)
nohup command &          # run immune to SIGHUP, survives terminal close
```

## Process Priority

```sh
nice -n 10 command       # start with lower priority (higher = nicer/lower priority)
renice -n 5 -p <PID>      # change priority of running process
```

## Resource / Limits Info

```sh
cat /proc/<PID>/status   # detailed process info (memory, state, etc.)
cat /proc/<PID>/cmdline  # full command line (null-separated)
ls /proc/<PID>/fd        # open file descriptors
free -m                   # memory usage (busybox applet, or apk add procps)
uptime                    # load averages
```

## Managing Services (OpenRC — Alpine's init system)

Alpine does **not** use systemd/systemctl. Use `rc-service` and `rc-update` instead:

```sh
rc-service nginx start        # start a service
rc-service nginx stop
rc-service nginx restart
rc-service nginx status

rc-update add nginx default    # enable service at boot (runlevel "default")
rc-update del nginx default    # disable at boot
rc-update show                 # list what's enabled per runlevel

rc-status                      # show status of all services in current runlevel
```

## Autostart / Background Daemons

- OpenRC services live in `/etc/init.d/`
- Custom init scripts should follow the OpenRC script format (`depend()`, `start()`, `stop()` functions)
- For simple one-off background daemons without a full service script, `nohup cmd &` or a supervisor like `s6` (common in Alpine-based Docker images) works well

## Alpine/Docker Note

Alpine containers often run as PID 1 without a real init system, which breaks signal handling and zombie reaping. Fixes:

```sh
# Use tini or dumb-init as PID 1
apk add tini
ENTRYPOINT ["/sbin/tini", "--"]
```

Or in `docker run`: `docker run --init ...`

## Quick Reference Table

| Task | Command |
|---|---|
| List processes | `ps aux` / `ps -ef` |
| Live monitor | `top`, `htop` |
| Find PID by name | `pgrep -a <name>` |
| Kill gracefully | `kill <PID>` |
| Force kill | `kill -9 <PID>` |
| Kill by name | `pkill <name>` |
| Background a job | `cmd &` |
| Survive logout | `nohup cmd &` |
| Manage service | `rc-service <name> start/stop/restart/status` |
| Enable at boot | `rc-update add <name> default` |
