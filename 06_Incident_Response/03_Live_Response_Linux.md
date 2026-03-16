# Live Response — Linux

Full command reference for Linux triage. Collect volatile data before any containment actions — network isolation and process termination destroy evidence. Run as root or with sudo.

---

## Running processes

```bash
# full process list with user, CPU, memory
ps aux --sort=-%cpu | head 30
ps aux --sort=-%mem | head 30

# process tree — shows parent-child relationships
pstree -aup

# process list with full command lines
ps -eo pid,ppid,user,cmd --sort=pid | less

# find processes running from unusual locations
ps aux | grep -E '/tmp/|/dev/shm/|/var/tmp/'

# processes listening on network (quick check)
ps aux | grep -E 'nc|ncat|socat|python.*http|php.*http'

# check a specific PID in detail
cat /proc/<PID>/cmdline | tr '\0' ' '; echo
cat /proc/<PID>/environ | tr '\0' '\n'
ls -la /proc/<PID>/exe
ls -la /proc/<PID>/fd/
```

---

## Network connections

```bash
# all connections with process info
ss -tulnp

# established connections only
ss -tp state established

# all TCP connections
ss -antp

# all UDP
ss -anup

# listening services
ss -tlnp

# netstat equivalent (if ss unavailable)
netstat -tulnp
netstat -antp

# connections with process names (includes process names in output)
ss -tulnp | column -t

# check what process owns a specific port
ss -tlnp | grep ':443'
fuser 443/tcp
```

---

## Listening services

```bash
# listening TCP ports with associated processes
ss -tlnp

# listening UDP ports
ss -ulnp

# lsof — open files and network sockets (full view)
lsof -i
lsof -i TCP -i UDP
lsof -i :4444        ← specific port
lsof -i TCP:80       ← TCP port 80
lsof -p <PID>        ← all open files for a process
```

---

## Recently modified files

```bash
# files modified in the last 24 hours (exclude /proc and /sys)
find / -type f -mtime -1 \
  -not -path '/proc/*' \
  -not -path '/sys/*' \
  -not -path '/run/*' \
  2>/dev/null | head 50

# recently modified files in /tmp (high-value staging area)
find /tmp /var/tmp /dev/shm -type f -newer /etc/passwd 2>/dev/null
ls -lat /tmp/ | head 20
ls -lat /var/tmp/ | head 20

# files modified in last 7 days, sorted by modification time
find / -type f -mtime -7 \
  -not -path '/proc/*' -not -path '/sys/*' \
  2>/dev/null | xargs ls -lt 2>/dev/null | head 30

# executables in /tmp (immediate red flag)
find /tmp /var/tmp /dev/shm -executable -type f 2>/dev/null
```

---

## Cron jobs

```bash
# system crontab
cat /etc/crontab

# cron drop directories
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# contents of each cron directory
for dir in /etc/cron.d /etc/cron.hourly /etc/cron.daily /etc/cron.weekly /etc/cron.monthly; do
  echo "=== $dir ==="; ls -la $dir 2>/dev/null; done

# per-user crontabs — loop through all users
for user in $(cut -d: -f1 /etc/passwd); do
  crontab_content=$(crontab -l -u $user 2>/dev/null)
  if [ -n "$crontab_content" ]; then
    echo "=== $user ==="
    echo "$crontab_content"
  fi
done

# crontab files on disk (requires root)
ls -la /var/spool/cron/crontabs/ 2>/dev/null
cat /var/spool/cron/crontabs/* 2>/dev/null
```

---

## SUID binaries

```bash
# find all SUID binaries (run as file owner, often root)
find / -perm -4000 -type f 2>/dev/null

# find all SUID and SGID
find / -perm /6000 -type f 2>/dev/null

# compare against expected baseline (common legitimate SUID binaries)
find / -perm -4000 -type f 2>/dev/null | \
  grep -vE '/usr/bin/|/usr/lib/|/bin/|/sbin/'
```

---

## Login history

