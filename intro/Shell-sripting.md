# Shell Scripting README (Basics to Intermediate)

This guide is structured for long-term study and interview revision.
It covers daily Linux commands, shell fundamentals, Bash scripting, permissions, automation examples, and intermediate command-line tools.

---

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [Linux and Shell Basics](#linux-and-shell-basics)
3. [Daily Commands (Must Know)](#daily-commands-must-know)
4. [File Permissions and chmod](#file-permissions-and-chmod)
5. [Shebang, sh, bash, dash](#shebang-sh-bash-dash)
6. [Shell Scripting Fundamentals](#shell-scripting-fundamentals)
7. [Daily Intermediate Commands](#daily-intermediate-commands)
8. [DevOps-Oriented Monitoring Commands](#devops-oriented-monitoring-commands)
9. [Automation Examples](#automation-examples)
10. [Interview Questions and Strong Answers](#interview-questions-and-strong-answers)
11. [Practice Tasks](#practice-tasks)
12. [Quick Revision Sheet](#quick-revision-sheet)

---

## How to Use This Guide

- Read once from top to bottom.
- Practice each command directly on terminal.
- Re-write scripts by hand (do not copy-paste only).
- Keep this as your short revision before interviews.

---

## Linux and Shell Basics

### What is Linux?
Linux is an operating system used widely in servers, cloud, and DevOps.

### What is a Shell?
A shell is a command interpreter that takes your commands and executes them in the OS.

Examples of shells:

- `sh`
- `bash`
- `dash`
- `ksh`
- `zsh`

### What is Bash?
Bash means Bourne Again SHell.
It is both:

- An interactive shell
- A scripting language used for automation

---

## Daily Commands (Must Know)

## 1) Navigation and Directory Commands

```bash
pwd                 # Print current working directory
ls                  # List files/folders
ls -l               # Long format
ls -la              # Long format + hidden files
ls -ltr             # Long + sort by time + reverse order
cd /path            # Move to specific path
cd ..               # Go one level up
cd ~                # Go to home directory
cd -                # Go to previous directory
mkdir project       # Create directory
mkdir -p a/b/c      # Create nested directories
```

## 2) File Operations

```bash
touch file.txt                      # Create empty file
cat file.txt                        # Show file content
cat > notes.txt                     # Create file by typing (Ctrl + D to save)
cp src.txt dst.txt                  # Copy file
cp -r dir1 dir2                     # Copy folder recursively
mv old.txt new.txt                  # Rename file
mv file.txt /tmp/                   # Move file
rm file.txt                         # Remove file
rm -r folder                        # Remove folder recursively
rm -rf folder                       # Force remove (be careful)
```

## 3) Read Files Quickly

```bash
head -n 20 app.log                  # First 20 lines
tail -n 20 app.log                  # Last 20 lines
tail -f app.log                     # Follow new logs in real time
less app.log                        # Scrollable view (q to quit)
wc -l app.log                       # Count lines
```

## 4) Search and Text Utilities

```bash
grep "ERROR" app.log               # Search text
grep -i "failed" app.log           # Case-insensitive search
grep -r "db_host" /etc/myapp/      # Recursive search
sort users.txt                      # Sort lines
uniq users.txt                      # Remove adjacent duplicates
sort users.txt | uniq               # Unique sorted lines
cut -d: -f1 /etc/passwd             # First field separated by ':'
tr 'a-z' 'A-Z' < input.txt          # Lowercase to uppercase
```

## 5) Help and Documentation

```bash
man ls                              # Manual for ls
ls --help                           # Quick help
which bash                          # Path of executable
```

---

## File Permissions and chmod

## Permission Basics

Linux permissions apply to:

- User/Owner (`u`)
- Group (`g`)
- Others (`o`)

Permission types:

- Read (`r`) = 4
- Write (`w`) = 2
- Execute (`x`) = 1

## Numeric Permission Table

| Number | Permission | Meaning |
|--------|------------|---------|
| 0 | --- | No permission |
| 1 | --x | Execute only |
| 2 | -w- | Write only |
| 3 | -wx | Write + execute |
| 4 | r-- | Read only |
| 5 | r-x | Read + execute |
| 6 | rw- | Read + write |
| 7 | rwx | Read + write + execute |

## Common chmod Values

```bash
chmod 755 script.sh    # rwxr-xr-x (common for scripts)
chmod 644 file.txt     # rw-r--r-- (common for normal files)
chmod 700 secret.sh    # rwx------ (owner only)
chmod 600 secret.txt   # rw------- (sensitive file)
chmod 777 file         # rwxrwxrwx (avoid unless required)
```

## Symbolic chmod

```bash
chmod u+x script.sh    # Add execute to user
chmod g-w file.txt     # Remove write from group
chmod o+r file.txt     # Add read for others
chmod a+x run.sh       # Add execute for all
```

## Understand `ls -l` Output

Example:

```text
-rwxr-xr-- 1 abhishek devops 120 Aug 17 script.sh
```

Meaning:

- `-` regular file (`d` would be directory)
- `rwx` owner permissions
- `r-x` group permissions
- `r--` others permissions

---

## Shebang, sh, bash, dash

## Shebang

A shebang tells the OS which interpreter to use.

```bash
#!/bin/bash
```

Other examples:

- `#!/bin/sh`
- `#!/bin/dash`
- `#!/bin/ksh`

## Why `bash` and `sh` are used interchangeably

Because many scripts use only POSIX-compatible syntax that runs on both.

But they are not identical.

Bash-only features include:

- Arrays
- `[[ ... ]]` tests
- Brace expansion `{1..10}`
- Advanced parameter expansion

If script needs Bash features, use `#!/bin/bash`.

## `/bin/sh` relation with bash/dash

Historically, many systems linked `/bin/sh` to `bash`.
Modern Debian/Ubuntu usually link `/bin/sh` to `dash` for speed.

Check with:

```bash
ls -l /bin/sh
```

---

## Shell Scripting Fundamentals

## Make and Run Script

```bash
touch hello.sh
chmod +x hello.sh
./hello.sh
```

Example script:

```bash
#!/bin/bash

echo "Hello DevOps"
```

## Variables

```bash
#!/bin/bash
name="Abhishek"
echo "Hello, $name"
```

## User Input

```bash
#!/bin/bash
read -p "Enter environment: " env
echo "Deploying to $env"
```

## Conditions

```bash
#!/bin/bash
num=10
if [ "$num" -gt 5 ]; then
  echo "Greater than 5"
else
  echo "5 or less"
fi
```

## Loops

```bash
#!/bin/bash
for srv in app1 app2 app3; do
  echo "Checking $srv"
done
```

```bash
#!/bin/bash
count=1
while [ "$count" -le 3 ]; do
  echo "Run $count"
  count=$((count + 1))
done
```

## Functions

```bash
#!/bin/bash
health_check() {
  echo "Host: $(hostname)"
  echo "Memory:"
  free -h
}

health_check
```

## Exit Codes

- `0` means success
- non-zero means error

```bash
#!/bin/bash
cp source.txt backup.txt
if [ $? -eq 0 ]; then
  echo "Copy success"
else
  echo "Copy failed"
fi
```

## Best Practices (Important)

Use these at top of reliable scripts:

```bash
#!/bin/bash
set -euo pipefail
```

Meaning:

- `-e`: stop on command error
- `-u`: error on undefined variable
- `-o pipefail`: fail if any command in pipeline fails

Also follow:

- Quote variables: `"$var"`
- Use absolute paths in cron scripts
- Log important output to files
- Add clear error messages

---

## Daily Intermediate Commands

## Find Files and Directories

```bash
find /var/log -name "*.log"                   # Find logs
find . -type f -size +100M                     # Files larger than 100MB
find . -type f -mtime -1                       # Modified in last 1 day
```

## xargs (Batch Execution)

```bash
find . -name "*.log" | xargs rm -f            # Delete matched logs
cat servers.txt | xargs -I{} ssh {} hostname   # Run command on each host
```

## sed (Stream Editor)

```bash
sed 's/dev/prod/g' config.txt                  # Replace text in output
sed -i 's/dev/prod/g' config.txt               # Replace text in file
sed -n '1,20p' app.log                         # Print line range
```

## awk (Pattern + Columns)

```bash
awk '{print $1}' file.txt                      # Print first column
awk -F: '{print $1}' /etc/passwd               # First field split by ':'
df -h | awk 'NR==1 || $5+0 > 80 {print}'      # Show disks above 80%
```

## Process Commands

```bash
ps -ef                                         # All processes
ps -ef | grep nginx                            # Search process
pgrep sshd                                     # Process IDs by name
kill 1234                                      # Stop process by PID
kill -9 1234                                   # Force kill (last option)
```

## Archive and Compression

```bash
tar -cvf backup.tar project/                   # Create tar
tar -xvf backup.tar                            # Extract tar
tar -czvf backup.tar.gz project/               # Create compressed tar
gzip app.log                                   # Compress file
```

---

## DevOps-Oriented Monitoring Commands

## CPU, Memory, Disk

```bash
nproc                                           # CPU cores
free -h                                         # RAM usage
df -h                                           # Disk usage
du -sh /var/log                                 # Directory size
top                                             # Real-time monitoring
htop                                            # Better interactive monitoring (if installed)
```

## Uptime and Load

```bash
uptime                                          # Uptime + load average
cat /proc/loadavg                               # Load values
```

## Network

```bash
ip a                                            # IP addresses
ss -tulnp                                       # Open ports and services
ping -c 4 8.8.8.8                               # Basic network test
curl -I https://example.com                     # HTTP header check
```

## System Logs and Services

```bash
journalctl -xe                                  # Recent system errors
journalctl -u nginx --since "1 hour ago"       # Service logs
systemctl status nginx                          # Service status
systemctl restart nginx                         # Restart service
```

---

## Automation Examples

## 1) Node Health Check Script

```bash
#!/bin/bash
set -euo pipefail

echo "===== NODE HEALTH ====="
echo "Hostname: $(hostname)"
echo "User: $(whoami)"
echo "Uptime: $(uptime -p)"
echo "CPU Cores: $(nproc)"
echo "Memory:"
free -h
echo "Disk:"
df -h
echo "Load: $(cat /proc/loadavg)"
echo "======================="
```

## 2) Disk Alert Script

```bash
#!/bin/bash
set -euo pipefail

threshold=80
usage=$(df / | awk 'NR==2 {gsub("%", "", $5); print $5}')

echo "Disk Usage: ${usage}%"
if [ "$usage" -ge "$threshold" ]; then
  echo "WARNING: Disk usage above ${threshold}%"
else
  echo "Disk usage is normal"
fi
```

## 3) Multi-Server Ping Check

`servers.txt` example:

```text
server1
server2
server3
```

Script:

```bash
#!/bin/bash
set -euo pipefail

while read -r server; do
  if ping -c 1 -W 1 "$server" > /dev/null 2>&1; then
    echo "$server UP"
  else
    echo "$server DOWN"
  fi
done < servers.txt
```

## 4) Daily Log Backup with Date

```bash
#!/bin/bash
set -euo pipefail

src="/var/log/application.log"
dst="/backup"
stamp=$(date +%F_%H-%M-%S)

mkdir -p "$dst"
cp "$src" "$dst/application-$stamp.log"
echo "Backup completed: $dst/application-$stamp.log"
```

---

## Interview Questions and Strong Answers

## Q1) Why shell scripting instead of tools in previous organization?

Suggested answer:

In my previous organization, I managed around 1,000 VMs and needed to automate node health checks. Manual checks were time-consuming and error-prone, so I created shell scripts to collect CPU, memory, disk, load, process, and uptime data in one run.

We had some tools available, but our requirement included custom parameters and output format not directly supported by those tools. Shell scripts gave us fast customization, low overhead, and immediate automation using native Linux commands.

## Q2) Why not Python for automation?

Suggested answer:

Python is excellent for complex workflows, APIs, integrations, and large-scale application logic. In this case, most tasks were OS-level checks and command execution, so shell scripting was the fastest and simplest fit.

If the requirement grows into complex logic, external API calls, structured reporting, or larger maintainable modules, I would move to Python.

---

## Practice Tasks

1. Create a script that checks if a service is running; if not, restart it.
2. Write a script to find files older than 7 days and archive them.
3. Build a script to monitor disk usage for all mounted filesystems.
4. Parse `/etc/passwd` and print only usernames using `awk`.
5. Create a menu-based script using `case` for common admin tasks.

---

## Quick Revision Sheet

```text
pwd             current directory
ls -la          list all including hidden
ls -ltr         old to new by modified time
mkdir -p        create nested directories
touch           create file
cp -r           copy directory
mv              move/rename
rm -rf          force remove (careful)
cat/head/tail   read files quickly
grep            search text
find            find files by name/type/time/size
sed             replace/edit stream
awk             column and pattern processing
chmod           permission changes
chown           owner/group change
ps/top/htop     process and resource monitoring
free -h         memory usage
df -h           disk usage
systemctl       service management
journalctl      system/service logs
ssh/scp         remote access and copy
```

---

## Final Reminder

Shell scripting in DevOps is about this flow:

Manual repeated task -> convert commands to script -> make executable -> run consistently -> save time and reduce errors.

For simple to medium Linux automation, Bash is often the fastest practical choice.
For complex workflows and integrations, Python is usually better.
