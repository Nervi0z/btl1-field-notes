# Linux Forensic Artifacts

Linux forensics relies heavily on understanding the standard filesystem hierarchy and log locations. The artifacts are less centralized than Windows — evidence is scattered across log files, hidden directories, and the /proc virtual filesystem — but the patterns are consistent across most distributions.

---

## /var/log — system logs

| File | What it records |
| :--- | :--- |
| `syslog` / `messages` | General system events from the kernel and daemons |
| `auth.log` (Debian/Ubuntu) / `secure` (RHEL/CentOS) | Authentication events — SSH logins, sudo, su, PAM |
| `kern.log` | Kernel messages — hardware, drivers, kernel errors |
| `boot.log` | Events from system boot |
| `wtmp` | Login/logout history — binary, read with `last` |
| `btmp` | Failed login attempts — binary, read with `lastb` |
| `lastlog` | Most recent login per user — binary, read with `lastlog` |
| `apt/history.log` | Package installs/removals (Debian/Ubuntu) |
| `yum.log` / `dnf.log` | Package history (RHEL/CentOS/Fedora) |
| `apache2/access.log` | Apache HTTP access log |
| `nginx/access.log` | Nginx HTTP access log |

```bash
# read login history
last -F          # includes timestamps
lastb -F         # failed logins
lastlog          # most recent login per account

# read auth logs
grep -i 'failed\|invalid\|accepted' /var/log/auth.log
grep 'sshd' /var/log/auth.log | grep 'Accepted'

# journalctl (systemd-based systems)
journalctl -u ssh --since "2024-01-01" --until "2024-01-02"
journalctl -u sshd -n 50 --no-pager
```

---

## Bash history

```bash
# current user history
cat ~/.bash_history
history

# all users' bash history
for user in $(cut -d: -f1 /etc/passwd); do
    home=$(eval echo ~$user)
    if [ -f "$home/.bash_history" ]; then
        echo "=== $user ==="
        cat "$home/.bash_history"
    fi
done

# check for timestamps (if HISTTIMEFORMAT was set)
cat ~/.bash_history | grep -B1 '#[0-9]\{10\}'

# other shell histories
cat ~/.zsh_history
cat ~/.local/share/fish/fish_history

# root history
sudo cat /root/.bash_history
```

> Bash history can be deleted or manipulated by a user. Its absence is a finding — document it. Check for `history -c` or truncated files.

---

## User accounts

```bash
# all user accounts — username:password:UID:GID:comment:home:shell
cat /etc/passwd

# accounts with login shells (not service accounts)
grep -v '/nologin\|/false' /etc/passwd

# accounts with UID 0 (root-equivalent)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# recently modified passwd file (sign of new account creation)
ls -la /etc/passwd

# group memberships
cat /etc/group
id <username>

# sudo privileges
cat /etc/sudoers
ls /etc/sudoers.d/
```

---

## Scheduled tasks (cron)

```bash
# system-wide crontab
cat /etc/crontab

# cron directories (scripts dropped here run at scheduled intervals)
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# per-user crontabs
crontab -l -u <username>

# loop through all user crontabs
for user in $(cut -d: -f1 /etc/passwd); do
    echo "=== $user ==="; crontab -l -u $user 2>/dev/null; done

# crontab files location (requires root)
ls -la /var/spool/cron/crontabs/
```

---

## Network state

```bash
# all connections with process info
ss -tulnp
ss -antp

# established connections
ss -tp state established

# listening ports
ss -tlnp

# ARP cache (recently connected hosts)
ip neigh
arp -a

# network interfaces and IPs
ip addr
ifconfig -a

# routing table
ip route
```

---

## Suspicious locations and recent changes

```bash
# files in /tmp and /var/tmp (common malware staging areas)
ls -la /tmp/
ls -la /var/tmp/
find /tmp /var/tmp -type f -newer /etc/passwd

# recently modified files system-wide (last 24 hours)
find / -type f -mtime -1 -not -path '/proc/*' -not -path '/sys/*' 2>/dev/null

# recently modified files (last 7 days)
find / -type f -mtime -7 -not -path '/proc/*' -not -path '/sys/*' 2>/dev/null

# SUID binaries (can run as root regardless of who executes them)
find / -perm -4000 -type f 2>/dev/null

# SGID binaries
find / -perm -2000 -type f 2>/dev/null

# world-writable files (unusual outside /tmp)
find / -perm -o+w -type f -not -path '/proc/*' 2>/dev/null

# executables in /tmp or /dev/shm (common malware drop locations)
find /tmp /dev/shm -executable -type f 2>/dev/null
```

---

## /proc filesystem

Virtual filesystem — exists only in memory, not on disk. Contains per-process information for every running process. Useful on a live system; not available in a disk image (unless specifically captured).

```bash
# list all running process IDs
ls /proc/ | grep '^[0-9]'

# command line of a specific process
cat /proc/<PID>/cmdline | tr '\0' ' '; echo

# environment variables of a process
cat /proc/<PID>/environ | tr '\0' '\n'

# files open by a process
ls -la /proc/<PID>/fd/

# memory maps — what libraries a process loaded
cat /proc/<PID>/maps

# executable path (useful when binary has been deleted)
ls -la /proc/<PID>/exe
```

---

## Loaded kernel modules

```bash
# list all loaded modules
lsmod

# get details on a specific module
modinfo <module_name>

# check for unsigned or unusual modules
lsmod | grep -v '^Module'

# kernel module files on disk
find /lib/modules/ -name '*.ko' -newer /etc/passwd 2>/dev/null
```

---

## Systemd services

```bash
# all active services
systemctl list-units --type=service --state=active

# all failed services
systemctl list-units --type=service --state=failed

# all service unit files (including disabled)
systemctl list-unit-files --type=service

# examine a specific service
systemctl status <service_name>
cat /etc/systemd/system/<service_name>.service

# services enabled at boot
systemctl list-unit-files --type=service --state=enabled
```
