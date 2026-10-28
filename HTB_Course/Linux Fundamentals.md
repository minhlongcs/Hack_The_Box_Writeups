---
tags: [htb-academy, linux, fundamentals]
course: Linux Fundamentals
---

# Linux Fundamentals

> [!info] Contents
> 1. [[#1. Linux Structure]]
> 2. [[#2. Distributions]]
> 3. [[#3. The Shell]]
> 4. [[#4. Prompt Description]]
> 5. [[#5. Getting Help]]
> 6. [[#6. System Information]]
> 7. [[#7. Navigation]]
> 8. [[#8. Working with Files and Directories]]
> 9. [[#9. Editing Files]]
> 10. [[#10. Finding Files and Directories]]
> 11. [[#11. File Descriptors and Redirections]]
> 12. [[#12. Filtering Contents]]
> 13. [[#13. Regular Expressions]]
> 14. [[#14. Permission Management]]
> 15. [[#15. User Management]]
> 16. [[#16. Package Management]]
> 17. [[#17. Service and Process Management]]
> 18. [[#18. Task Scheduling]]
> 19. [[#19. Network Services]]
> 20. [[#20. Working with Web Services]]
> 21. [[#21. Backup and Restore]]
> 22. [[#22. File System Management]]
> 23. [[#23. Containerization]]
> 24. [[#24. Network Configuration]]
> 25. [[#25. Remote Desktop Protocols in Linux]]
> 26. [[#26. Linux Security]]
> 27. [[#27. Firewall Setup]]
> 28. [[#28. System Logs]]
> 29. [[#29. Solaris]]
> 30. [[#30. Shortcuts]]

---

## 1. Linux Structure

**Linux** is an operating system used on personal computers, servers and mobile devices. It is known for being robust, flexible and open source, and comes in many distributions ("distros").

### Philosophy
- **Everything is a file.** Hardware devices, processes and network connections are represented as files, so the same standard tools can read and write them.
- Small, single-purpose programs.
- Programs can be chained together to do complex tasks.
- Avoid captive user interfaces.
- Configuration data is stored in text files.

### Components
| Component | Role |
|---|---|
| **Bootloader** | Code that guides the boot process |
| **OS Kernel** | Main component of the OS; manages hardware resources and I/O devices |
| **Daemons** | Background services (scheduling, printing, multimedia, …) |
| **OS Shell** | Interface between the user and the OS |
| **Graphics Server** | Graphical sub-system ("X" / X-server) that runs graphical programs locally or remotely |
| **Window Manager** | The graphical user interface (GUI) |
| **Utilities** | Programs that perform specific functions for the user or other programs |

### Architecture (bottom → top)
1. **Hardware**: RAM, hard drive, CPU, peripherals.
2. **Kernel**: virtualises and controls hardware resources (CPU, memory, data access). Gives each process its own virtual resources and prevents conflicts between processes.
3. **Shell**: the command-line interface (CLI).
4. **System utilities**: tools that expose OS functionality to the user.

### File System Hierarchy
![[Pasted image 20260928114442.png]]

| Path | Description |
|---|---|
| `/bin` | Essential command binaries |
| `/boot` | Bootloader, kernel executable and files needed to boot |
| `/dev` | Device files for every attached hardware device |
| `/etc` | Local system and application configuration files |
| `/home` | One subdirectory per user for storage |
| `/lib` | Shared libraries required for system boot |
| `/media` | Mount point for removable media (USB drives, …) |
| `/mnt` | Temporary mount point for regular filesystems |
| `/opt` | Optional / third-party tools |
| `/root` | Home directory of the root user |
| `/sbin` | System administration binaries |
| `/tmp` | Temporary files; usually cleared at boot and may be deleted any time |
| `/usr` | Executables, libraries, man pages, etc. |
| `/var` | Variable data: logs, mail, web app files, cron files, … |

---

## 2. Distributions

### Cybersecurity relevance
- Security specialists favour Linux because its open-source code can be inspected, optimised and specially configured.
- Popular security distros: **Kali Linux** (most popular), **ParrotOS**, **BlackArch**, **BackBox**, **Pentoo**.

### Debian overview
- **Package management:** `apt` (Advanced Package Tool) installs software and automates security updates.
- **Stability:** Long-Term Support (LTS) releases get up to 5 years of updates, which suits 24/7 servers.
- **Trade-off:** steeper learning curve and more complex setup, in exchange for deep control and flexibility.

---

## 3. The Shell

- The most common Linux shell is **Bash** (Bourne-Again Shell), part of the GNU project.
- Anything done in the GUI can be done in the shell, usually faster, and it can be automated with scripts.
- Other shells: [Tcsh/Csh](https://en.wikipedia.org/wiki/Tcsh), [Ksh](https://en.wikipedia.org/wiki/KornShell), [Zsh](https://en.wikipedia.org/wiki/Z_shell), [Fish](https://en.wikipedia.org/wiki/Friendly_interactive_shell).

---

## 4. Prompt Description

- **`PS1`** is the environment variable that defines what the primary prompt looks like.
- It is usually set in `~/.bashrc` and can contain text, colours and dynamic data (user, host, directory, IP, time, exit status).
- **Pentesting use:** a custom `PS1` keeps the current context visible (target IP, full path). Combined with `script` or `~/.bash_history`, it makes documenting an engagement easier.

### Common escape sequences
| Sequence | Description |
|---|---|
| `\u` | Current username |
| `\h` / `\H` | Short hostname / full domain hostname |
| `\w` | Full path of current working directory |
| `\d` | Date (`Weekday Month Day`, e.g. Mon Feb 6) |
| `\D{format}` | Custom date, e.g. `\D{%Y-%m-%d}` |
| `\t` / `\T` / `\@` | Time: 24-hr / 12-hr / 12-hr am/pm |
| `\j` | Number of jobs managed by the shell |
| `\s` | Name of the shell |
| `\n` / `\r` | Newline / carriage return |

---

## 5. Getting Help

`apropos` searches the short descriptions of all man pages for a keyword.

```bash
apropos <keyword>
apropos sudo
```

---

## 6. System Information

### SSH
Encrypted remote command-line access, no GUI needed.
```bash
ssh <username>@<target_ip>
```

### Commands
| Command | Description |
|---|---|
| `whoami` | Current username |
| `id` | User identity (UID, GID, groups) |
| `hostname` | Print or set the host name |
| `uname` | OS and hardware information |
| `pwd` | Current working directory |
| `ifconfig` | View/configure network interfaces |
| `ip` | Show/manipulate routing, devices, interfaces, tunnels |
| `netstat` | Network status |
| `ss` | Investigate sockets |
| `ps` | Process status |
| `who` | Who is logged in |
| `env` | Print environment variables, or run a command with a modified environment |
| `lsblk` | List block devices |
| `lsusb` | List USB devices |
| `lsof` | List open files |
| `lspci` | List PCI devices |

### Enumeration notes (pentesting)
- **`whoami`**: the first thing to run after getting a shell, to confirm which user you are.
- **`id`**: check group memberships. Groups worth noticing:
  - `sudo`: can run commands as root, a key privilege escalation target.
  - `adm`: can read logs in `/var/log`, which may leak sensitive data.
  - Non-standard groups (e.g. `hackthebox`): often linked to custom apps or permissions.
- **`uname`**:
  - `uname -a`: all details (kernel name, hostname, kernel release, kernel version, architecture, OS).
  - `uname -r`: kernel release only (e.g. `4.15.0-99-generic`). Use it to search for public kernel exploits.

---

## 7. Navigation

> [!example] Lab connection
> `ssh htb-student@10.129.227.243` with password `HTB_@cademy_stdnt!`

### Hidden files (dotfiles)
- Files starting with `.` are hidden by default.
- Show them with `ls -la`, e.g. `ls -la /home/htb-user`.
- The Bash command history file is `.bash_history`.

### Inodes
- An **inode** stores a file's metadata (permissions, owner, size, block locations), but not its content or name.
- Show the inode number: `ls -i /etc/sudoers` or `stat -c %i /etc/sudoers`.

---

## 8. Working with Files and Directories

| Command | Example | Purpose |
|---|---|---|
| `touch` | `touch info.txt` | Create an empty file or update its timestamps |
| `mkdir` | `mkdir Storage` | Create a directory |
| `mkdir -p` | `mkdir -p Storage/local/user/documents` | Create nested directories in one go |
| `tree` | `tree .` | Show the directory structure as a tree |
| `mv` (rename) | `mv info.txt information.txt` | Rename a file or directory |
| `mv` (move) | `mv file1.txt file2.txt Storage/` | Move one or more files into a directory |
| `cp` | `cp Storage/readme.txt Storage/local/` | Copy a file or directory |

---

## 9. Editing Files

### Nano
- Simple, modeless editor: `nano notes.txt` opens or creates the file.
- Shortcuts (`^` = `Ctrl`):
  - `Ctrl+O`: save ("write out")
  - `Ctrl+X`: exit
  - `Ctrl+W`: search (repeat with `Ctrl+W` then `Enter`)
- Check the result with `cat notes.txt`.

### Vim
- Modal, lightweight editor that works well with `grep`, `awk`, `sed`.
- **Exit:** `Esc`, then `:q` (quit), `:q!` (discard changes) or `:wq` (save and quit).
- **Learn:** run `vimtutor` in the shell or `:Tutor` inside Vim (about 25–30 minutes).

| Mode | Key | Function |
|---|---|---|
| **Normal** | `Esc` | Default mode; keys are commands, not text |
| **Insert** | `i`, `a`, `o` | Type text into the buffer |
| **Visual** | `v`, `V`, `Ctrl+v` | Select text for copy/delete/replace |
| **Command** | `:` | Single-line commands (search/replace, save, quit) |
| **Replace** | `R` | Overwrite characters under the cursor |
| **Ex** | `Q` | Batch command-line mode for several commands in a row |

### `/etc/passwd` vs `/etc/shadow`
- **`/etc/passwd`**: account info (username, UID, GID, home, shell). The password field now shows `x`, meaning the hash is in `/etc/shadow`. If this file is writable, it can lead directly to privilege escalation.
- **`/etc/shadow`**: salted password hashes; readable by root only.

---

## 10. Finding Files and Directories

| Tool | How it works |
|---|---|
| `which` | Shows the path of an executable, e.g. `which python` → `/usr/bin/python` |
| `find` | Searches the live filesystem by name, size, date, owner, type; can run actions with `-exec` |
| `locate` | Searches a prebuilt index, so it is faster than `find`. Refresh it with `sudo updatedb`; it has fewer filters |

### `find` options
| Option | Meaning | Example |
|---|---|---|
| `-type f` | Regular files only | — |
| `-name <pattern>` | Match a filename pattern | `-name "*.conf"` |
| `-size <range>` | Filter by size | `+25k` (> 25 KB), `-28k` (< 28 KB) |
| `-newermt <date>` | Modified after a date | `-newermt "2020-03-03"` |
| `-user <name>` | Owned by a user | `-user root` |
| `-exec <cmd> {} \;` | Run a command on each result | `-exec ls -al {} \;` |
| `2>/dev/null` | Hide error/permission messages | — |

### Lab answers
1. **Full path of `xxd`:** `which xxd` → `/usr/bin/xxd`
2. **Number of `.bak` files:**
   ```bash
   find / -type f -name "*.bak" 2>/dev/null | wc -l
   ```
3. **`.conf` file modified after 2020-03-03, between 25k and 28k:**
   ```bash
   find / -type f -name "*.conf" -newermt "2020-03-03" -size +25k -size -28k 2>/dev/null
   ```

---

## 11. File Descriptors and Redirections

A **file descriptor (FD)** is a non-negative integer the kernel uses as a handle for an open file, device or socket, like a coat-check ticket. Windows calls this a "file handle".

| FD | Stream | Default | Common redirection |
|---|---|---|---|
| `0` | STDIN | Keyboard | `< file.txt` |
| `1` | STDOUT | Terminal | `> out.txt` (overwrite), `>> out.txt` (append) |
| `2` | STDERR | Terminal | `2>/dev/null`, `2> err.txt` |

### Redirection cheat sheet
| Goal | Command |
|---|---|
| Discard errors | `command 2>/dev/null` |
| Overwrite file | `command > results.txt` |
| Append to file | `command >> results.txt` |
| Split streams | `command 1> stdout.txt 2> stderr.txt` |
| File as input | `cat < input.txt` |
| Heredoc | `cat << EOF > file.txt` (type lines, end with `EOF`) |

### Pipes `|`
A pipe sends the STDOUT of the left command into the STDIN of the right command.
```bash
find /etc/ -name "*.conf" 2>/dev/null | grep systemd | wc -l
```
1. `find` lists `.conf` files in `/etc/` and hides errors.
2. `grep systemd` keeps only lines containing "systemd".
3. `wc -l` counts the lines.

---

## 12. Filtering Contents

| Tool | Role | Key flags | Example |
|---|---|---|---|
| `more` | Pager, forward only; output stays on screen after exit | `Space` next page, `q` quit | `more /etc/passwd` |
| `less` | Pager, forward and backward; screen is cleared on exit | `b` back, `Space` forward, `q` quit | `less /etc/passwd` |
| `head` | First lines (default 10) | `-n <N>` | `head -n 5 /etc/passwd` |
| `tail` | Last lines (default 10) | `-n <N>`, `-f` follow a log | `tail -n 5 /etc/passwd` |
| `sort` | Sort lines | `-n` numeric, `-r` reverse, `-k` key column | `sort /etc/passwd` |
| `grep` | Match patterns | `-v` invert, `-i` ignore case, `-E` extended regex, `-c` count | `grep -v "nologin" /etc/passwd` |
| `cut` | Extract fields | `-d` delimiter, `-f` fields | `cut -d: -f1,3 /etc/passwd` |
| `tr` | Replace/delete characters | `tr "a" "b"`, `-d` | `tr ":" " " < /etc/passwd` |
| `column` | Align output into a table | `-t`, `-s` separator | `tr ":" " " < /etc/passwd \| column -t` |
| `awk` | Field processing | `-F` separator, `'{print $1, $NF}'` | `awk -F: '{print $1, $3}' /etc/passwd` |
| `sed` | Stream editing / substitution | `'s/find/replace/g'` | `sed 's/bin/HTB/g' /etc/passwd` |
| `wc` | Count | `-l` lines, `-w` words, `-c` bytes | `grep bash /etc/passwd \| wc -l` |

### Practice exercises
`/etc/passwd` format: `username:password:UID:GID:GECOS:home:shell` → `$1` username, `$3` UID, `$7` shell.

1. Line for user `cry0l1t3`
   ```bash
   grep "^cry0l1t3:" /etc/passwd
   ```
2. All usernames
   ```bash
   cut -d: -f1 /etc/passwd
   ```
3. `cry0l1t3` and UID
   ```bash
   grep "^cry0l1t3:" /etc/passwd | cut -d: -f1,3
   ```
4. `cry0l1t3` and UID, comma-separated
   ```bash
   grep "^cry0l1t3:" /etc/passwd | cut -d: -f1,3 | tr ":" ","
   ```
5. `cry0l1t3`, UID and shell, comma-separated
   ```bash
   grep "^cry0l1t3:" /etc/passwd | awk -F: '{print $1","$3","$7}'
   ```
6. All users with UID and shell
   ```bash
   awk -F: '{print $1","$3","$7}' /etc/passwd
   ```
7. Same, excluding `nologin` and `false`
   ```bash
   grep -vE "nologin|false" /etc/passwd | awk -F: '{print $1","$3","$7}'
   ```
8. Same, excluding `nologin`, counted
   ```bash
   grep -v "nologin" /etc/passwd | awk -F: '{print $1","$3","$7}' | wc -l
   ```

---

## 13. Regular Expressions

| Syntax | Meaning |
|---|---|
| `( … )` | Group sub-expressions into one unit |
| `[ … ]` | Character class: any one listed character, e.g. `[a-z0-9]` |
| `{ … }` | Repetition range, e.g. `{1,3}` |
| `\|` | OR (needs `grep -E`) |
| `.*` | Any characters in between, i.e. "A followed later by B" |
| `^` / `$` | Start / end of line |
| `\b` | Word boundary |

### Exercises (`/etc/ssh/sshd_config`)
| Task | Command |
|---|---|
| Lines without `#` | `grep -v "#" /etc/ssh/sshd_config` |
| Without `#` and blank lines | `grep -vE "(#\|^$)" /etc/ssh/sshd_config` |
| Word starting with `Permit` | `grep -E "\bPermit" /etc/ssh/sshd_config` |
| Word ending with `Authentication` | `grep -E "Authentication\b" /etc/ssh/sshd_config` |
| Exact word `Key` | `grep -E "\bKey\b" …` or `grep -w "Key" …` |
| Starts with `Password` and contains `yes` | `grep -E "^Password.*yes" /etc/ssh/sshd_config` |
| Ends with `yes` | `grep -E "yes$" /etc/ssh/sshd_config` |

---

## 14. Permission Management

### Change permissions: `chmod`
Who: `u` owner, `g` group, `o` others, `a` all. Operators: `+` add, `-` remove, `=` set exactly.
```bash
chmod a+r shell        # everyone can read
chmod u+x,g-w script   # owner can execute, group loses write
chmod 754 shell        # octal: r=4 w=2 x=1 → rwxr-xr--
```

### Change owner: `chown`
```bash
chown <user>:<group> <file/directory>
```

### SUID / SGID
- **What it does:** the program runs with the permissions of the file's owner (SUID) or group (SGID), not those of the user who runs it.
- **How to spot it:** an `s` replaces the `x` in the owner field (SUID) or group field (SGID), e.g. `-rwsr-xr-x`.
- **Risk:** if a root-owned SUID binary can run commands or spawn a shell (e.g. `journalctl`), a normal user can get a root shell. Check [GTFOBins](https://gtfobins.github.io/).
- **Enumerate:**
  ```bash
  find / -perm -4000 2>/dev/null   # SUID
  find / -perm -2000 2>/dev/null   # SGID
  ```

### Sticky bit
- **What it does:** in a shared directory like `/tmp`, only the file owner, the directory owner or root can delete or rename a file, even if others can write to the directory.
- **How to spot it:** shown in the execute position of the "others" field:
  - `t`: sticky bit set and others have execute (`+x`).
  - `T`: sticky bit set and others don't have execute (`-x`), so they can't traverse the directory.

---

## 15. User Management

**Why it matters:** admins create users, assign groups and run commands as other users. Groups control who can read or change which files. Knowing how users, groups and authentication work helps you find misconfigurations when attacking or auditing.

### Running commands as another user
```bash
cat /etc/shadow        # Permission denied (root-only file)
sudo cat /etc/shadow   # works if your account has sudo rights
```
`sudo` ("superuser do") runs one command as another user (usually root) without logging in as root, which is the recommended practice.

### Key commands
| Command | Purpose |
|---|---|
| `sudo` | Run a command as another user (root by default) |
| `su` | Switch user (root by default) after PAM authentication and open a shell |
| `useradd` | Create a user or update the defaults for new users |
| `userdel` | Delete a user and related files |
| `usermod` | Modify a user account |
| `addgroup` / `delgroup` | Add / remove a group |
| `passwd` | Change a user's password |

> [!tip] Practice
> On the lab target: create users, give them files and directories, filter data from system files and redirect it into the new users' folders. Reset the target if something breaks.

---

## 16. Package Management

**Packages** are archives that hold a program's binaries, config files, dependency info and update tracking. Package managers provide:
- Package downloading
- Dependency resolution
- A standard binary format (`.deb`, `.rpm`, …)
- Common install and config locations
- Extra system configuration
- Quality control

When a package needs other packages that aren't installed, the manager either warns you or fetches them from a repository first. Removing a package deletes its files based on the package's info.

### Package managers
| Tool | Description |
|---|---|
| `dpkg` | Low-level tool to install, build, remove and manage Debian (`.deb`) packages |
| `apt` | High-level CLI for the package system; resolves dependencies automatically |
| `aptitude` | Alternative high-level front-end to the package manager |
| `snap` | Install/refresh/remove snap packages (self-contained apps for cloud, servers, desktops, IoT) |
| `gem` | Front-end to RubyGems, Ruby's package manager |
| `pip` | Python package installer for packages not in the Debian archive; downloads all requirements first to avoid partial installs |
| `git` | Distributed version control; used to clone tools from GitHub |

### APT
- Used on Debian-based distros. `dpkg` installs single `.deb` files but doesn't fetch dependencies; APT does.
- **Repositories** are labelled stable, testing or unstable. Most distros use the stable "main" one.
  - Repo list: `/etc/apt/sources.list` (Parrot OS: `/etc/apt/sources.list.d/parrot.list`)
- **APT cache**: a local database for looking up packages offline.

| Task | Command |
|---|---|
| Search the cache | `apt-cache search impacket` |
| Package details (version, dependencies, size) | `apt-cache show impacket-scripts` |
| List installed packages | `apt list --installed` |
| Install a package | `sudo apt install impacket-scripts -y` |

### Git: cloning tools
```bash
git clone https://github.com/samratashok/nishang.git ~/nishang
```
Example: **Nishang**, a collection of offensive PowerShell scripts. `git clone` creates the target folder itself, so a separate `mkdir` isn't needed.

### dpkg: installing a downloaded `.deb`
```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/s/strace/strace_4.21-1ubuntu1_amd64.deb
sudo dpkg -i strace_4.21-1ubuntu1_amd64.deb
strace -h    # check that it works
```
> [!warning] `dpkg -i` doesn't install missing dependencies. If it complains, run `sudo apt -f install` to fix them.

---

## 17. Service and Process Management

**Services (daemons)** run in the background without user interaction. Their names often end in `d` (`sshd`, `systemd`).
- **System services**: needed at boot for hardware and core OS setup (like a car's engine).
- **User-installed services**: servers and extra features added by users (like a car's GPS).

**Typical goals:** start/restart, stop, check status/logs, enable/disable at boot, find a service or process.

### systemd, PIDs and `/proc`
- Most modern distros use **systemd** as the init system. It is the first process started at boot and gets **PID 1**.
- Every process has a **PID**; details live in `/proc/<PID>/`.
- A process started by another one has a **PPID** (parent PID) and is its child.

### `systemctl` and `journalctl`
| Task | Command |
|---|---|
| Start a service | `systemctl start ssh` |
| Check status | `systemctl status ssh` |
| Start at boot | `systemctl enable ssh` |
| Confirm it's running | `ps aux \| grep ssh` |
| List all services | `systemctl list-units --type=service` |
| View a service's logs | `journalctl -u ssh.service --no-pager` |

> [!tip] If a service won't start, `journalctl -u <service>` usually shows why.

### Process states
- **Running**
- **Waiting**: waiting for an event or resource
- **Stopped**
- **Zombie**: finished but still has an entry in the process table

### Signals and killing processes
Tools: `kill`, `pkill`, `pgrep`, `killall`. List all signals with `kill -l`.

| # | Signal | Meaning |
|---|---|---|
| 1 | SIGHUP | The controlling terminal was closed |
| 2 | SIGINT | Interrupt (`Ctrl+C`) |
| 3 | SIGQUIT | Quit (`Ctrl+\`) |
| 9 | SIGKILL | Kill immediately, no clean-up |
| 15 | SIGTERM | Ask the program to terminate (default for `kill`) |
| 19 | SIGSTOP | Stop the program; it cannot be caught or ignored |
| 20 | SIGTSTP | Suspend (`Ctrl+Z`); can be resumed later |

```bash
kill -9 <PID>    # force-kill a frozen process
```

### Background and foreground jobs
| Action | Command |
|---|---|
| Suspend the running process | `Ctrl+Z` (sends SIGTSTP) |
| List jobs | `jobs` |
| Resume a suspended job in the background | `bg` |
| Start directly in the background | `ping -c 10 www.hackthebox.eu &` |
| Bring a job back to the foreground | `fg <job_id>`, e.g. `fg 1` |

A suspended job does nothing until you run `bg` or `fg`. Background jobs let you keep using the shell, and the terminal tells you when they finish.

### Running multiple commands
| Separator | Behaviour | Example |
|---|---|---|
| `;` | Runs every command, ignoring errors | `echo 1; ls MISSING_FILE; echo 3` → prints 1, error, 3 |
| `&&` | Stops at the first failing command | `echo 1 && ls MISSING_FILE && echo 3` → prints 1, error |
| `\|` | Passes each command's output into the next | `ps aux \| grep ssh` |

---

## 18. Task Scheduling

Task scheduling runs jobs automatically at set times or intervals: updates, scripts, database maintenance, backups and alerts. It works like a coffee maker set to brew every morning.

> [!warning] Security relevance
> Scheduled tasks are a normal admin tool, but attackers also use them for **persistence**: a cron job or timer can run a malicious script or re-open a backdoor on schedule. Check them during audits. During a pentest they can also be a privilege escalation route, e.g. a root job that runs a script you can write to.

### systemd timers
Three steps:
1. Create a **timer**, which decides *when* the job runs.
2. Create a **service** with the same name, which decides *what* runs.
3. Reload systemd, then start and enable the timer.

**`/etc/systemd/system/mytimer.timer`**
```ini
[Unit]
Description=My Timer

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```
- `[Unit]`: description.
- `[Timer]`: when to run.
  - `OnBootSec`: once, this long after boot.
  - `OnUnitActiveSec`: repeatedly, this long after the last run.
- `[Install]`: where the timer is hooked in (`timers.target`).

**`/etc/systemd/system/mytimer.service`**
```ini
[Unit]
Description=My Service

[Service]
ExecStart=/full/path/to/my/script.sh

[Install]
WantedBy=multi-user.target
```
`multi-user.target` is the normal multi-user startup state, i.e. the services started on a regular boot.

**Activate**
```bash
sudo systemctl daemon-reload          # re-read unit files
sudo systemctl start mytimer.timer
sudo systemctl enable mytimer.timer   # start at boot
systemctl list-timers                 # check scheduled timers
```
> [!note] The module also runs `sudo mkdir /etc/systemd/system/mytimer.timer.d`. That folder is only for optional override ("drop-in") files and isn't needed for a basic timer.

### Cron
Tasks live in a **crontab** file, and the cron daemon runs them on schedule.

**Format:** `minute hour day-of-month month day-of-week command`

| Field | Range |
|---|---|
| Minute | 0–59 |
| Hour | 0–23 |
| Day of month | 1–31 |
| Month | 1–12 |
| Day of week | 0–7 (0 and 7 are both Sunday) |

`*` means every value, and `*/N` means every N units.

**Example crontab**
```text
# System update: every 6 hours
0 */6 * * * /path/to/update_software.sh

# Run scripts: 1st of every month at midnight
0 0 1 * * /path/to/scripts/run_scripts.sh

# Clean DB: every Sunday at midnight
0 0 * * 0 /path/to/scripts/clean_database.sh

# Backups: every Sunday at midnight (7 = Sunday too)
0 0 * * 7 /path/to/scripts/backup.sh
```
Cron can also send notifications on success or failure, and log each run.

**Useful commands and locations**
| Task | Command / path |
|---|---|
| Edit your crontab | `crontab -e` |
| List your crontab | `crontab -l` |
| System-wide jobs | `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.{hourly,daily,weekly,monthly}/` |

### systemd vs cron
| | systemd timers | cron |
|---|---|---|
| Setup | A `.timer` file plus a `.service` file | One line in a crontab |
| Scheduling | Relative to boot or last run, or calendar times | Calendar fields |
| Logs | `journalctl -u <service>` | Syslog / mail output |

---

## 19. Network Services

Network services let hosts talk to each other: remote access, file sharing, web hosting, tunnelling. Knowing how they're configured helps you spot weaknesses.

> [!example] Why it matters
> During a pentest you watch traffic from a Linux host and see a user logging in to an **FTP** server. FTP isn't encrypted, so the credentials show up in **plain text**. Knowing which services encrypt traffic, and which don't, is basic security hygiene.

### SSH (OpenSSH)
Encrypted remote shell, command execution, file transfer, tunnelling and port forwarding. **OpenSSH** is the most common server.

| Task | Command |
|---|---|
| Install the server | `sudo apt install openssh-server -y` |
| Check status | `systemctl status ssh` |
| Log in | `ssh cry0l1t3@10.129.17.122` |
| Config file | `/etc/ssh/sshd_config` (max connections, password vs key login, host key checking, …) |

- On the first connection you're asked to trust the host's key fingerprint. It is then saved in `~/.ssh/known_hosts`.
- Edit `sshd_config` carefully, since a mistake can lock you out. Restart the service afterwards: `sudo systemctl restart ssh`.

### NFS (Network File System)
NFS mounts remote directories as if they were local. It's used for central storage, sharing between servers and replication, and can stand in for FTP when no FTP client is available.

| Task | Command |
|---|---|
| Install the server | `sudo apt install nfs-kernel-server -y` |
| Check status | `systemctl status nfs-kernel-server` |
| Config file | `/etc/exports` (what is shared, and to whom) |

**Export options**
| Option | Meaning |
|---|---|
| `rw` | Read and write |
| `ro` | Read-only |
| `root_squash` | Root on the client is treated as an unprivileged user (default) |
| `no_root_squash` | Root on the client keeps root rights on the share |
| `sync` | Writes are confirmed only after they're saved to disk (safer) |
| `async` | Faster, but data can be lost or inconsistent if a write isn't finished |

**Create a share** (server)
```bash
mkdir ~/nfs_sharing
echo '/home/cry0l1t3/nfs_sharing hostname(rw,sync,no_root_squash)' | sudo tee -a /etc/exports
sudo exportfs -ra                 # apply the changes
grep -v "#" /etc/exports          # check
```

**Mount a share** (client)
```bash
showmount -e 10.129.12.17          # list the target's shares
mkdir ~/target_nfs
sudo mount 10.129.12.17:/home/john/dev_scripts ~/target_nfs
tree ~/target_nfs
```

> [!warning] Pentest note
> A share exported with **`no_root_squash`** is a well-known privilege escalation route: files that root creates on it from your machine keep root ownership on the target. Always check the export options.

### Web servers
A web server delivers content over **HTTP/HTTPS** to clients such as browsers, which render the HTML. Common ones on Linux are **Apache**, **Nginx**, **Lighttpd** and **Caddy**. Web servers are frequent pentest targets and also handy for moving files to and from targets.

**Apache**
```bash
sudo apt install apache2 -y
```
Global config: `/etc/apache2/apache2.conf`
```apache
<Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```
| Directive | Meaning |
|---|---|
| `Options Indexes` | List the directory contents if there's no index page |
| `Options FollowSymLinks` | Follow symbolic links |
| `AllowOverride All` | `.htaccess` files may override these settings |
| `Require all granted` | Everyone can access the folder |

- **File transfer:** put files in `/var/www/html`, then fetch them on the target with `wget` or `curl`.
- **`.htaccess`:** settings for one directory (e.g. access control) without editing the main config.
- **Modules:** `mod_rewrite`, `mod_security`, `mod_ssl` add URL rewriting, a web application firewall and HTTPS.

**Python web server** (quick file sharing)
```bash
python3 -m http.server                                        # current folder on TCP/8000
python3 -m http.server --directory /home/cry0l1t3/target_files
sudo python3 -m http.server 443                               # custom port (below 1024 needs root)
```

### VPN
A **VPN** builds an encrypted tunnel to another network, so you work as if you were on it. Companies use VPNs to give employees remote access to internal resources.

- Common solutions: **OpenVPN** (open source, the most popular), L2TP/IPsec, PPTP, SSTP, SoftEther.
- For pentesters: reach internal networks that aren't directly accessible (this is how you connect to HTB labs).

| Task | Command |
|---|---|
| Install | `sudo apt install openvpn -y` |
| Server config | `/etc/openvpn/server.conf` (encryption, tunnelling, traffic shaping, …) |
| Connect with a profile | `sudo openvpn --config internal.ovpn` |

Once connected, you can reach the hosts on the internal network.

---

## 20. Working with Web Services

### Apache
Apache is one of the most widely used web servers (others: **Nginx**, and **IIS** on Windows). Its strength is **modularity**: modules add features.

| Module | Role |
|---|---|
| `mod_ssl` | Encrypts browser ↔ server traffic (HTTPS) |
| `mod_proxy` | Routes requests to other destinations (proxy / reverse proxy) |
| `mod_headers` | Changes HTTP headers |
| `mod_rewrite` | Rewrites URLs on the fly |

Apache also serves **dynamic pages** through server-side languages: PHP, Perl, Ruby, Python, JavaScript, Lua, .NET.

**Install and start**
```bash
sudo apt install apache2 -y
sudo systemctl start apache2      # or: sudo apache2ctl start / sudo service apache2 start
```
- Don't start the `apache2` binary directly: the default config relies on environment variables that those wrappers set.
- Apache listens on **port 80** by default. Browse to `http://localhost` and you should see the "It works!" default page.

**Change the port** (e.g. port 80 is already used on Pwnbox)
Edit `/etc/apache2/ports.conf`:
```apache
Listen 8080
```
Also update the `<VirtualHost *:80>` line in `/etc/apache2/sites-enabled/000-default.conf` to match. Then:
```bash
sudo systemctl restart apache2
curl -I http://localhost:8080     # -I = headers only → "HTTP/1.1 200 OK"
```

### cURL
Transfers data over HTTP, HTTPS, FTP, SFTP, FTPS, SCP and more, and lets you test websites and inspect requests and responses from the shell. It is installed on most systems.
```bash
curl http://localhost             # prints the page source to STDOUT
curl -I http://localhost          # response headers only
curl -v http://localhost          # verbose: full request/response exchange
curl -o page.html http://localhost   # save to a file
```
A browser renders HTML/CSS/JS; curl shows the **raw source**, which is useful for reading titles, comments and hidden data.

### wget
A download manager for HTTP and FTP. Unlike curl, it **saves the content to a file** by default.
```bash
wget http://localhost             # saves as index.html
```

### Python 3 web server
Serves the current directory as the web root, which is great for quick file transfers.
```bash
python3 -m http.server            # 0.0.0.0:8000
```
The terminal **logs every request** (client IP, time, method, path, status code), e.g.
```text
127.0.0.1 - - [15/May/2020 17:56:29] "GET /readme.html HTTP/1.1" 200 -
```
This shows you which files a target actually fetched.

| Tool | Best for |
|---|---|
| `curl` | Inspecting and testing requests/responses; output to the screen |
| `wget` | Downloading files to disk |
| `python3 -m http.server` | Quickly hosting files for another machine |

> [!tip] Mindset
> From here on, the exercises push beyond what the module explains. Research and try different approaches; that's part of learning.

---

## 21. Backup and Restore

### Tools (Ubuntu)
| Tool | Description |
|---|---|
| **Rsync** | Fast local or remote backups; transfers only the **changed parts** of files (efficient for large data and network syncs) |
| **Duplicity** | Built on the rsync algorithm (librsync) and adds **encryption** (GnuPG); can back up to remote servers, FTP or cloud (e.g. Amazon S3) |
| **Deja Dup** | Easy **GUI** front-end (it uses Duplicity behind the scenes); supports encrypted backups |

> [!note] Analogy
> Rsync = a fast truck that only carries what's new. Duplicity = a high-security safe with a combination lock. Deja Dup = a simple safe anyone can operate. Encryption = an extra lock on the safe.

Extra encryption tools: **GnuPG** (files), **eCryptfs** (encrypted directories), **LUKS** (full-disk/partition encryption).

### Rsync
```bash
sudo apt install rsync -y
```

| Task | Command |
|---|---|
| Back up a directory to a server | `rsync -av /path/to/mydirectory user@backup_server:/path/to/backup/directory` |
| Compressed, incremental, mirrored | `rsync -avz --backup --backup-dir=/path/to/backup/folder --delete /path/to/mydirectory user@backup_server:/path/to/backup/directory` |
| Restore | `rsync -av user@remote_host:/path/to/backup/directory /path/to/mydirectory` |
| Encrypted transfer over SSH | `rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory` |

| Option | Meaning |
|---|---|
| `-a` | Archive: keeps permissions, timestamps, owners, symlinks; recursive |
| `-v` | Verbose progress |
| `-z` | Compress during transfer |
| `--backup` + `--backup-dir=` | Keep the old versions of changed or deleted files in that folder (incremental) |
| `--delete` | Delete files on the destination that no longer exist in the source |
| `-e ssh` | Use SSH as the transport, so the transfer is encrypted |

> [!warning] Trailing slash
> `rsync -a src/ dest` copies the **contents** of `src` into `dest`.
> `rsync -a src dest` copies the **folder itself**, giving `dest/src`.

> [!tip] Test first
> Add `-n` (`--dry-run`) to see what would change, especially before using `--delete`.

### Auto-synchronisation (rsync + cron)
A script can't type a password, so set up **SSH key authentication** first.

1. **Create a key pair** (default `~/.ssh/id_rsa`; leave the passphrase empty for unattended use)
   ```bash
   ssh-keygen -t rsa -b 2048
   ```
2. **Copy the public key to the server**
   ```bash
   ssh-copy-id user@backup_server
   ```
3. **Write the script**: `RSYNC_Backup.sh`
   ```bash
   #!/bin/bash
   rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
   ```
4. **Make it executable** (and owned by the right user, so nobody else can change it)
   ```bash
   chmod +x RSYNC_Backup.sh
   ```
5. **Schedule it**: `crontab -e`, then add a line to run it every hour at minute 0:
   ```text
   0 * * * * /path/to/RSYNC_Backup.sh
   ```

### Practice (Pwnbox)
Sync `to_backup` → `synced_backup` on the same machine, using `127.0.0.1` as the "remote" host, every minute.
```bash
mkdir ~/to_backup ~/synced_backup
echo "test" > ~/to_backup/file.txt
ssh-keygen -t rsa -b 2048            # if no key yet
ssh-copy-id $USER@127.0.0.1
rsync -avz -e ssh ~/to_backup/ $USER@127.0.0.1:~/synced_backup/
crontab -e
# * * * * * rsync -az -e ssh /home/<user>/to_backup/ <user>@127.0.0.1:/home/<user>/synced_backup/
```
> [!note] Use full paths in cron: it runs with a minimal environment, and `~` may not expand the way you expect.

---

## 22. File System Management

### File system types
| FS | Journaling | Best for |
|---|---|---|
| **ext2** | ✗ | Older, low overhead (e.g. USB drives) |
| **ext3** | ✓ | Older journaling version of ext |
| **ext4** | ✓ | Default on most Linux systems; good balance of speed, reliability and large-file support |
| **Btrfs** | ✓ (copy-on-write) | Snapshots and built-in data integrity checks; complex storage setups |
| **XFS** | ✓ | Very large files and heavy I/O |
| **NTFS** | ✓ | Windows compatibility (dual boot, shared external drives) |

**Journaling** records changes before they are written, so the file system can recover cleanly after a crash. Choose a file system based on performance, data integrity, compatibility and storage needs.

### Inodes
- An **inode** stores a file's metadata: permissions, owner, size, timestamps and **pointers to its data blocks**. It does **not** store the file's name or content.
- The **inode table** is the kernel's index of every inode on the file system.
- A disk can run out of **inodes** before it runs out of space (e.g. millions of tiny files). Check with `df -i`.

> [!note] Analogy
> Library: the inode = an index card, the inode table = the catalogue, the file = the book itself.

### File types
| Type | `ls -l` first char | Description |
|---|---|---|
| Regular file | `-` | Text (ASCII) or binary data (images, executables, …); can live in any directory |
| Directory | `d` | A container for files; the **parent directory** of what it holds |
| Symbolic link | `l` | A shortcut that points to another file or directory, without copying it |

```bash
ls -il
# 10678872 -rw-r--r-- 1 cry0l1t3 htb 234123 Feb 14 19:30 myscript.py
#   ↑ inode   ↑ perms   ↑ owner  ↑ group
```
Owner, group and others each have their own permissions; changing one doesn't affect the others.

### Disks and partitions
- **Partitioning** splits a physical disk into logical sections. Each one is formatted with a file system (ext4, NTFS, FAT32, …) and can be mounted separately.
- Tools: **`fdisk`** (CLI, the main one), **`gpart`**, **GParted** (GUI).

```bash
sudo fdisk -l      # list disks, partitions, sizes and types
lsblk              # tree view of disks/partitions and where they're mounted
sudo blkid         # partition UUIDs and file system types
df -h              # disk usage of mounted file systems
```
Example `fdisk -l` output: `/dev/vda1` = 75.8G Linux (`*` marks the boot partition), `/dev/vda2` = 4.2G Linux swap.

### Mounting
**Mounting** attaches a partition or drive to a directory (the **mount point**), so its contents appear in the file system tree.

| Task | Command |
|---|---|
| List mounted file systems | `mount` |
| Mount a USB drive | `sudo mount /dev/sdb1 /mnt/usb` |
| Unmount | `sudo umount /mnt/usb` |
| Find processes using it | `lsof \| grep <user>` or `lsof /mnt/usb` |
| Mount everything in fstab | `sudo mount -a` |

- You can't unmount a file system that's in use ("target is busy"). Find the process with `lsof` and stop it first.
- At shutdown, all mounted file systems are unmounted automatically.

### `/etc/fstab`: mounts at boot
Format: `<device> <mount point> <type> <options> <dump> <pass>`
```text
/dev/sda1           /         ext4  defaults          0 0
/dev/sda2           /home     ext4  defaults          0 0
/dev/sdb1           /mnt/usb  ext4  rw,noauto,user    0 0
192.168.1.100:/nfs  /mnt/nfs  nfs   defaults          0 0
```
| Field / option | Meaning |
|---|---|
| `UUID=…` | Identify the device by UUID (more reliable than `/dev/sdX`; find it with `blkid`) |
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `noauto` | **Don't** mount at boot (mount manually when needed) |
| `user` | A normal user may mount it |
| `noatime` | Don't update access times (faster) |
| `dump` | Backup flag (usually `0`) |
| `pass` | fsck order at boot: `1` = root, `2` = others, `0` = skip |

> [!warning] Pentest note
> `/etc/fstab` sometimes contains **credentials** for network shares (e.g. `cifs … username=…,password=…`). It's always worth reading during enumeration. Also look for unmounted drives with `lsblk`.

### Swap
**Swap** is disk space used as overflow RAM. When RAM is full, the kernel moves **inactive memory pages** to swap ("swapping").

| Command | Role |
|---|---|
| `mkswap` | Format a partition or file as a swap area |
| `swapon` | Turn the swap area on |
| `swapon --show` / `free -h` | Show current swap usage |

```bash
# Example: create a 1 GB swap file
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
- **Size** depends on RAM and workload. Systems with lots of RAM may need little or none.
- Use a **dedicated partition or file**.
- **Encrypt swap**: sensitive data from RAM (passwords, keys) can end up on disk there.
- **Hibernation** saves the whole system state to swap and powers off, then restores it at the next boot. This needs swap at least as large as RAM.

---

## 23. Containerization

**Containerization** packages an application with everything it needs (libraries, dependencies, config) into an isolated **container** that behaves the same wherever it runs.
- Containers **share the host's kernel**, so they are much lighter than VMs and many can run on one host. That makes them useful for scaling and microservices.
- Tools: **Docker**, **Docker Compose**, **LXC**.

> [!note] Analogy
> VM = building a whole new stage for each band. Container = portable "stage pods" that each band brings to the same main stage. Each pod is isolated, but they share the venue.

> [!warning] Security
> Containers isolate apps from the host and from each other, but **less strongly than VMs**. Misconfigurations can allow **privilege escalation** or **container escape** to the host (covered in the *Linux Privilege Escalation* module).

### Docker
Open-source platform that runs apps as containers, using a **layered filesystem** and resource isolation.

> [!note] Analogy
> A container is a sealed lunchbox: when you throw it away, everything inside is gone. To change the contents, write a new recipe (**Dockerfile**). To serve many lunchboxes, use a kitchen system (**Docker Compose / Kubernetes**).

#### Install Docker on Kali Linux
Kali is a rolling release based on **Debian**, so the Ubuntu script below doesn't work as-is: `lsb_release -cs` returns `kali-rolling`, which Docker's repository doesn't recognise.

**Option A: Kali's own package (simplest)**
```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker        # start now and at boot
sudo usermod -aG docker $USER             # log out and back in afterwards
docker run hello-world                    # test
```

**Option B: official Docker CE (newest version, uses the Debian repo)**
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
docker run hello-world
```
> [!note] Option B differs from the Ubuntu script in two places: it uses the `debian` repo URL, and it hardcodes the Debian codename `bookworm` instead of `$(lsb_release -cs)`.

| | Option A (`docker.io`) | Option B (`docker-ce`) |
|---|---|---|
| Source | Kali repo | Docker's official repo |
| Version | Slightly older | Latest |
| Compose command | `docker-compose` | `docker compose` |

Check the install: `docker --version`, `systemctl status docker`, and `groups` should list `docker`.

> [!bug] Troubleshooting: `permission denied ... /var/run/docker.sock`
> Group changes only apply to **new** login sessions. After `usermod -aG docker $USER`, run `newgrp docker` (this terminal only) or log out and back in, then check `groups`. Also make sure the service is running (`sudo systemctl enable --now docker`). The test image is `hello-world` (with a hyphen).

#### Install Docker Engine (Ubuntu)
```bash
#!/bin/bash
# Preparation
sudo apt update -y
sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update -y
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Let htb-student run docker without sudo (log out and back in afterwards)
sudo usermod -aG docker htb-student

# Test
docker run hello-world
```
> [!warning] The `docker` group is **root-equivalent**
> Any member can start a container that mounts the host's `/` and gain root on the host. When `id` shows `docker` during enumeration, that's a privilege escalation path.

#### Images, Docker Hub, Dockerfile
- **Image** = a read-only template (filesystem + settings). **Container** = a running instance of an image.
- **Docker Hub** = the cloud registry of images: public (community and official images) or private (company/team only). An image is pulled automatically if it isn't available locally.
- **Dockerfile** = the build instructions for an image.

Example: an Ubuntu 22.04 image with **Apache + SSH**, used as a file-hosting box (upload with `scp`, download on the target with `curl`/`wget`):
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y apache2 openssh-server && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m docker-user && \
    echo "docker-user:password" | chpasswd

RUN chown -R docker-user:docker-user /var/www/html && \
    chown -R docker-user:docker-user /var/run/apache2 && \
    chown -R docker-user:docker-user /var/log/apache2 && \
    chown -R docker-user:docker-user /var/lock/apache2 && \
    usermod -aG sudo docker-user && \
    echo "docker-user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

EXPOSE 22 80

CMD service ssh start && /usr/sbin/apache2ctl -D FOREGROUND
```
| Instruction | Role |
|---|---|
| `FROM` | Base image |
| `RUN` | Run a command at build time (each one adds a layer) |
| `EXPOSE` | Document which ports the container listens on |
| `CMD` | Command run when the container starts |

> [!caution] This Dockerfile uses a hardcoded password and password-less sudo. That's fine for a lab, but never do it in production.

#### Build and run
```bash
docker build -t fs_docker .                          # -t = tag/name (must be lowercase)
docker run -p 8022:22 -p 8080:80 -d fs_docker        # -p host:container, -d = background
```
Syntax: `docker run -p <host port>:<container port> -d <image>`. Here the container's SSH is reachable on host port 8022 and its HTTP on 8080.

> [!note] The module uses the tag `FS_docker`, but Docker rejects uppercase letters in image names. Use `fs_docker`.

#### Management
| Command | Description |
|---|---|
| `docker ps` | List running containers (`-a` = all) |
| `docker stop <id>` | Stop a container |
| `docker start <id>` | Start a stopped container |
| `docker restart <id>` | Restart a container |
| `docker rm <id>` | Remove a container |
| `docker rmi <image>` | Remove an image |
| `docker logs <id>` | View a container's logs |
| `docker exec -it <id> bash` | Open a shell inside a running container |
| `docker images` | List local images |

- Extra options: port mapping (`-p`), **volumes** for persistent data (`-v host_path:container_path`), environment variables (`-e`).
- Changes made inside a running container are **not saved to the image**. To keep them, write a new Dockerfile (`FROM` the base image + your changes) and `docker build` it with a new tag.
- Treat containers as **disposable**: data inside is lost when the container is removed. Keep it in **volumes**.
- At scale, use **Docker Compose** (multi-container apps) or **Kubernetes** (orchestration).

### LXC (Linux Containers)
Lightweight, system-level virtualization: several isolated Linux systems run on one host and share its kernel. Isolation comes from **namespaces** and resources are limited with **cgroups**.

#### Docker vs LXC
| Category | LXC | Docker |
|---|---|---|
| Approach | System-level; behaves like a lightweight VM | Application-focused; one app/microservice per container |
| Image building | Manual: build a root filesystem and install packages | Standardised images built from a Dockerfile |
| Portability | Tied to the host's configuration | Very portable via Docker Hub / registries |
| Ease of use | Needs more Linux admin knowledge | Simple CLI, large community |
| Security | Needs extra configuration to match Docker's defaults | AppArmor/SELinux and read-only filesystem options out of the box |

Both can be a **local privilege escalation** route when misconfigured.

#### Install and create
```bash
sudo apt install lxc -y
sudo lxc-create -n linuxcontainer -t ubuntu    # -n name, -t template
```
LXC needs kernel support for containers. Modern kernels have it; older ones may need extra configuration.

#### Management
| Command | Description |
|---|---|
| `lxc-ls` | List all containers |
| `lxc-info -n <c>` | State, PID, IP, resource stats (`-i` IPs only, `-c <key>` one config key) |
| `lxc-start -n <c>` | Start |
| `lxc-stop -n <c>` | Stop (`-r` = reboot) |
| `lxc-attach -n <c>` | Run a shell or command inside the container |
| `lxc-attach -n <c> -f <file>` | Attach using a custom config file (`--rcfile`) |
| `lxc-console -n <c>` | Attach to the container console |
| `lxc-cgroup -n <c> <key> [value]` | Get or set a live cgroup value |
| `lxc-config -l` / `lxc-config lxc.lxcpath` | List or print **global** LXC settings only |

- **Per-container config**: `/var/lib/lxc/<container>/config` (documented in `man lxc.container.conf`). `/usr/share/lxc/config/` only holds shared templates.
- **Network**: edit `lxc.net.0.type`, `lxc.net.0.link`, `lxc.net.0.flags` in that file, then restart the container.
- **Share a host folder** with a bind mount in the config, then restart:
  ```text
  lxc.mount.entry = /path/on/host path/in/container none bind,create=dir 0 0
  ```
- Don't confuse classic LXC (`lxc-start`, `lxc-attach`) with **LXD**'s `lxc` client (`lxc config …`, space-separated).

#### Why pentesters use containers
- Quickly build an environment with exact versions (e.g. a specific web server or database) without setting it up on your own machine.
- Safely test exploits or malware against a simulated vulnerable system.

#### Securing LXC
- Restrict access (disable unneeded services such as SSH, allow only trusted IPs, use strong authentication)
- Limit resources
- Isolate the container from the host
- Enforce mandatory access control (AppArmor/SELinux)
- Keep the container up to date

**Limit CPU and memory with cgroups**: add to `/var/lib/lxc/linuxcontainer/config`:
```text
lxc.cgroup.cpu.shares = 512
lxc.cgroup.memory.limit_in_bytes = 512M
```
| Key | Meaning |
|---|---|
| `cpu.shares` | Relative CPU weight (default **1024**). `512` = half the share of a default container when the CPU is busy |
| `memory.limit_in_bytes` | Maximum RAM; accepts K, M, G, T |

Apply by restarting the container (no need to restart the whole LXC service):
```bash
sudo lxc-stop -n linuxcontainer
sudo lxc-start -n linuxcontainer
```
Temporary limit on a running container:
```bash
sudo lxc-cgroup -n linuxcontainer cpu.shares 512
sudo lxc-cgroup -n linuxcontainer memory.limit_in_bytes 512M
```
> [!note] cgroup v2
> Newer distros use cgroup v2, where the keys are `lxc.cgroup2.cpu.weight` and `lxc.cgroup2.memory.max`. If the v1 keys above fail, use these.

#### Namespaces
Kernel feature that gives each container its own view of system resources:
| Namespace | Isolates |
|---|---|
| `pid` | Process IDs: the container can't see or interfere with host processes |
| `net` | Network interfaces, routing tables, firewall rules |
| `mnt` | Root filesystem and mounts: changes don't affect the host's filesystem |

Namespaces give strong isolation but **not complete security**, so add other protections too.

### Practice exercises (LXC)
1. Install LXC and create your first container.
2. Configure networking via `lxc.net.0.*` in `/var/lib/lxc/<container>/config`.
3. Create a custom LXC image and launch a container from it.
4. Set resource limits (CPU, memory, disk).
5. Explore the `lxc-*` commands.
6. Run a specific web server version (Apache, Nginx) in a container.
7. Set up SSH access to a container and connect remotely.
8. Create a persistent container whose changes are saved and reusable.
9. Test software (a vulnerable web app or malware) in a controlled container.

### Exercise walkthrough (Ubuntu / Pwnbox)

> [!info] Conventions
> Container name: `web1`. Run host commands with `sudo`; commands inside the container are marked `(container)`. Config file = `/var/lib/lxc/web1/config`.

#### 1. Install LXC and create a container
> [!info] On Kali (Debian-based)
> New containers have **no network** by default on Kali/Debian. Before creating containers, enable the LXC bridge:
> ```bash
> sudo apt install -y lxc lxc-templates debootstrap
> echo 'USE_LXC_BRIDGE="true"' | sudo tee /etc/default/lxc-net
> sudo systemctl enable --now lxc-net          # creates lxcbr0 (10.0.3.1)
> sudo tee /etc/lxc/default.conf << 'CONF'
> lxc.net.0.type = veth
> lxc.net.0.link = lxcbr0
> lxc.net.0.flags = up
> lxc.apparmor.profile = generated
> lxc.apparmor.allow_nesting = 1
> CONF
> ip a show lxcbr0                              # check the bridge exists
> ```
> After that, the steps below work the same. If `lxc-start` fails with an AppArmor error, replace the two `apparmor` lines with `lxc.apparmor.profile = unconfined` (lab use only).

```bash
sudo apt update && sudo apt install lxc lxc-templates -y
sudo lxc-checkconfig                         # kernel support: everything should say "enabled"
sudo lxc-create -n web1 -t download -- -d ubuntu -r jammy -a amd64
sudo lxc-start -n web1
sudo lxc-ls -f                               # state + IP
sudo lxc-attach -n web1                      # shell inside; `exit` to leave
```
The `download` template pulls a ready-made image and is more reliable than `-t ubuntu`, which builds the image locally with debootstrap.

#### 2. Configure networking
Check the default bridge on the host: `ip a show lxcbr0` (normally `10.0.3.1/24`). In the config file:
```text
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
# optional static IP instead of DHCP:
lxc.net.0.ipv4.address = 10.0.3.100/24
lxc.net.0.ipv4.gateway = 10.0.3.1
```
```bash
sudo lxc-stop -n web1 && sudo lxc-start -n web1
sudo lxc-info -n web1 -i                     # shows the IP
sudo lxc-attach -n web1 -- ping -c 2 1.1.1.1 # internet access check
```
If a static IP doesn't stick, the container's own DHCP client may be overriding it. Leave it on DHCP or disable netplan DHCP inside the container.

#### 3. Make a custom image and launch from it
Set up a "golden" container, then clone it:
```bash
sudo lxc-attach -n web1 -- bash -c "apt update && apt install -y curl vim"
sudo lxc-stop -n web1
sudo lxc-copy -n web1 -N web1-clone          # full copy of config + rootfs
sudo lxc-start -n web1-clone
```
Alternative: package the rootfs as an archive you can reuse or move to another host.
```bash
sudo tar -czf web1-rootfs.tar.gz -C /var/lib/lxc/web1 rootfs config
```

#### 4. Resource limits
Check which cgroup version you have: `stat -fc %T /sys/fs/cgroup` → `cgroup2fs` = v2.

| Limit | cgroup v2 key (newer distros) | cgroup v1 key (module) |
|---|---|---|
| Memory | `lxc.cgroup2.memory.max = 512M` | `lxc.cgroup.memory.limit_in_bytes = 512M` |
| CPU weight | `lxc.cgroup2.cpu.weight = 50` (default 100) | `lxc.cgroup.cpu.shares = 512` (default 1024) |
| CPU hard cap | `lxc.cgroup2.cpu.max = 50000 100000` (= 0.5 CPU) | — |

Restart, then verify:
```bash
sudo lxc-attach -n web1 -- cat /sys/fs/cgroup/memory.max
sudo lxc-attach -n web1 -- free -m
```
**Disk:** the default `dir` storage has no size limit. Create the container on a fixed-size loop file instead:
```bash
sudo lxc-create -n small -B loop --fssize 2G -t download -- -d ubuntu -r jammy -a amd64
```

#### 5. Explore the `lxc-*` tools
| Command | Use |
|---|---|
| `lxc-ls -f` | Table of containers, state, IPs |
| `lxc-info -n c` | Details |
| `lxc-top` | Live resource usage |
| `lxc-freeze` / `lxc-unfreeze` | Pause / resume all processes |
| `lxc-snapshot -n c` | Snapshot (`-L` list, `-r snap0` restore) |
| `lxc-copy` | Clone |
| `lxc-destroy -n c` | Delete (stop it first) |
| `lxc-monitor -n c` | Watch state changes |
Use `man lxc-<cmd>` for each.

#### 6. Run a specific web server version
```bash
sudo lxc-attach -n web1
(container) apt update
(container) apt-cache policy nginx           # versions available
(container) apt install -y nginx=<version>   # pin an exact version
(container) apt-mark hold nginx              # stop upgrades replacing it
(container) nginx -v
```
From the host: `curl http://<container IP>`. Different distro releases ship different versions, so pick the release with `-r` (e.g. `focal`, `jammy`) for older or newer ones.

#### 7. SSH into the container
```bash
sudo lxc-attach -n web1
(container) apt install -y openssh-server
(container) useradd -m -s /bin/bash tester && passwd tester
(container) systemctl enable --now ssh
```
From the host: `ssh tester@<container IP>`. Optionally set up keys with `ssh-copy-id tester@<IP>`.
To reach it from **another machine**, forward a host port to the container:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination <container IP>:22
# then: ssh -p 2222 tester@<host IP>
```

#### 8. Persistence
LXC containers are **already persistent**: the rootfs lives in `/var/lib/lxc/web1/rootfs`. Prove it:
```bash
sudo lxc-attach -n web1 -- bash -c "echo hello > /root/proof.txt"
sudo lxc-stop -n web1 && sudo lxc-start -n web1
sudo lxc-attach -n web1 -- cat /root/proof.txt   # still there
```
Extras:
- `lxc.start.auto = 1` in the config: start at host boot.
- Bind-mount a host folder for data that should outlive the container:
  `lxc.mount.entry = /srv/web1-data var/www/html none bind,create=dir 0 0`
- `lxc-snapshot` before risky changes, so you can roll back.

#### 9. Test software in a controlled environment
1. Clone or snapshot a clean container first (`lxc-copy` / `lxc-snapshot`).
2. Set up the vulnerable app inside (e.g. an old Apache/PHP version from exercise 6).
3. **Isolate it**:
   - For untrusted code, cut networking (`lxc.net.0.type = empty`), or keep it on `lxcbr0` only with no port forwards.
   - Don't add host bind mounts.
   - Prefer **unprivileged** containers.
4. Test, observe (`lxc-top`, logs inside), then **restore the snapshot** or `lxc-destroy` and re-clone.

> [!warning] Containers share the host kernel. For real malware, use a full VM; a kernel exploit inside a container can reach the host.

---

## 24. Network Configuration

Configuring and managing network settings lets a pentester build test environments, manipulate traffic and spot or exploit weaknesses. Key protocols to know: **TCP/IP**, **DNS** (names → IPs), **DHCP** (automatic IPs) and **FTP** (unencrypted file transfer). Be able to tell wired from wireless interfaces and troubleshoot connectivity.

> [!note] Analogy
> Interfaces = the building's wiring. NAC = the building's security (open rooms = DAC, restricted rooms = MAC/RBAC). Monitoring = cameras and alarms. Troubleshooting tools = a toolkit: `ping` (broken connection), `nslookup` (faulty lock), `nmap` (vulnerable entrance).

### Configuring network interfaces
`ifconfig` (from the `net-tools` package) is **deprecated** in favour of `ip`, but both are still common. `net-tools` may not be installed by default.

| Task | `ifconfig` / `route` (old) | `ip` (modern) |
|---|---|---|
| Show interfaces | `ifconfig` | `ip addr` (or `ip a`) |
| Bring an interface up | `sudo ifconfig eth0 up` | `sudo ip link set eth0 up` |
| Assign an IP | `sudo ifconfig eth0 192.168.1.2` | `sudo ip addr add 192.168.1.2/24 dev eth0` |
| Set the netmask | `sudo ifconfig eth0 netmask 255.255.255.0` | included in the `/24` above |
| Set the default gateway | `sudo route add default gw 192.168.1.1 eth0` | `sudo ip route add default via 192.168.1.1 dev eth0` |
| Show routes | `route -n` | `ip route` |
| Show neighbours (ARP) | `arp -a` | `ip neigh` |

Reading the output (`ifconfig`/`ip addr`):
- `eth0`, `eth1`: Ethernet interfaces. `lo`: **loopback** (`127.0.0.1`).
- `inet` = IPv4 address, `inet6` = IPv6, `ether` / `link/ether` = MAC address.
- `/18`, `/20` = **CIDR prefix**, the same thing as a netmask: `/24` = `255.255.255.0`, `/16` = `255.255.0.0`.
- `RX`/`TX` = packets received/sent, plus errors and drops (non-zero errors point to cable or driver problems).
- `scope global dynamic` = address came from **DHCP**.

> [!warning] All of the above are **temporary**: they are lost on reboot or when the interface restarts.

### DNS
DNS servers translate names like `example.com` into IP addresses. Wrong DNS is a classic cause of "the network is down" ("it's always DNS").

`/etc/resolv.conf`:
```text
nameserver 8.8.8.8
nameserver 8.8.4.4
```
- Public examples: Google `8.8.8.8` / `8.8.4.4`.
- **Not persistent**: `NetworkManager` or `systemd-resolved` overwrites it. On many systems it is a symlink to a stub resolver (`127.0.0.53`). Check with `resolvectl status`.
- For permanent DNS, set it in the network configuration (below).

### Making changes persistent
The right file depends on the distro:

| Distro / tool | Where to configure | Apply |
|---|---|---|
| Debian (ifupdown) | `/etc/network/interfaces` | `sudo systemctl restart networking` |
| Ubuntu (netplan) | `/etc/netplan/*.yaml` | `sudo netplan apply` |
| Kali / desktop (NetworkManager) | `nmcli` / `nmtui` | takes effect immediately |

**`/etc/network/interfaces`** (the module's example, static IP):
```text
auto eth0
iface eth0 inet static
  address 192.168.1.2
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8 8.8.4.4
```
**Netplan equivalent** (Ubuntu):
```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.2/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
**NetworkManager equivalent:**
```bash
nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.1.2/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con up "Wired connection 1"
```
> [!tip] Set a static IP over SSH carefully: a typo can cut you off. Test with `sudo netplan try` (it rolls back automatically) or keep a console open.

### Network Access Control (NAC)
NAC ensures only **authorised and compliant** devices/users reach resources. Three models:

| Model | Who decides access | Characteristics | Typical use |
|---|---|---|---|
| **DAC** (Discretionary) | The resource **owner** sets permissions (read/write/execute/delete) | Flexible, but depends on owners getting it right. Standard Linux file permissions are DAC | General-purpose systems |
| **MAC** (Mandatory) | The **OS** enforces rules from security **labels** (resource) and **clearances** (user/process). Access only if clearance ≥ label | Very strict, less flexible, owners can't override it | Military, government, finance, healthcare |
| **RBAC** (Role-based) | Permissions come from the user's **role** (job function) | Scalable, easy to manage, least privilege, supports compliance | Large organisations with many users |

### Monitoring
Capturing and analysing traffic to find threats, performance problems and suspicious behaviour. Pentest example: a user logging in to an **unencrypted FTP** server exposes credentials in plain text, which can help you move further or escalate.

Tools: `syslog` / `rsyslog` (logs), `ss` (sockets), `lsof` (open files), the **ELK stack** (Elasticsearch, Logstash, Kibana), and for packet capture **Wireshark**, **tshark**, **tcpdump** (covered in the *Intro to Network Traffic Analysis* module).

### Troubleshooting
Common tools: `ping`, `traceroute`, `netstat`, `tcpdump`, Wireshark, `nmap`.

**ping**: tests reachability and round-trip time with ICMP echo.
```bash
ping 8.8.8.8          # stop with Ctrl+C
ping -c 4 8.8.8.8     # send 4 packets only
```
Output: `ttl` = remaining hops allowed, `time` = round-trip in ms, then packet loss % and min/avg/max/mdev.

**traceroute**: shows each router on the path by sending packets with increasing **TTL**.
```bash
traceroute www.inlanefreight.com
```
- Each line is one **hop**; the three times are three probes.
- `* * *` = no reply: the device is down, **blocks ICMP/UDP probes**, or the packets were dropped. It doesn't necessarily mean a fault.
- In the module's example, hop 1 and hop 3 are `* * *`, hop 2 is `10.80.71.5`, and hop 4 shows two devices (load balancing).

**netstat / ss**: active connections and listening ports.
```bash
netstat -a            # old
ss -tulpn             # modern: TCP+UDP, listening, with process names (needs sudo for processes)
```
Columns: protocol, Recv-Q/Send-Q, local address, foreign address, state (`LISTEN`, `ESTABLISHED`, …). Example listeners: `5901` VNC, `sunrpc` (111), `http` (80), `ssh` (22). Knowing the port → service mapping speeds up diagnosis and enumeration.

**Most common problems**
- Connectivity issues
- DNS resolution issues
- Packet loss
- Poor performance

**Common causes**
- Misconfigured firewalls or routers
- Damaged cables/connections
- Wrong network settings
- Hardware failures
- Wrong DNS servers, DNS failure or bad DNS entries
- Congestion
- Outdated hardware
- Unpatched software/firmware and missing security controls

**Quick troubleshooting order**
1. `ip a`: does the interface have the right IP?
2. `ping <gateway>`: is the local network OK?
3. `ping 8.8.8.8`: is there internet (IP-level)?
4. `ping google.com`: does DNS work?
5. `traceroute`: where does it stop?
6. `ss -tulpn`: is the service actually listening?

### Hardening
Three tools that protect Linux systems from unauthorised access and compromise:

| Tool | Type | How it works | Pros | Cons |
|---|---|---|---|---|
| **SELinux** | MAC, built into the **kernel** | Policies define permissions for every process and file; limits the damage of a compromised service | Very fine-grained | Complex to configure |
| **AppArmor** | MAC, a **Linux Security Module** | Per-application **profiles** define which resources an app can access | Easier to manage | Less granular than SELinux |
| **TCP Wrappers** | Host-based network ACL | Allows or denies connections to services by **client IP** | Simple | Network-level only; no resource protection |

- Distro defaults: AppArmor on Ubuntu/Debian/SUSE, SELinux on RHEL/Fedora/CentOS.
- TCP Wrappers is a **legacy** tool: it was removed from RHEL 8 and later, and many modern services no longer support it. Firewalls (`ufw`, `nftables`) do the same job today.
- Similarities: all restrict access to resources and services, ship with most distros, and are configured with standard tools.

> [!tip] Practise on a personal VM and **take a snapshot before every change**: a wrong SELinux or AppArmor policy can lock you out.

### Practice exercises
**SELinux**
1. Install SELinux on your VM.
2. Configure SELinux to prevent a user from accessing a specific file.
3. Configure SELinux to allow a single user to access a specific network service but deny everyone else.
4. Configure SELinux to deny a specific user or group access to a specific network service.

**AppArmor**
5. Configure AppArmor to prevent a user from accessing a specific file.
6. Configure AppArmor to allow a single user to access a specific network service but deny everyone else.
7. Configure AppArmor to deny a specific user or group access to a specific network service.

**TCP Wrappers**
8. Allow access to a specific network service from a specific IP address.
9. Deny access to a specific network service from a specific IP address.
10. Allow access to a specific network service from a range of IP addresses.

### Exercise walkthrough (SELinux, AppArmor, TCP Wrappers)

> [!warning] Before you start
> Use a **throw-away VM and take a snapshot first**: a wrong SELinux/AppArmor setting can lock you out or stop the system booting. Keep a root console open while testing.

> [!info] Important concept: these tools confine **programs and labels, not usernames**
> The exercises say "a user", but SELinux and AppArmor don't filter by Linux username the way file permissions do. The walkthrough shows the closest correct method for each, and where a classic permission (DAC) is paired with them.
> Test users used below: `alice` and `bob` (`sudo useradd -m alice; sudo passwd alice`, same for `bob`).

#### Which distro for which exercise
| Exercises | Best VM | Why |
|---|---|---|
| 1–4 SELinux | **Rocky / Alma / Fedora** | SELinux is installed and enabled by default |
| 5–7 AppArmor | **Ubuntu** (or Debian/Kali) | AppArmor is the default there (Kali needs it switched on, see below) |
| 8–10 TCP Wrappers | Debian / Ubuntu / Kali | `tcpd` package available (see the note about support) |

SELinux *can* be installed on Debian/Ubuntu (`sudo apt install selinux-basics selinux-policy-default auditd && sudo selinux-activate`, then reboot and wait for the relabel, check with `check-selinux-installation`), but it is much less polished and can break boot. Prefer a RHEL-family VM.

> [!bug] Kali note
> Kali doesn't enable SELinux by default. `sestatus` shows `disabled` (or isn't installed), so `semanage` commands can run but **nothing is enforced** and you'll never see "Permission denied". Use Rocky/Fedora for exercises 1–4.

---

#### SELinux (exercises 1–4, on Rocky/Fedora)

**1. Install / enable SELinux**
```bash
sestatus                      # "SELinux status: enabled", "Current mode: enforcing"
getenforce                    # Enforcing / Permissive / Disabled
sudo dnf install -y policycoreutils policycoreutils-python-utils setools-console setroubleshoot-server
```
If it's `Disabled`: set `SELINUX=enforcing` in `/etc/selinux/config`, run `sudo touch /.autorelabel`, reboot (the relabel takes a while). Use `sudo setenforce 0` (permissive: log only) when debugging and `setenforce 1` to enforce again.

**Key vocabulary:** every process and file has a **context** `user:role:type:level`. The **type** decides what's allowed. See them with `ls -Z`, `ps -eZ`, `id -Z`. Denials are logged as **AVC** messages: `sudo ausearch -m avc -ts recent`.

> [!note] Why "a user" is tricky
> In the default (targeted) policy, normal Linux users are mapped to `unconfined_u`, so SELinux **doesn't restrict them**. To restrict a Linux user you first map them to a **confined SELinux user** with `semanage login`:
> | SELinux user | Network | su/sudo | Notes |
> |---|---|---|---|
> | `unconfined_u` | yes | yes | default, no restriction |
> | `staff_u` / `user_u` | yes | `staff_u` yes, `user_u` no | normal confined users |
> | `guest_u` | **no** | no | very restricted |
> View mappings: `sudo semanage login -l`, `sudo semanage user -l`.

**2. Prevent a user from accessing a specific file**
```bash
sudo useradd -m -s /bin/bash alice && sudo passwd alice   # create the users first (semanage fails with
sudo useradd -m -s /bin/bash bob && sudo passwd bob       # "Linux User alice does not exist" otherwise)
sudo semanage login -a -s user_u alice                 # make alice a confined user
echo "top secret" | sudo tee /home/alice/secret.txt
sudo chown alice:alice /home/alice/secret.txt          # DAC says alice may read it
sudo semanage fcontext -a -t shadow_t "/home/alice/secret.txt"
sudo restorecon -v /home/alice/secret.txt              # apply the new label (type shadow_t)
ls -Z /home/alice/secret.txt
```
Log in **as alice through a real login** (e.g. `ssh alice@localhost`, not `su`), because that's what sets the confined context. Check with `id -Z` (expect `user_u:user_r:user_t:s0`), then:
```bash
cat ~/secret.txt             # Permission denied, even though alice owns the file
```
Prove it was SELinux (as root): `sudo ausearch -m avc -ts recent`. Undo: `sudo semanage fcontext -d "/home/alice/secret.txt"; sudo restorecon -Fv /home/alice/secret.txt`.
*Why it works:* `user_t` isn't allowed to read `shadow_t` files, whoever owns them.

**3. Allow a single user to use the network, deny all others**
Make the **default** mapping the no-network user, then give one user the normal one:
```bash
sudo semanage login -m -s guest_u -r s0 __default__    # every user without a mapping → guest_u
sudo semanage login -a -s user_u alice                 # alice keeps network access
sudo semanage login -l                                 # root stays unconfined_u
```
Test with two real logins (`ssh alice@localhost`, `ssh bob@localhost`) while a service is running on the VM (e.g. `python3 -m http.server 8000`):
```bash
curl http://localhost:8000       # alice: works     bob (guest_u): fails
```
Check bob's denial in `sudo ausearch -m avc -ts recent`.
> [!caution] Keep `root` and your admin account mapped to `unconfined_u` (the default for `root`), or you can lock yourself out of sudo.

**4. Deny a specific user or group access to a network service**
```bash
sudo semanage login -a -s guest_u bob                  # one user → no network
sudo groupadd blocked && sudo usermod -aG blocked carol
sudo semanage login -a -s guest_u %blocked             # a whole group (note the %)
```
Verify as in exercise 3. To deny **one service** for everyone rather than one user, change the service's own domain, e.g. `sudo setsebool -P httpd_can_network_connect off` (stops Apache connecting outward), or label the port (`semanage port -l | grep http`).
Undo: `sudo semanage login -d bob`.

> [!tip] If a legitimate action is blocked
> `sudo sealert -a /var/log/audit/audit.log` explains the AVC and suggests a fix. `audit2allow -a` can generate a custom policy module, but read what it allows before loading it.

---

#### AppArmor (exercises 5–7, on Ubuntu/Debian/Kali)

**Setup**
```bash
sudo apt install apparmor apparmor-utils apparmor-profiles -y
sudo aa-status                          # lists loaded profiles and enforce/complain modes
```
Ubuntu has it on by default. On **Kali**, if `aa-status` says the module isn't loaded: add `apparmor=1 security=apparmor` to `GRUB_CMDLINE_LINUX` in `/etc/default/grub`, run `sudo update-grub`, reboot.

**Key vocabulary:** a **profile** is attached to a **program path** (`/etc/apparmor.d/`, file name = path with dots, e.g. `usr.local.bin.mycat`). Modes: **enforce** (block + log) and **complain** (log only). Handy: `aa-complain`, `aa-enforce`, `aa-genprof`, `aa-logprof`, `apparmor_parser -r <file>` (load/reload).
Denials appear as `apparmor="DENIED"` in `sudo journalctl -k` or `dmesg`. A `deny` rule always beats an allow rule.

**5. Prevent a user from accessing a specific file**
AppArmor confines a *program*, so confine the program that user would use to read the file:
```bash
sudo cp /usr/bin/cat /usr/local/bin/mycat
echo "top secret" > ~/secret.txt
sudo tee /etc/apparmor.d/usr.local.bin.mycat << 'EOF'
#include <tunables/global>
/usr/local/bin/mycat {
  #include <abstractions/base>
  /usr/local/bin/mycat mr,
  /** r,                          # may read everything…
  deny /home/*/secret.txt r,      # …except this file
}
EOF
sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.mycat
sudo aa-status | grep mycat
mycat /etc/hostname               # works
mycat ~/secret.txt                # Permission denied
sudo journalctl -k | grep -i 'apparmor="DENIED"' | tail -n 3
```
The restriction applies to anyone running `mycat`. To make it user-specific, combine it with permissions (exercises 6–7).

**6. Allow a single user to use a network service, deny all others**
Use a normal permission to pick *who* may run the program, and AppArmor to limit *what* it may do:
```bash
sudo groupadd netusers && sudo usermod -aG netusers alice
sudo cp /usr/bin/curl /usr/local/bin/mycurl
sudo chown root:netusers /usr/local/bin/mycurl
sudo chmod 750 /usr/local/bin/mycurl              # only the group can execute it
sudo tee /etc/apparmor.d/usr.local.bin.mycurl << 'EOF'
#include <tunables/global>
/usr/local/bin/mycurl {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  /usr/local/bin/mycurl mr,
  /usr/lib/x86_64-linux-gnu/** mr,
  /lib/x86_64-linux-gnu/** mr,
  network inet stream,
  network inet6 stream,
}
EOF
sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.mycurl
python3 -m http.server 8000 &                     # a service to talk to
```
Log in (or `su -`) as each user and test:
```bash
mycurl -I http://localhost:8000     # alice: HTTP/1.0 200 OK
mycurl -I http://localhost:8000     # bob: Permission denied (not in netusers)
```
Even for alice, `mycurl` can only open TCP sockets (no UDP, raw sockets, …).

**7. Deny a specific user or group access to a network service**
- **By user/group (permissions):**
  ```bash
  sudo setfacl -m u:bob:--- /usr/local/bin/mycurl       # one user
  sudo setfacl -m g:blocked:--- /usr/local/bin/mycurl   # a group
  ```
- **By program (AppArmor):** a profile that forbids networking for everyone who runs it:
  ```text
  /usr/local/bin/mycurl {
    #include <abstractions/base>
    /usr/local/bin/mycurl mr,
    /usr/lib/x86_64-linux-gnu/** mr,
    deny network,
  }
  ```
  Reload it and `mycurl -I http://localhost:8000` fails with a socket error; the denial appears in the kernel log.

> [!note] Check the profile syntax first with `sudo apparmor_parser -p /etc/apparmor.d/<file>`, and if something unexpected breaks, switch the profile to complain mode (`sudo aa-complain /usr/local/bin/mycurl`) and read the log.
> Remove a profile: `sudo apparmor_parser -R /etc/apparmor.d/usr.local.bin.mycurl && sudo rm /etc/apparmor.d/usr.local.bin.mycurl`.

---

#### TCP Wrappers (exercises 8–10)

**How it works:** a service built with **libwrap** (or started through `tcpd`) checks two files on each connection:
1. `/etc/hosts.allow`: if a rule matches, **allow**
2. otherwise `/etc/hosts.deny`: if a rule matches, **deny**
3. if nothing matches, **allow**

Rule format: `service_list : client_list` (e.g. `sshd : 192.168.1.50`). Wildcards: `ALL`. Client patterns: full IP, **prefix** with trailing dot (`192.168.1.`), or **netmask** form (`192.168.1.0/255.255.255.0`). The `hosts_access(5)` man page documents the **netmask** form; some builds also accept CIDR (`/24`), as the module's examples use. Check with `tcpdmatch`, and switch to the mask or prefix form if CIDR isn't accepted.

**Setup**
```bash
sudo apt install tcpd -y                      # provides tcpdmatch and the docs (man hosts_access)
ldd /usr/sbin/sshd | grep libwrap             # does sshd honour the files?
```
> [!warning] Support varies
> Many current OpenSSH builds no longer link libwrap, so the files may be ignored for `sshd`. If `ldd` shows nothing, you can still **test the rules** with `tcpdmatch` (below), or practise with a service started through `tcpd` (e.g. `openbsd-inetd`). Real-world blocking today is done with a firewall (`ufw`/`nftables`).

**8. Allow a service from one specific IP**
```text
# /etc/hosts.allow
sshd : 192.168.1.50
# /etc/hosts.deny
sshd : ALL
```
**9. Deny a service from one specific IP**
```text
# /etc/hosts.deny
sshd : 192.168.1.50
```
**10. Allow a service from a range of IPs**
```text
# /etc/hosts.allow
sshd : 192.168.1.0/255.255.255.0       # or the prefix form:  sshd : 192.168.1.
# /etc/hosts.deny
sshd : ALL
```
**Test without a second machine** (simulates a connection from that address):
```bash
tcpdmatch sshd 192.168.1.50       # access: granted / denied
tcpdmatch sshd 192.168.1.77
tcpdmatch sshd 10.0.0.5
```
`tcpdchk -v` checks the syntax of both files. With a libwrap-enabled service, try a real connection from another VM and read the denial in `journalctl -u ssh`.
> [!tip] Order matters: always put your **allow** rules in `hosts.allow` and the catch-all `ALL : ALL` in `hosts.deny`, and keep an existing session open while editing so you can undo a mistake.

---

## 25. Remote Desktop Protocols in Linux

Remote desktop protocols give administrators **graphical** remote access to install software, troubleshoot and manage systems.

| Protocol | Mainly used for | Default port |
|---|---|---|
| **RDP** (Remote Desktop Protocol) | Windows desktops | TCP **3389** |
| **VNC** (Virtual Network Computing, RFB protocol) | Linux (but cross-platform) | TCP **5900 + display number** (`:1` → 5901) |
| **X11 / X server** | Linux/Unix graphical apps | TCP **6000 + display number** (module: 6000–6010) |
| **XDMCP** | Remote X display manager login | UDP **177** |

> [!note] Analogy
> Different keys for different buildings: RDP is cut for Windows buildings, VNC is more of a universal key that is mostly used on Linux.

### X server (X11)
- The **X server** draws the graphical interface and talks to apps (X11 / "X" = the X Window System). It's part of almost every Ubuntu desktop install.
- Even on one machine, the desktop talks to the X server over local sockets or TCP. The first display `:0` uses TCP **6000**, `:1` uses 6001, and so on.
- **Network transparency:** apps run on one computer and are **displayed on another**. X11 renders on the **local** machine, which saves bandwidth and load on the remote machine compared with VNC/RDP (which render remotely and send the picture). If both machines are Unix/Linux, VNC/RDP aren't needed.
- **Big drawback:** the X11 protocol is **not encrypted**. The fix is to tunnel it through SSH.

**X11 forwarding over SSH**
1. On the server, in `/etc/ssh/sshd_config`: `X11Forwarding yes` (check with `grep X11Forwarding /etc/ssh/sshd_config`), then `sudo systemctl restart ssh`.
2. From the client, run an app remotely but see it locally:
```bash
ssh -X htb-student@10.129.23.11 /usr/bin/firefox
```
| Flag | Meaning |
|---|---|
| `-X` | X11 forwarding (restricted/"untrusted" mode) |
| `-Y` | Trusted forwarding (fewer restrictions, less safe; needed by some apps) |

On a Windows client you need a local X server (e.g. VcXsrv or MobaXterm). Firefox runs on the **server** but its window appears on your screen.

### X11 security
- Unencrypted by default, so look for **TCP 6000–6010** open on Linux targets.
- An **open X server** (no access control) lets an attacker on the network read what's on screen, capture windows and see keystrokes **without sniffing traffic**, using ordinary tools such as `xwd` (screenshots of X windows) and `xgrabsc`.
- Access control: `xhost +` allows **anyone** to connect, so never use it. Prefer SSH tunnelling and the cookie-based `xauth`.
- 2017: several Xorg-related vulnerabilities from weak, predictable or brute-forceable session keys could let an attacker run code in another user's Xorg session: **CVE-2017-2624, CVE-2017-2625, CVE-2017-2626**.

```bash
# Pentest checks (only on systems you're authorised to test)
nmap -p 6000-6010 <target>              # is an X server exposed?
xdpyinfo -display <target>:0            # does it accept connections without credentials?
xwd -root -display <target>:0 -out screen.xwd   # capture the screen (view with xwud -in screen.xwd)
```

### XDMCP
- **X Display Manager Control Protocol**: lets X terminals and Unix/Linux computers manage remote X sessions through the display manager, over **UDP 177**. It can redirect a whole desktop (KDE, GNOME) to a client.
- Server requirements: an X system with a GUI installed and configured.
- **Insecure**: not for environments needing high security. A **man-in-the-middle** attacker can intercept and impersonate either side, then run commands or read data.

### VNC
- A remote desktop sharing system based on the **RFB protocol**: view and control a desktop as if sitting in front of it. Very common for Linux; also used for screen sharing and remote support.
- **Two server types:**
  1. Shares the **real screen** (e.g. `x0vncserver`): keyboard/mouse still work locally, so it's good for user support.
  2. Provides separate **virtual sessions** per login (e.g. `vncserver`), like a terminal server.
- Ports: display 0 → TCP 5900, display 1 → 5901, display 2 → 5902, and so on.
- Tools: **TigerVNC**, **TightVNC**, **RealVNC**, **UltraVNC** (TeamViewer and RDP serve similar purposes).

> [!warning] Correction: plain VNC is **not** encrypted
> The module calls VNC "generally secure", but the default VNC password login (`VncAuth`) only protects the *login* with a weak challenge, uses at most **8 password characters**, and sends the **screen and keystrokes unencrypted**. Only some products or settings add encryption. **Always tunnel VNC through SSH**, as below.

**Set up a TigerVNC server (Ubuntu/Debian)**
GNOME is unstable over VNC, so use **XFCE4**.
```bash
sudo apt install xfce4 xfce4-goodies tigervnc-standalone-server -y
vncpasswd                                  # set the VNC password (view-only password: n)
```
This creates `~/.vnc/`. Add two files: `xstartup` (how the session starts) and `config` (settings).
```bash
cat > ~/.vnc/xstartup << 'EOT'
#!/bin/bash
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4
EOT

cat > ~/.vnc/config << 'EOT'
geometry=1920x1080
dpi=96
localhost=yes
EOT

chmod +x ~/.vnc/xstartup                   # must be executable
```
- The `#!/bin/bash` line must be the **first** line of the file (no blank line before it).
- `localhost=yes` (my addition) makes the server listen only on `127.0.0.1`, so it can only be reached through the SSH tunnel.
- The module's `xstartup` runs `/usr/bin/startxfce4` first, which blocks until the desktop closes, so the lines after it don't do anything. `exec startxfce4` is the simpler version.

**Start and manage**
```bash
vncserver                  # → "New 'linux:1 (htb-student)' desktop at :1"
vncserver -list            # sessions: DISPLAY :1, RFB port 5901, PID
vncserver -kill :1         # stop session :1
```

**Connect securely: SSH tunnel + viewer**
```bash
ssh -L 5901:127.0.0.1:5901 -N -f -l htb-student 10.129.14.130
xtightvncviewer localhost:5901
```
| Flag | Meaning |
|---|---|
| `-L 5901:127.0.0.1:5901` | Local port 5901 → port 5901 **as seen from the server** |
| `-N` | Don't run a remote command (tunnel only) |
| `-f` | Go to the background after login |
| `-l htb-student` | Login name |

Everything between viewer and server now travels inside the encrypted SSH connection. Alternative viewers: `vncviewer` / `xtigervncviewer` (`sudo apt install tigervnc-viewer`). Stop the tunnel with `pkill -f "ssh -L 5901"`. Tunnelling is covered in detail in the *Pivoting, Tunneling, and Port Forwarding* module.

### Quick comparison
| | X11 forwarding | XDMCP | VNC | RDP |
|---|---|---|---|---|
| Sends | Drawing commands (rendered locally) | Whole desktop | Rendered screen | Rendered screen |
| Encrypted by default | **No** (use `ssh -X`) | **No** | **No** (tunnel it) | Yes |
| Port | TCP 6000+ | UDP 177 | TCP 5900+ | TCP 3389 |
| Typical risk | Open X server → screen/keystroke capture | MITM, session hijack | Weak password, sniffing | Brute force, exposed 3389 |

---

## 26. Linux Security

Every system has some risk of intrusion, and an internet-facing web server hosting complex applications has more than most. Linux gets fewer viruses than Windows and has a smaller attack surface than an Active Directory domain-joined host, but the fundamentals still apply.

> [!quote] "Security is not a product but a process." The better administrators know their system, the better their protections.

### Core measures

#### 1. Keep the system up to date
```bash
sudo apt update && sudo apt dist-upgrade      # (same effect: apt full-upgrade)
```
- Some **kernel** updates are not applied automatically and need a **reboot**. Compare `uname -r` with the newest installed kernel, and look for `/var/run/reboot-required`.
- Automatic security updates: `sudo apt install unattended-upgrades`.

#### 2. Firewall (host or network level)
Use the Linux firewall (`iptables` / `nftables`, or the friendlier `ufw`) to restrict traffic into and out of the host.
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp                         # keep SSH reachable BEFORE enabling
sudo ufw enable
sudo ufw status verbose
```

#### 3. Secure SSH
In `/etc/ssh/sshd_config`:
```text
PasswordAuthentication no      # keys only
PermitRootLogin no             # no direct root login
PubkeyAuthentication yes
```
Then `sudo systemctl restart ssh`.
> [!warning] Set up and test **key login first**, in a second terminal, before disabling passwords, or you can lock yourself out. Avoid administering the system as root: log in as a normal user and use `sudo`.

#### 4. Least privilege and sudoers
Give users only what they need. If a user must run **one** command as root, allow **that command** in sudoers instead of full `sudo` rights:
```bash
sudo visudo -f /etc/sudoers.d/alice       # always edit with visudo (it checks the syntax)
```
```text
alice ALL=(root) /usr/bin/systemctl restart apache2
```
Check what a user can do with `sudo -l`.
> [!caution] Many binaries can spawn a shell (see the GTFOBins note in Section 14 on SUID), so allowing something like `vim`, `less` or `find` via sudo is effectively giving full root.

#### 5. fail2ban
Counts failed logins and, after the maximum, **bans the offending host** (e.g. with a firewall rule) for a set time.
```bash
sudo apt install fail2ban -y
sudo tee /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled  = true
maxretry = 5
findtime = 10m
bantime  = 1h
EOF
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd                         # banned IPs
sudo fail2ban-client set sshd unbanip <ip>               # un-ban
```

#### 6. Audit regularly for privilege-escalation paths
| Check | Command |
|---|---|
| Old kernel / OS | `uname -r`, `lsb_release -a` |
| SUID / SGID binaries | `find / -perm -4000 -type f 2>/dev/null` and `-perm -2000` |
| World-writable files | `find / -type f -perm -0002 2>/dev/null` |
| World-writable directories | `find / -type d -perm -0002 ! -perm -1000 2>/dev/null` (no sticky bit) |
| Cron jobs | `cat /etc/crontab; ls -la /etc/cron.*; crontab -l` |
| Listening services | `ss -tulpn`, `systemctl list-units --type=service` |
| Sudo rights | `sudo -l` |

Look at user permission mistakes, misconfigured cron jobs (e.g. root job running a script anyone can edit) and misconfigured services.

#### 7. SELinux / AppArmor
Kernel security modules for access control policies (see Section 24 for the comparison and exercises). In **SELinux**, every process, file, directory and object has a **label**; policy rules control access between labels and are enforced by the kernel. It can be very granular, e.g. controlling who may **append to** or **move** a file.

#### 8. Security tools
| Tool | Purpose | Example |
|---|---|---|
| **Snort** | Network intrusion detection / prevention (IDS/IPS) | rules-based traffic inspection |
| **chkrootkit** | Scans for known rootkits | `sudo chkrootkit` |
| **rkhunter** | Rootkit, backdoor and local-exploit scanner | `sudo rkhunter --check` |
| **Lynis** | Security auditing and hardening suggestions | `sudo lynis audit system` |

### Hardening checklist (the module's list, with how to do each)
| Item | How |
|---|---|
| Remove/disable unnecessary services and software | `sudo systemctl disable --now <service>`, `sudo apt purge <pkg>` |
| Remove services using **unencrypted authentication** | Replace telnet, FTP, rsh, plain VNC with SSH, SFTP, SSH-tunnelled VNC |
| NTP enabled, syslog running | `timedatectl` (look for `System clock synchronized: yes`, enable with `timedatectl set-ntp true`), `systemctl status rsyslog` |
| One account per user (no shared logins) | `sudo adduser <name>` per person |
| Strong passwords | `sudo apt install libpam-pwquality`, then set `minlen`, etc. in `/etc/security/pwquality.conf` |
| Password aging | `sudo chage -M 90 -m 7 -W 14 <user>`, view with `chage -l <user>` |
| Restrict reuse of old passwords | `remember=5` on the `pam_pwhistory` line in `/etc/pam.d/common-password` |
| Lock accounts after failed logins | `pam_faillock` (settings in `/etc/security/faillock.conf`, e.g. `deny=5`, `unlock_time=900`), or fail2ban for network logins. Unlock with `faillock --user <user> --reset` |
| Disable unwanted SUID/SGID binaries | find them (above), then `sudo chmod u-s <file>` / `g-s` |

> [!warning] PAM files (`/etc/pam.d/*`) are easy to break. Keep a root shell open while editing, and test with a second login before closing it. The list is **not complete**: security is an ongoing process.

### TCP Wrappers
A host-based access control tool: it allows or denies access to **services** based on the client's **IP address or hostname**. When a client connects, the service consults two files:
- `/etc/hosts.allow`: who **is** allowed
- `/etc/hosts.deny`: who is **not** allowed

**`/etc/hosts.allow`** (module example)
```text
# Allow access to SSH from the local network
sshd : 10.129.14.0/24

# Allow access to FTP from a specific host
ftpd : 10.129.14.10

# Allow access to Telnet from any host in the inlanefreight.local domain
telnetd : .inlanefreight.local
```
**`/etc/hosts.deny`** (module example)
```text
# Deny access to all services from any host in the inlanefreight.com domain
ALL : .inlanefreight.com

# Deny access to SSH from a specific host
sshd : 10.129.22.22

# Deny access to FTP from hosts in 10.129.22.0 – 10.129.22.255
ftpd : 10.129.22.0/24
```
**How rules are evaluated**
1. `hosts.allow` is read first, **top to bottom**. The **first** matching rule wins, and the connection is **allowed**.
2. Otherwise `hosts.deny` is read the same way. The first match **denies**.
3. If **no** rule matches, access is **allowed**.
4. Order inside each file matters.

| Pattern | Matches |
|---|---|
| `ALL` | Every service or every host |
| `10.129.14.10` | One IP address |
| `10.129.14.` | Any address starting with that prefix |
| `10.129.14.0/24` | A network (CIDR form; accepted by some builds, see below) |
| `10.129.14.0/255.255.255.0` | A network (netmask form, as in `hosts_access(5)`) |
| `.inlanefreight.local` | Any host in that domain (leading dot) |

> [!note] Things to remember
> - **Not a firewall replacement**: it controls access to **services**, not to **ports**, and only for programs built with libwrap (or started via `tcpd`).
> - The **service name** is the daemon's name as it identifies itself (e.g. `sshd`, `vsftpd`), which may differ from names like `ftpd` in the example.
> - **Hostname/domain rules depend on DNS** (reverse lookups), which can be spoofed or fail, so IP rules are more reliable.
> - **CIDR (`/24`)** is used in the module, but `hosts_access(5)` documents the netmask form. If a CIDR rule doesn't work on your system, switch to the mask or prefix form.
> - It's a **legacy** tool, and many current distributions and daemons no longer support it. Test rules with `tcpdmatch <service> <ip>` (see the Section 24 walkthrough).

---

## 27. Firewall Setup

A **firewall** controls and monitors traffic between network segments (internal/external, different zones) and filters it by rules: addresses, protocols, ports and more. It protects against unauthorised access, malicious traffic and other threats, and supports the **confidentiality, integrity and availability** of resources.

### History and architecture
- Firewall tools on Linux evolved as **ipfwadm → ipchains → iptables** (introduced with the **Linux 2.4** kernel, 2000), and today **nftables**.
- All of them sit on **Netfilter**, the packet-filtering framework inside the kernel. Netfilter provides **hooks** where packets can be inspected, modified, accepted or dropped as they pass through. `iptables` is the command-line front end that sets the rules.
- `iptables` can protect against DoS attempts, port scans and intrusion attempts.

| Tool | Notes |
|---|---|
| **iptables** | The classic standard; powerful but low-level |
| **nftables** | Modern replacement with a newer syntax and better performance; **not** syntax-compatible with iptables (migration takes effort) |
| **ufw** | "Uncomplicated Firewall": simple front end built on top of iptables/nftables |
| **firewalld** | Dynamic firewall with **zones** and **services** (default on RHEL/Fedora): `firewall-cmd` |

> [!note] On current Debian, Ubuntu, Kali and RHEL, the `iptables` command is often a compatibility layer that actually writes **nftables** rules. `iptables --version` shows `(nf_tables)` or `(legacy)`. The commands below work the same either way.

### iptables components
| Component | Description |
|---|---|
| **Tables** | Organise rules by purpose |
| **Chains** | Lists of rules applied to a type of traffic |
| **Rules** | Criteria + action for matching packets |
| **Matches** | The criteria (IP, port, protocol, state, …) |
| **Targets** | The action (accept, drop, modify, …) |

### Tables
| Table | Purpose | Built-in chains |
|---|---|---|
| **filter** (default) | Allow/block traffic by IP, port, protocol | `INPUT`, `OUTPUT`, `FORWARD` |
| **nat** | Rewrite source/destination addresses | `PREROUTING`, `POSTROUTING` (and `OUTPUT`) |
| **mangle** | Modify packet header fields | `PREROUTING`, `OUTPUT`, `INPUT`, `FORWARD`, `POSTROUTING` |
| **raw** | Special processing, e.g. exempt packets from connection tracking | `PREROUTING`, `OUTPUT` |

(A fifth table, `security`, exists for SELinux-related marking.)

### Chains
- **Built-in chains** are created automatically with their table.
  - `INPUT`: packets **to** this host
  - `OUTPUT`: packets **from** this host
  - `FORWARD`: packets **routed through** this host (it must have IP forwarding on)
  - `PREROUTING`: as packets arrive, **before** routing (used to change the *destination*, e.g. DNAT)
  - `POSTROUTING`: just before packets leave, **after** routing (used to change the *source*, e.g. SNAT/MASQUERADE)
- **User-defined chains** group related rules (e.g. all rules for web servers or for port 80). A built-in chain **jumps** to them with `-j <chain>`.

```text
 incoming ─► PREROUTING ─► [routing decision] ─┬─► INPUT ─► local process
                                               └─► FORWARD ─► POSTROUTING ─► out
 local process ─► OUTPUT ─► POSTROUTING ─► out
```

### Rules and targets
Rules are added to a chain with `-A` (append), and changed or removed with other options. Each rule = **matches** + a **target**. Rules are checked **top to bottom**, and the **first match with a final target wins**. If nothing matches, the chain's **default policy** (`ACCEPT` unless changed) applies.

| Target | Description |
|---|---|
| `ACCEPT` | Let the packet through |
| `DROP` | Silently discard it (the sender just times out) |
| `REJECT` | Discard it and send an error back (`--reject-with tcp-reset`, ICMP unreachable, …) |
| `LOG` | Write packet info to the system log (does **not** stop processing; add a DROP/ACCEPT after it) |
| `SNAT` | Change the **source** address (nat, POSTROUTING) |
| `DNAT` | Change the **destination** address, e.g. port forwarding (nat, PREROUTING) |
| `MASQUERADE` | Like SNAT, for dynamic public IPs |
| `REDIRECT` | Redirect to another port on this host (nat) |
| `MARK` | Set a Netfilter mark for routing/other purposes (mangle) |
| `RETURN` | Stop processing this chain and go back to the calling chain |

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # allow SSH in
```
Reads as: **append** (`-A`) to chain `INPUT` a rule matching **protocol** TCP (`-p tcp`) and **destination port** 22 (`--dport 22`), then **jump** (`-j`) to target `ACCEPT`.

### Matches
| Match | Description |
|---|---|
| `-p` / `--protocol` | Protocol: `tcp`, `udp`, `icmp` |
| `--dport` / `--sport` | Destination / source port (needs `-p tcp` or `-p udp`) |
| `-s` / `--source` | Source IP or network (`10.0.0.0/24`) |
| `-d` / `--destination` | Destination IP or network |
| `-i` / `-o` | Incoming / outgoing interface (`-i eth0`, `-i lo`) |
| `-m state --state` | Connection state: `NEW`, `ESTABLISHED`, `RELATED` (older; prefer conntrack) |
| `-m conntrack --ctstate` | Same, via connection tracking (modern) |
| `-m multiport --dports 80,443` | Several ports or ranges in one rule |
| `-m tcp` / `-m udp` | Extra TCP/UDP options (e.g. `--tcp-flags`) |
| `-m string --string "x" --algo bm` | Packets containing a string |
| `-m limit --limit 5/min` | Rate limit |
| `-m mark` | Match the Netfilter mark |
| `-m mac --mac-source` | Source MAC address |
| `-m iprange --src-range a-b` | IP address range |

`-m` loads a match module. Example: `sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT`.

### Command cheat sheet
| Task | Command |
|---|---|
| Append a rule | `iptables -A INPUT <match> -j <target>` |
| Insert at the top (position 1) | `iptables -I INPUT 1 <match> -j <target>` |
| Delete by specification | `iptables -D INPUT <same match> -j <target>` |
| Delete by number | `iptables -D INPUT 3` |
| List rules (numeric, counters, numbers) | `iptables -L -n -v --line-numbers` |
| Show as commands | `iptables -S` |
| Other table | `iptables -t nat -L -n -v` |
| Default policy | `iptables -P INPUT DROP` |
| Flush a chain / everything | `iptables -F INPUT` / `iptables -F` |
| New / delete user chain | `iptables -N NAME` / `iptables -X NAME` |
| Save / restore | `iptables-save > rules.v4` / `iptables-restore < rules.v4` |

> [!warning] Safe practice
> - **Rules are not persistent**: they disappear at reboot. Persist them with `sudo apt install iptables-persistent` then `sudo netfilter-persistent save` (Debian/Ubuntu/Kali).
> - **Order matters**: use `-I` to put an exception **above** an existing rule; `-A` goes to the end.
> - Before `iptables -P INPUT DROP`, first allow **established connections** and **SSH**, or you'll cut yourself off:
>   ```bash
>   sudo iptables -A INPUT -i lo -j ACCEPT
>   sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
>   sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
>   ```
> - Use `-n` when listing, so it doesn't do slow DNS lookups.
> - **Reset to a clean state** if something goes wrong:
>   ```bash
>   sudo iptables -P INPUT ACCEPT; sudo iptables -P FORWARD ACCEPT; sudo iptables -P OUTPUT ACCEPT
>   sudo iptables -F; sudo iptables -X; sudo iptables -t nat -F; sudo iptables -t mangle -F
>   ```

### Practice exercises: worked solutions
> [!info] Setup
> Use a VM (snapshot first). Start a test web server, and use `curl` with a timeout so a DROP doesn't hang the terminal.
> ```bash
> python3 -m http.server 8080 &              # web server on TCP/8080
> curl -sI --max-time 3 http://localhost:8080 | head -n 1     # expect: HTTP/1.0 200 OK
> ```
> Traffic to `localhost` goes through the `lo` interface and **does** pass through the INPUT chain, so rules for `8080` affect these tests. For a more realistic test, `curl` from another machine.

**1. Block incoming traffic on TCP/8080**
```bash
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j DROP
curl -sI --max-time 3 http://localhost:8080     # times out (no reply)
```
Try `-j REJECT` instead: `curl` fails **immediately** with "connection refused" (DROP = silent, REJECT = tells the sender).

**2. Allow incoming traffic on TCP/8080 again**
```bash
sudo iptables -D INPUT -p tcp --dport 8080 -j DROP               # remove the block, or…
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT           # …put an ACCEPT above it
curl -sI --max-time 3 http://localhost:8080 | head -n 1
```

**3. Block traffic from a specific IP**
```bash
sudo iptables -I INPUT 1 -s 127.0.0.1 -j DROP        # lab test: block your own loopback address
ping -c 2 -W 1 127.0.0.1                             # 100% packet loss
# real use: -s 192.168.1.50   (or a network: -s 192.168.1.0/24)
```

**4. Allow traffic from a specific IP (and block everyone else)**
```bash
sudo iptables -I INPUT 1 -s 192.168.1.50 -p tcp --dport 8080 -j ACCEPT   # the exception first…
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP                       # …then the general block
```
Rule order is the whole trick: the ACCEPT for that IP is matched before the DROP.

**5. Block traffic based on protocol**
```bash
sudo iptables -A INPUT -p icmp -j DROP         # block ping
ping -c 2 -W 1 127.0.0.1                       # no replies
sudo iptables -A INPUT -p udp -j DROP          # block UDP (see warning)
```
> [!caution] Blocking all UDP also blocks **DNS replies**, so name resolution stops working. Test briefly, then delete the rule.

**6. Allow traffic based on protocol**
```bash
sudo iptables -I INPUT 1 -p icmp -j ACCEPT                         # all ICMP, or only pings:
sudo iptables -I INPUT 1 -p icmp --icmp-type echo-request -j ACCEPT
ping -c 2 127.0.0.1                                                # works again
```

**7. Create a new chain**
```bash
sudo iptables -N WEB_RULES
sudo iptables -A WEB_RULES -p tcp --dport 80 -j ACCEPT
sudo iptables -A WEB_RULES -p tcp --dport 8080 -j ACCEPT
sudo iptables -L WEB_RULES -n -v
```

**8. Send traffic to a specific chain**
(Here "forward" means **jump** with `-j`, not the `FORWARD` chain.)
```bash
sudo iptables -A INPUT -p tcp -m multiport --dports 80,8080 -j WEB_RULES
curl -sI --max-time 3 http://localhost:8080 | head -n 1
sudo iptables -L -n -v --line-numbers        # the pkts counter on WEB_RULES' rule increases
```
If no rule in the user chain matches, processing **returns** to the next rule in the calling chain.

**9. Delete a specific rule**
```bash
sudo iptables -L INPUT -n --line-numbers     # find the rule number
sudo iptables -D INPUT 2                     # delete by number (numbers shift after each delete!)
sudo iptables -D INPUT -s 127.0.0.1 -j DROP  # or delete by exact specification
```
To remove a user chain: delete the rule that jumps to it, then `sudo iptables -F WEB_RULES && sudo iptables -X WEB_RULES`.

**10. List all existing rules**
```bash
sudo iptables -L -n -v --line-numbers      # filter table
sudo iptables -S                           # as iptables commands
sudo iptables -t nat -L -n -v              # nat table (also mangle, raw)
sudo iptables-save                         # every table, in restore format
```
Clean up afterwards: stop the test server (`kill %1`) and run the **reset** commands above.

---

## 28. System Logs

System logs are files recording what the system and its applications are doing. They are used for **monitoring and troubleshooting** and are a goldmine for **security**: unauthorised logins, attempted attacks, clear-text credentials, unusual file access, failing services and so on.

**Why pentesters read logs**
- On a target: find system behaviour, user activity, network activity, weak points and **credentials leaked in logs**.
- After a test: check whether your actions **triggered alerts or warnings**, then refine your approach (and, in a report, show the client what was and wasn't detected).

**Why admins configure them properly:** set suitable **log levels**, **rotate** logs so they don't grow without limit, **protect** them from unauthorised access, and **review** them regularly.

> [!warning] On an authorised test, don't delete or edit logs unless the scope explicitly allows it. Tampering with logs destroys evidence and is normally out of bounds.

### Types of logs
| Type | Content | Default file (Debian/Ubuntu) | RHEL/CentOS |
|---|---|---|---|
| **Kernel** | Drivers, hardware, system calls, kernel events | `/var/log/kern.log` | `/var/log/messages` |
| **System** | Service start/stop, reboots, cron, general events | `/var/log/syslog` | `/var/log/messages` |
| **Authentication** | Logins (success/failure), `sudo`, `su`, SSH | `/var/log/auth.log` | `/var/log/secure` |
| **Application** | Per-application activity (web server, database, …) | e.g. `/var/log/apache2/error.log` | e.g. `/var/log/httpd/` |
| **Security** | Output of security tools (fail2ban, ufw, auditd) | see below | see below |

> [!note] Where are my logs?
> Newer systems may log **only to journald** (binary logs read with `journalctl`). If `/var/log/syslog` or `/var/log/auth.log` doesn't exist (recent Kali images, for example, may not install `rsyslog`), use `sudo journalctl` instead, or `sudo apt install rsyslog` to get the classic text files back.

### Kernel logs: `/var/log/kern.log`
Hardware drivers, kernel events, crashes and resource limits. Useful to spot **old or vulnerable drivers**, events that lead to denial of service, and suspicious system calls that may point to malware. Quick view: `dmesg` or `sudo journalctl -k`.

### System log: `/var/log/syslog`
Service starts/stops, login attempts, reboots and other system-level events. Good for detecting access, failing services and availability problems.
```text
Feb 28 2023 15:00:01 server CRON[2715]: (root) CMD (/usr/local/bin/backup.sh)
Feb 28 2023 15:04:22 server sshd[3010]: Failed password for htb-student from 10.14.15.2 port 50223 ssh2
Feb 28 2023 15:05:02 server kernel: [  138.303596] ata3.00: exception Emask 0x0 SAct 0x0 SErr 0x0 action 0x6 frozen
Feb 28 2023 15:06:43 server apache2[2904]: 127.0.0.1 - - [28/Feb/2023:15:06:43 +0000] "GET /index.html HTTP/1.1" 200 13484 ...
Feb 28 2023 15:07:19 server sshd[3010]: Accepted password for htb-student from 10.14.15.2 port 50223 ssh2
Feb 28 2023 15:09:54 server kernel: [  367.543975] EXT4-fs (sda1): re-mounted. Opts: errors=remount-ro
Feb 28 2023 15:12:07 server systemd[1]: Started Clean PHP session files.
```
Format: `timestamp  host  process[PID]: message`. In this example: a root **cron** job runs a backup script, a **failed** SSH password followed by a **successful** one for the same user (possible guessing), a disk error from the kernel, and an Apache request.

### Authentication log: `/var/log/auth.log`
Focuses on **authentication**, so it is more valuable than syslog for finding threats (SSH, `sudo`, `su`, PAM, cron sessions).
```text
sshd[5678]: Accepted publickey for admin from 10.14.15.2 port 43210 ssh2: RSA SHA256:+KjEz...
sudo:   admin : TTY=pts/1 ; PWD=/home/admin ; USER=root ; COMMAND=/bin/bash
sudo:   admin : TTY=pts/1 ; PWD=/home/admin ; USER=root ; COMMAND=/usr/bin/apt-get install netcat-traditional
sshd[5678]: Disconnected from 10.14.15.2 port 43210 [preauth]
kernel: [  778.941871] firewall: unexpected traffic allowed on port 22
auditd[9876]: Audit daemon started successfully
systemd-logind[1234]: New session 4321 of user admin.
CRON[2345]: pam_unix(cron:session): session opened for user root by (uid=0)
CRON[2345]: pam_unix(cron:session): session closed for user root
```
Reading it:
- `Accepted publickey for admin` = a **key-based SSH login** succeeded.
- `sudo: admin ... COMMAND=/bin/bash` = admin **got a root shell** through sudo (so admin has sudo rights). Every `sudo` command is logged with user, working directory, target user and full command.
- The kernel line says unexpected traffic was allowed on port 22, which is worth investigating.
- `systemd-logind` = a new login session; `pam_unix(cron:...)` = cron opening/closing a root session.

> [!tip] Credentials in logs
> Users sometimes type their **password into the username field** (`Failed password for invalid user MyP@ss…`), and `sudo` lines or application logs may contain secrets passed on the command line. Always search logs for these.

### Application logs
Each application writes its own files, e.g. `/var/log/apache2/error.log` or `/var/log/mysql/error.log`. They show how the app handles data, and reveal misconfigurations, injection attempts, unauthorised access and data exfiltration.
- **Access logs**: record requests and activity (who accessed what, from where).
- **Audit logs**: record security-relevant changes (system config files, attempts to modify files or settings). On Linux the audit daemon writes `/var/log/audit/audit.log` (`auditd`).

Example access-log style entry:
```text
2023-03-07T10:15:23+00:00 servername privileged.sh: htb-student accessed /root/hidden/api-keys.txt
```
The user `htb-student` used the script `privileged.sh` to open `/root/hidden/api-keys.txt`.

**Default locations**
| Service | Log location |
|---|---|
| Apache | `/var/log/apache2/access.log` (and `error.log`); RHEL: `/var/log/httpd/` |
| Nginx | `/var/log/nginx/access.log` (and `error.log`) |
| OpenSSH | `/var/log/auth.log` (Ubuntu/Debian), `/var/log/secure` (CentOS/RHEL) |
| MySQL / MariaDB | `/var/log/mysql/error.log` (the general query log `mysql.log` is **off by default**) |
| PostgreSQL | `/var/log/postgresql/postgresql-<version>-main.log` |
| systemd (journald) | `/var/log/journal/` (binary, read with `journalctl`) |

### Security logs
| Tool | Log |
|---|---|
| fail2ban | `/var/log/fail2ban.log` (failed logins and bans) |
| ufw firewall | `/var/log/ufw.log` (needs `sudo ufw logging on`) |
| auditd | `/var/log/audit/audit.log` |
| General security events | `/var/log/syslog`, `/var/log/auth.log` |

### Other useful logs and records
| File / command | Content |
|---|---|
| `last` (`/var/log/wtmp`) | History of logins and reboots |
| `lastb` (`/var/log/btmp`), needs root | **Failed** login attempts |
| `lastlog` | Last login of every user |
| `/var/log/dpkg.log`, `/var/log/apt/history.log` | Packages installed or removed |
| `~/.bash_history` | Shell commands a user ran |
| `/var/log/boot.log`, `dmesg` | Boot messages |

> [!note] Many logs are readable only by root and members of the **`adm`** group (see the `id` notes in Section 6). A user in `adm` can read logs that may expose sensitive data.

### Reading and analysing logs
Use any log viewer from the desktop, or the command line: `tail`, `grep`, `sed`, `awk`, `less`, `sort`, `uniq`.
```bash
sudo tail -f /var/log/auth.log                          # follow live
sudo grep "Failed password" /var/log/auth.log           # failed SSH logins
sudo grep "Accepted" /var/log/auth.log                  # successful logins
sudo grep "COMMAND" /var/log/auth.log                   # everything run through sudo
sudo grep CRON /var/log/syslog                          # cron activity
sudo zgrep "Failed password" /var/log/auth.log*         # include rotated/compressed (.gz) logs
```
**Count failed SSH logins per IP** (brute-force detection):
```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
```
(The IP is the 4th field from the end, `... from <IP> port <N> ssh2`.)

**Busiest IPs and scanners in a web log:**
```bash
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head      # top requesters
grep -E '" (404|403|401) ' /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head
```
**Login history:** `last -n 10`, `sudo lastb -n 10`, `lastlog`.

### `journalctl` (systemd journal)
| Task | Command |
|---|---|
| Everything, newest last | `sudo journalctl` |
| Follow live | `sudo journalctl -f` |
| One service | `sudo journalctl -u ssh.service` (`ssh` or `sshd`, depending on the distro) |
| Since the last boot | `sudo journalctl -b` |
| Kernel messages | `sudo journalctl -k` |
| Only errors and worse | `sudo journalctl -p err` |
| Time window | `sudo journalctl --since "1 hour ago"` / `--since "2023-02-28" --until "2023-03-01"` |
| By program | `sudo journalctl _COMM=sudo` |
| Without paging | `--no-pager` |

### Configuring logging
- **rsyslog**: `/etc/rsyslog.conf` and `/etc/rsyslog.d/`. Rules are `facility.priority  destination`, e.g. `auth,authpriv.*  /var/log/auth.log`. Priorities run from `emerg` (0, worst) through `err` (3) and `info` (6) to `debug` (7). **Log levels** decide how much detail is recorded.
- **logrotate**: `/etc/logrotate.conf` and `/etc/logrotate.d/` rotate, compress and delete old logs (this is why you see `auth.log.1`, `auth.log.2.gz`).
- **Protect logs**: keep restrictive permissions, ideally forward logs to a **remote log server** (rsyslog, ELK stack) so an attacker who gets root can't erase the only copy, and keep NTP running so timestamps are reliable.

---

## 29. Solaris

> [!abstract] Overview
> **Solaris** is a Unix (System V-based) operating system created by Sun Microsystems and now owned by **Oracle**. It targets enterprise workloads such as databases, banking, telecoms and large data centres, where stability, scalability, fault tolerance and security matter most. Linux and Solaris share a Unix heritage, so many commands look alike, but the tooling for packages, services, filesystems and privileges differs.

### 29.1 What Solaris is known for

| Feature | What it does |
|---|---|
| **ZFS** | Combined filesystem and volume manager with snapshots, compression, checksums and pools. It was created for Solaris. |
| **SMF** (Service Management Facility) | Starts, monitors and restarts services and tracks their dependencies. |
| **IPS** (Image Packaging System) | Package manager in Solaris 11 (`pkg`). |
| **Zones** | OS-level virtualisation (similar to containers). |
| **RBAC** | Role-Based Access Control with fine-grained privileges. |
| **Oracle VM Server for SPARC** | Hypervisor for SPARC hardware. |
| **DTrace** | Dynamic tracing of the kernel and applications. |

> [!warning] Corrections to the module text
> - **ZFS is not typical of Linux.** The module says Linux distributions commonly use ZFS. In fact ZFS originated in Solaris. Most Linux distros default to **ext4**, **XFS** or **Btrfs**, and ZFS on Linux (OpenZFS) is an optional add-on, for example on Ubuntu.
> - **Open source:** modern Solaris (11.x) is proprietary. Sun released **OpenSolaris** until 2010, and its descendants (**illumos**, **OpenIndiana**, **OmniOS**) are open source.
> - **Vendor claims:** the module says Amazon, IBM and Dell use Solaris in their products. I couldn't verify that, so treat it as marketing.

### 29.2 Linux vs Solaris at a glance

| Area | Linux (Ubuntu/Debian) | Solaris |
|---|---|---|
| Licence | Mostly open source | Proprietary (Oracle) |
| Default filesystem | ext4 / XFS / Btrfs | **ZFS** (UFS on older releases) |
| Service manager | systemd (`systemctl`) | **SMF** (`svcs`, `svcadm`, `svccfg`) |
| Package manager | APT (`apt`, `dpkg`) | **IPS** (`pkg`) on 11, SVR4 (`pkgadd`) on 10 |
| Privilege model | `sudo`, root | **RBAC** (roles, profiles, `pfexec`), `sudo` since 11 |
| System info | `uname -a` | `showrev -a`, `uname -a` |
| Open files of a process | `lsof` | `pfiles` |
| Syscall tracing | `strace` | `truss` |
| NFS share | `/etc/exports`, `exportfs` | `share`, `/etc/dfs/dfstab` |
| Hardware | Mostly x86/ARM | SPARC and x86 |

The module groups the differences into **filesystem**, **process management**, **package management**, **kernel and hardware support**, **system monitoring** and **security**.

### 29.3 Directory layout

Largely the same as Linux, with a few Solaris-specific entries.

| Directory | Description |
|---|---|
| `/` | Root of the file system |
| `/bin` | Essential binaries (a symlink to `/usr/bin` on Solaris 11) |
| `/boot` | Boot loader and kernel images |
| `/dev` | Device files |
| `/etc` | System configuration and authentication data |
| `/home` | Users' home directories |
| `/kernel` | Kernel modules and kernel-related files (**Solaris-specific**) |
| `/lib` | Libraries for `/bin` and `/sbin` |
| `/lost+found` | Files recovered by filesystem check and repair |
| `/mnt` | Temporary mount points |
| `/opt` | Optional software packages |
| `/proc` | Process and kernel status as files |
| `/sbin` | Administration binaries |
| `/tmp` | Temporary files |
| `/usr` | System-wide read-only data and programs |
| `/var` | Variable data (logs, mail and print spools) |

### 29.4 Command comparisons

#### System information

```shell-session
# Ubuntu
$ uname -a
Linux ubuntu 5.4.0-1045 #48-Ubuntu SMP Fri Jan 15 10:47:29 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

# Solaris
$ showrev -a
Hostname: solaris
Kernel architecture: sun4u
OS version: Solaris 10 8/07 s10s_u4wos_12b SPARC
Application architecture: sparc
Hardware provider: Sun_Microsystems
Domain: sun.com
Kernel version: SunOS 5.10 Generic_139555-08
```

`showrev` also reports patch level and hardware provider. Other useful Solaris commands:

```bash
uname -a           # works on Solaris too (kernel shows as SunOS)
cat /etc/release   # exact Solaris release
prtconf            # hardware configuration
```

#### Installing packages

```bash
# Ubuntu
sudo apt-get install apache2

# Solaris 10 (SVR4 packages)
pkgadd -d SUNWapchr

# Solaris 11 (IPS)
pkg install web/server/apache-24
pkg search apache      # search packages
pkg list               # installed packages
```

> [!note] Package managers and `sudo`
> The module calls the Solaris tool "Solaris Package Manager (SPM)". The classic SVR4 tools are `pkgadd`, `pkgrm` and `pkginfo`, and **Solaris 11 replaced them with IPS (`pkg`)**. The module also notes that no `sudo` is needed in its example, because Solaris traditionally used **RBAC** to give users granular privileges, and `sudo` has been supported since Solaris 11.

```bash
roles              # roles assigned to the current user
profiles           # rights profiles
auths              # authorisations
pfexec <command>   # run a command with the privileges of a rights profile
```

#### Permission management

`chmod` works the same way on both systems:

```bash
chmod 700 filename
```

Finding **SUID** files:

```bash
# Linux
find / -perm -4000 2>/dev/null

# Solaris
find / -perm -4000 2>/dev/null
```

> [!warning] The `-` in `-perm -4000` is not a Solaris/Linux difference
> The module shows `-perm 4000` for Linux and `-perm -4000` for Solaris and says this is because "Solaris uses a different permission system". That is not accurate.
> - `-perm 4000` matches files whose mode is **exactly** `4000`, so it misses almost every real SUID binary (for example `4755`).
> - `-perm -4000` matches files that have **at least** the SUID bit set.
>
> Use `-perm -4000` on both systems (GNU `find` also accepts `-perm /4000`). The `2>/dev/null` hides permission-denied noise.

#### NFS

```bash
# Share a directory (Solaris)
share -F nfs -o rw /export/home

# Mount a share
mount -F nfs 10.129.15.122:/nfs_share /mnt/local   # Solaris
sudo mount -t nfs 10.129.15.122:/nfs_share /mnt/local   # Linux
```

Shares are made permanent in `/etc/dfs/dfstab`:

```bash
cat /etc/dfs/dfstab
share -F nfs -o rw /export/home
```

Useful extras: `shareall` (apply `dfstab`), `unshare`, `share` (list current shares), `dfshares <host>` (list a server's shares). On Solaris 11 with ZFS, sharing is often done through a dataset property: `zfs set share.nfs=on pool/dataset`. The Linux `-t` flag selects the filesystem type, while Solaris uses `-F`.

#### Process mapping (open files)

```bash
# Ubuntu
sudo lsof -c apache2

# Solaris
pfiles `pgrep httpd`
```

`pfiles` shows file-descriptor type, number and file name for a process. The Apache process is named `apache2` on Debian-family systems and `httpd` on Solaris. `lsof` can also be installed on Solaris, but `pfiles` is the native tool.

#### Executable access (tracing system calls)

```bash
# Ubuntu
sudo strace -p `pgrep apache2`

# Solaris
truss ls
execve("/usr/bin/ls", 0xFFBFFDC4, 0xFFBFFDC8)  argc = 1
...SNIP...
```

`truss` (Solaris) and `strace` (Linux) both trace the **system calls** a process makes, with arguments and return values. This helps with debugging errors, performance and network issues, and it can reveal sensitive data such as file paths or credentials passed to syscalls.

> [!warning] strace can follow child processes
> The module claims `strace` cannot trace signals or child processes. Both are wrong: `strace` shows signals received by the process, and **`strace -f`** follows child processes (`truss -f` does the same on Solaris).

### 29.5 Service management (SMF vs systemd)

The module mentions SMF but doesn't show commands. These are the equivalents:

| Task | systemd | SMF |
|---|---|---|
| List services | `systemctl list-units --type=service` | `svcs -a` |
| Status of a service | `systemctl status ssh` | `svcs -l ssh` |
| Start / stop | `systemctl start\|stop ssh` | `svcadm enable\|disable ssh` |
| Restart | `systemctl restart ssh` | `svcadm restart ssh` |
| Why a service failed | `journalctl -u ssh` | `svcs -xv` |
| Configuration | unit files | `svccfg` (service manifests) |

### 29.6 Why this matters for security work

- You may meet Solaris on legacy enterprise targets. The same ideas apply (SUID binaries, NFS misconfigurations, weak permissions, credentials in logs), but the commands differ.
- Check **NFS shares** with `dfshares` or `showmount -e <ip>`, and look for world-writable exports in `/etc/dfs/dfstab`.
- Enumerate **RBAC**: `roles`, `profiles` and `auths` show what a user can escalate to.
- Check SMF services with `svcs -a`, since a vulnerable network service may be running.

> [!tip] Quick Solaris cheat sheet
> `showrev -a` (system info) · `pkg install` (packages) · `svcs` / `svcadm` (services) · `share` / `dfstab` (NFS) · `pfiles <pid>` (open files) · `truss` (syscall trace) · `find / -perm -4000` (SUID).

---

## 30. Shortcuts

> [!abstract] Overview
> Keyboard shortcuts in the terminal (handled by the shell's **Readline** library, which Bash uses) save typing and keep your hands off the mouse. Learn the few below until they become habit.

### 30.1 Auto-complete

| Shortcut | Action |
|---|---|
| `TAB` | Completes commands, file and directory names, and options based on what you've typed. |
| `TAB TAB` | Press twice to list all possible completions when there is more than one match. |

### 30.2 Cursor movement

| Shortcut | Action |
|---|---|
| `Ctrl + A` | Jump to the **beginning** of the line |
| `Ctrl + E` | Jump to the **end** of the line |
| `Ctrl + ←` / `Ctrl + →` | Move one word backward / forward |
| `Alt + B` / `Alt + F` | Move one word backward / forward |

### 30.3 Erase and paste text

| Shortcut | Action |
|---|---|
| `Ctrl + U` | Erase from the cursor to the **beginning** of the line |
| `Ctrl + K` | Erase from the cursor to the **end** of the line |
| `Ctrl + W` | Erase the **word** before the cursor |
| `Ctrl + Y` | **Paste** (yank) the text you just erased |

> [!tip]
> The erased text goes into a "kill buffer", so `Ctrl + U` followed by `Ctrl + Y` works like cut and paste. This is handy for moving a half-typed command out of the way, running something else, then pasting it back.

### 30.4 Process control

| Shortcut | Signal | Action |
|---|---|---|
| `Ctrl + C` | `SIGINT` | Interrupts the running process (for example a scan), with no confirmation. A program can catch or ignore it. |
| `Ctrl + Z` | `SIGTSTP` | **Suspends** the process (it stays in memory). Resume with `fg` (foreground) or `bg` (background). |
| `Ctrl + D` | none (EOF) | Closes `STDIN` (End-of-File). On an empty prompt it **logs you out or exits the shell**. |
| `Ctrl + \` | `SIGQUIT` | Quits and may dump core (a stronger interrupt). |

> [!warning] Corrections and clarifications
> - `Ctrl + Z` **suspends** a process, it doesn't put it in the background. Use `bg` to let it continue in the background, or `fg` to bring it back.
> - `Ctrl + D` isn't a signal. It sends an end-of-file marker, which is why it also exits a shell at an empty prompt.
> - `Ctrl + C` can be handled by programs, so it won't always stop a stubborn process. Then use `Ctrl + Z` followed by `kill %1`, or `kill -9 <PID>` as a last resort.

```bash
ping 10.10.10.10     # running...
^Z                   # suspended
[1]+  Stopped        ping 10.10.10.10
bg                   # continue in the background
jobs                 # list jobs
fg %1                # bring it back to the foreground
```

### 30.5 Screen and history

| Shortcut | Action |
|---|---|
| `Ctrl + L` | Clear the terminal (same as `clear`; it keeps your scrollback) |
| `Ctrl + R` | **Reverse search** through command history; type part of a command, press `Ctrl + R` again for the next match |
| `↑` / `↓` | Previous / next command in history |

Extra history tricks (beyond the module):

| Shortcut | Action |
|---|---|
| `Ctrl + G` | Cancel a history search |
| `!!` | Repeat the last command (`sudo !!` reruns it with sudo) |
| `!$` | Reuse the last argument of the previous command |
| `history \| grep <text>` | Search the saved history |

### 30.6 Desktop shortcuts

| Shortcut | Action |
|---|---|
| `Alt + Tab` | Switch between open applications |
| `Ctrl + +` / `Ctrl + -` | Zoom in / out (in most terminal emulators; some need `Ctrl + Shift + +`) |

> [!tip] Quick cheat sheet
> **Move:** `Ctrl+A` / `Ctrl+E` · **Delete:** `Ctrl+U` / `Ctrl+K` / `Ctrl+W` · **Paste:** `Ctrl+Y` · **Stop:** `Ctrl+C` · **Suspend:** `Ctrl+Z` · **Exit:** `Ctrl+D` · **Clear:** `Ctrl+L` · **Search history:** `Ctrl+R`