```bash
# successful login history (reads /var/log/wtmp)
last -F | head 30
last -F -n 50       ← last 50 entries

# failed login attempts (reads /var/log/btmp) — requires root
lastb -F | head 30

# most recent login per user
lastlog | grep -v 'Never'

# currently logged in users
who
w

# SSH logon events from auth.log
grep 'Accepted\|Failed\|Invalid' /var/log/auth.log | tail 30
grep 'sshd.*Accepted' /var/log/auth.log | awk '{print $1, $2, $3, $9, $11}'
```

---

## User accounts

```bash
# all accounts with their shells
cat /etc/passwd
grep -v '/nologin\|/false' /etc/passwd

# accounts with UID 0 (root-equivalent — should only be root)
awk -F: '$3 == 0 {print $1, $3, $6}' /etc/passwd

# recently modified /etc/passwd (new accounts created)
stat /etc/passwd
ls -la /etc/passwd /etc/shadow /etc/group

# accounts with empty passwords (check /etc/shadow)
sudo awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow

# sudo privileges
cat /etc/sudoers
ls /etc/sudoers.d/
sudo grep -r 'ALL' /etc/sudoers.d/ 2>/dev/null
```

---

## Bash history

```bash
# current user history
history
cat ~/.bash_history

# root history
sudo cat /root/.bash_history

# all users' bash history
for user in $(cut -d: -f1 /etc/passwd); do
  home=$(eval echo ~$user 2>/dev/null)
  hist="$home/.bash_history"
  if [ -f "$hist" ]; then
    echo "=== $user ($hist) ==="
    cat "$hist"
  fi
done

# check other shell histories
cat ~/.zsh_history 2>/dev/null
cat ~/.local/share/fish/fish_history 2>/dev/null

# check if history was cleared or modified recently
ls -la ~/.bash_history
stat ~/.bash_history
```

---

## Loaded kernel modules

```bash
# list all loaded kernel modules
lsmod

# get details on a suspicious module
modinfo <module_name>

# check for modules not in standard locations
find /lib/modules/ -name '*.ko' -newer /etc/passwd 2>/dev/null

# modules loaded outside of standard lib paths (rootkit indicator)
lsmod | awk '{print $1}' | while read mod; do
  path=$(modinfo -n $mod 2>/dev/null)
  echo "$mod: $path"
done | grep -v '/lib/modules/'
```

---

## Systemd services

```bash
# active services
systemctl list-units --type=service --state=active --no-pager

# failed services
systemctl list-units --type=service --state=failed --no-pager

# all services including inactive
systemctl list-unit-files --type=service --no-pager

# enabled at boot
systemctl list-unit-files --type=service --state=enabled --no-pager

# examine a specific service
systemctl status <service_name>
cat /etc/systemd/system/<service_name>.service
journalctl -u <service_name> -n 50 --no-pager

# recently modified service files (possible malicious service creation)
find /etc/systemd/system/ /lib/systemd/system/ -newer /etc/passwd -name '*.service' 2>/dev/null
```

---

## Open files and sockets — lsof

```bash
# all open files
lsof

# open network connections
lsof -i

# TCP connections only
lsof -i TCP

# open files for a specific process
lsof -p <PID>

# what process has a file open
lsof /var/log/auth.log

# open files for a specific user
lsof -u <username>

# deleted files still open (common malware indicator — file deleted to hide but still running)
lsof | grep '(deleted)'
```

---

## /proc filesystem — per-process forensics

```bash
# executable path (even if the binary was deleted)
ls -la /proc/<PID>/exe

# full command line
cat /proc/<PID>/cmdline | tr '\0' ' '; echo

# environment variables
cat /proc/<PID>/environ | tr '\0' '\n'

# open file descriptors
ls -la /proc/<PID>/fd/

# memory maps — loaded libraries, memory-mapped files
cat /proc/<PID>/maps

# network connections for a specific process
cat /proc/<PID>/net/tcp    ← hex encoded; use ss -p instead for readability

# dump process memory (careful — large output)
# can reveal strings, keys, credentials in memory
strings /proc/<PID>/mem 2>/dev/null | grep -i 'password\|token\|key' | head 20
```
