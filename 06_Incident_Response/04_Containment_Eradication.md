# Containment and Eradication

Containment stops the attacker from doing more damage. Eradication removes their presence. Neither happens until you've collected the evidence you need — volatile data is gone the moment you isolate a system or kill a process.

---

## Evidence preservation — do this before containment

Capture volatile data in order of volatility. Once you isolate or shut down, this data is gone.

```bash
# Linux — capture in this order before isolation

# 1. running processes
ps aux > /tmp/evidence/ps_aux.txt
ps -eo pid,ppid,user,cmd > /tmp/evidence/ps_full.txt

# 2. network connections
ss -tulnp > /tmp/evidence/ss_output.txt
netstat -ano > /tmp/evidence/netstat.txt 2>/dev/null

# 3. open files and network sockets
lsof > /tmp/evidence/lsof.txt
lsof -i > /tmp/evidence/lsof_network.txt

# 4. active users
who > /tmp/evidence/who.txt
w > /tmp/evidence/w.txt
last -F > /tmp/evidence/last.txt

# 5. ARP cache and routing
ip neigh > /tmp/evidence/arp.txt
ip route > /tmp/evidence/routes.txt

# 6. loaded modules
lsmod > /tmp/evidence/lsmod.txt

# then — if memory dump is needed (do this before any process termination)
sudo insmod lime-<kernel_version>.ko "path=/tmp/evidence/memory.lime format=lime"
```

```powershell
# Windows — capture before isolation

# processes
Get-Process | Select-Object Name, Id, Path, StartTime |
  Export-Csv C:\evidence\processes.csv -NoTypeInformation

# network connections
Get-NetTCPConnection | Select-Object * |
  Export-Csv C:\evidence\tcp_connections.csv -NoTypeInformation
netstat -ano > C:\evidence\netstat.txt

# scheduled tasks
Get-ScheduledTask | Select-Object * |
  Export-Csv C:\evidence\scheduled_tasks.csv -NoTypeInformation
```

---

## Network isolation

**Windows — firewall-based isolation:**

```powershell
# block all inbound and outbound connections except local
# use when you need to keep the system running but cut network access

# block all outbound traffic
netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound

# allow only specific management IP (your analysis station)
netsh advfirewall firewall add rule name="Allow Management" `
  protocol=TCP dir=in remoteip=<your_analysis_ip> action=allow

# undo isolation
netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound
```

```powershell
# disable network adapter (harder-stop isolation)
Disable-NetAdapter -Name "Ethernet" -Confirm:$false

# re-enable
Enable-NetAdapter -Name "Ethernet"
```

**Linux — iptables isolation:**

```bash
# drop all traffic except from management host
iptables -I INPUT -s <your_analysis_ip> -j ACCEPT
iptables -I OUTPUT -d <your_analysis_ip> -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# verify rules applied
iptables -L -n

# flush all rules (undo isolation)
iptables -F
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
```

---

## Account containment

**Windows:**

```powershell
# disable a local user account
Disable-LocalUser -Name "compromised_user"

# disable an Active Directory account (requires RSAT)
Disable-ADAccount -Identity "compromised_user"

# force logoff of active sessions (get session ID from quser)
quser
logoff <session_id>

# reset password (forcing attacker out of any active sessions)
Set-LocalUser -Name "compromised_user" -Password (ConvertTo-SecureString "NewSecureP@ss!" -AsPlainText -Force)

# remove from Administrators group
Remove-LocalGroupMember -Group "Administrators" -Member "compromised_user"
```

**Linux:**

```bash
# lock an account (prepends ! to password hash in /etc/shadow)
passwd -l <username>
usermod -L <username>

# expire account immediately (forces password change on next login)
chage -E 0 <username>

# kill all processes for a user
pkill -u <username>
killall -u <username>

# remove from sudo group
gpasswd -d <username> sudo
gpasswd -d <username> wheel

# check active sessions and kill them
who | grep <username>
pkill -KILL -u <username>
```

---

## Persistence removal

### Registry Run keys (Windows)

```powershell
# list current Run key values
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# remove a malicious Run key value
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "MaliciousEntry"
Remove-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "MaliciousEntry"
```

### Scheduled tasks (Windows)

```powershell
# list suspicious tasks
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} |
  Select-Object TaskName, TaskPath

# disable a task (preserve for evidence, don't delete yet)
Disable-ScheduledTask -TaskName "MaliciousTask"

# delete a task after evidence is collected
Unregister-ScheduledTask -TaskName "MaliciousTask" -Confirm:$false
```

### Malicious services (Windows)

```powershell
# stop a service
Stop-Service -Name "MaliciousService" -Force

# disable it (prevents restart)
Set-Service -Name "MaliciousService" -StartupType Disabled

# delete the service (after stopping)
sc.exe delete "MaliciousService"

# verify the associated file before deleting
Get-WmiObject Win32_Service | Where-Object {$_.Name -eq "MaliciousService"} |
  Select-Object PathName
```

### Cron jobs (Linux)

```bash
# view the malicious cron entry
crontab -l -u <username>

# edit and remove the malicious line
crontab -e -u <username>

# or remove the user's entire crontab (only if you're sure)
crontab -r -u <username>

# check and clean cron.d entries
ls -la /etc/cron.d/
cat /etc/cron.d/<suspicious_file>
rm /etc/cron.d/<suspicious_file>
```

### Malicious services (Linux)

```bash
# stop the service
systemctl stop <malicious_service>

# disable it
systemctl disable <malicious_service>

# remove the service file
rm /etc/systemd/system/<malicious_service>.service

# reload systemd
systemctl daemon-reload

# verify it's gone
systemctl status <malicious_service>
```

---

## Verification — confirming eradication

Before marking a system as clean and returning it to service:

```powershell
# Windows verification checklist
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"  # no malicious entries
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}              # no malicious tasks
Get-Service | Where-Object {$_.Status -eq "Running"}                    # no malicious services
Get-Process | Select-Object Name, Id, Path                              # no malicious processes
Get-NetTCPConnection -State Established                                 # no C2 connections
```

```bash
# Linux verification checklist
ps aux | grep -v '\[.*\]'          # review all non-kernel processes
ss -tulnp                          # no unexpected listeners
crontab -l -u root                 # root cron clean
for user in $(cut -d: -f1 /etc/passwd); do crontab -l -u $user 2>/dev/null; done  # all users clean
systemctl list-units --type=service --state=active  # no malicious services
find /tmp /var/tmp /dev/shm -executable -type f     # no executables in temp
```

> Re-run the same IOC checks from the initial investigation after eradication. If the same C2 IP shows up in new network connections, persistence was missed. If the same hash appears in a new location, there are additional copies.
