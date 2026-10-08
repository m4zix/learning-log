# Linux Cheat Sheet (Backend Essentials)

Commands for Ubuntu/Debian. This covers what a backend developer needs: working in the terminal, reading logs, managing files and permissions, connecting to servers, and running services.

> **Golden rules**
> 1. Linux has no recycle bin. `rm` is permanent, so read the command twice.
> 2. Do not use `sudo` unless you must.
> 3. Do not copy and paste commands you do not understand. Check them with `man <command>` or `<command> --help`.

---

## 1. Navigation

| Command | What it does |
|---|---|
| `pwd` | Prints the current directory |
| `ls` | Lists files |
| `ls -la` | Lists everything (including hidden files that start with `.`) with details |
| `ls -lh` | Same, with human-readable sizes (KB, MB) |
| `cd <dir>` | Goes into a directory |
| `cd ..` | Goes up one level |
| `cd ~` or `cd` | Goes to your home directory |
| `cd -` | Goes back to the previous directory |

**Paths**
- **Absolute:** starts from the root, e.g. `/home/mazen/project`
- **Relative:** starts from where you are, e.g. `./project` or `../other`
- `~` is your home folder, `.` is the current folder, `..` is the parent folder

**Important directories**

| Path | Contains |
|---|---|
| `/home/<user>` | Your files |
| `/etc` | Configuration files |
| `/var/log` | Logs |
| `/tmp` | Temporary files |
| `/usr/bin` | Installed programs |
| `/opt` | Optional/third-party software |
| `/root` | The administrator's home |

---

## 2. Files and directories

| Command | What it does |
|---|---|
| `mkdir <dir>` | Creates a directory |
| `mkdir -p a/b/c` | Creates nested directories |
| `touch <file>` | Creates an empty file (or updates its timestamp) |
| `cp <src> <dest>` | Copies a file |
| `cp -r <dir> <dest>` | Copies a directory |
| `mv <src> <dest>` | Moves **or renames** |
| `rm <file>` | Deletes a file |
| `rm -r <dir>` | Deletes a directory and everything inside it |
| `rm -rf <dir>` | Deletes without asking. **Dangerous**, double-check the path |

**Wildcards:** `*` (any characters), `?` (one character), `[abc]` (one of these). Example: `ls *.py`.

**Tricky filenames**

```bash
cat "my file.txt"        # spaces: use quotes
cat my\ file.txt         # or escape the space with \
cat ./-file              # a name starting with "-": add ./ so it is not read as an option
cat .hidden              # hidden files start with a dot
```

---

## 3. Viewing and editing files

| Command | What it does |
|---|---|
| `cat <file>` | Prints the whole file |
| `less <file>` | Scrollable viewer. `/word` searches, `n` next match, `q` quits |
| `head -n 20 <file>` | First 20 lines |
| `tail -n 20 <file>` | Last 20 lines |
| `tail -f <file>` | Follows a file live (great for logs). `Ctrl+C` stops |
| `wc -l <file>` | Counts lines |
| `file <file>` | Shows the file type (text, binary, archive...) |
| `strings <file>` | Prints readable text inside a binary file |
| `diff <a> <b>` | Shows differences between two files |

**Editing on a server**
- `nano <file>`: `Ctrl+O` then Enter to save, `Ctrl+X` to exit.
- `vim <file>`: press `i` to type, `Esc` when done, then `:wq` (save and quit) or `:q!` (quit without saving).

---

## 4. Searching

### `grep`: search inside files

| Command | What it does |
|---|---|
| `grep "text" <file>` | Lines containing the text |
| `grep -i "text" <file>` | Ignore upper/lower case |
| `grep -n "text" <file>` | Show line numbers |
| `grep -v "text" <file>` | Lines that do **not** contain the text |
| `grep -c "text" <file>` | Count matching lines |
| `grep -r "text" <dir>` | Search all files in a directory |
| `grep -rl "text" .` | Only the names of files that match |

### `find`: search for files

| Command | What it does |
|---|---|
| `find . -name "*.py"` | Files by name |
| `find . -type f` | Only files (`-type d` for directories) |
| `find . -size 1033c` | Exact size in bytes (`c` = bytes, `k` = KB, `M` = MB) |
| `find . -size +10M` | Bigger than 10 MB |
| `find . -user alice` | Files owned by a user |
| `find . -perm 644` | Files with exact permissions |
| `find . -mtime -1` | Modified in the last day |
| `find / -name "x" 2>/dev/null` | Search the whole system and hide "Permission denied" errors |

Other helpers: `which <command>` shows where a program is installed, and `man <command>` opens its manual.

---

## 5. Pipes and redirection

| Symbol | Meaning | Example |
|---|---|---|
| `\|` | Sends the output of one command into another | `cat log.txt \| grep error` |
| `>` | Writes output to a file (**overwrites**) | `ls > files.txt` |
| `>>` | Appends output to a file | `echo "hi" >> notes.txt` |
| `<` | Reads input from a file | `sort < names.txt` |
| `2>` | Redirects errors | `find / -name x 2> errors.txt` |
| `2>/dev/null` | Throws errors away | `find / -name x 2>/dev/null` |
| `2>&1` | Sends errors to the same place as normal output | `cmd > out.txt 2>&1` |

**Text tools to chain with pipes**

| Command | What it does | Example |
|---|---|---|
| `sort` | Sorts lines | `sort names.txt` |
| `uniq` | Removes **adjacent** duplicates (sort first!) | `sort names.txt \| uniq` |
| `uniq -c` | Counts duplicates | `sort names.txt \| uniq -c` |
| `uniq -u` | Shows only lines that appear exactly once | `sort data.txt \| uniq -u` |
| `cut -d',' -f2` | Takes column 2 of a comma-separated file | `cut -d',' -f2 data.csv` |
| `tr` | Replaces characters | `tr 'a-z' 'A-Z'` |
| `base64 -d` | Decodes Base64 | `base64 -d encoded.txt` |

Example: top 5 IP addresses in an access log (assuming the IP is the first field):
```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn | head -5
```

---

## 6. Permissions

Run `ls -l` and you see something like:

```
-rwxr-xr-- 1 mazen devs 1204 Oct 7 10:00 script.sh
```

| Part | Meaning |
|---|---|
| `-` | Type: `-` file, `d` directory, `l` link |
| `rwx` | **Owner** permissions |
| `r-x` | **Group** permissions |
| `r--` | **Others** permissions |
| `mazen devs` | Owner and group |

**r = read (4), w = write (2), x = execute (1).** Add the numbers: `rwx` = 7, `r-x` = 5, `r--` = 4.

| Command | What it does |
|---|---|
| `chmod 755 <file>` | Owner full, others read + execute |
| `chmod 644 <file>` | Owner read/write, others read |
| `chmod 600 <file>` | Only the owner can read and write |
| `chmod +x script.sh` | Makes a file executable |
| `chmod u+x,g-w <file>` | Symbolic form (u=owner, g=group, o=others) |
| `chown user:group <file>` | Changes the owner and group |
| `chown -R user:group <dir>` | Same for a whole directory |

**Common values**

| Value | Use for |
|---|---|
| `644` | Normal files |
| `755` | Directories and scripts |
| `600` | **Private keys** and secret files like `.env` |
| `700` | Private directories (like `~/.ssh`) |
| `777` | **Never use.** Everyone can do everything |

---

## 7. Users and sudo

| Command | What it does |
|---|---|
| `whoami` | Your username |
| `id` | Your user ID and groups |
| `sudo <command>` | Runs one command as the administrator |
| `sudo -i` | Opens an admin shell (use rarely) |
| `su - <user>` | Switches to another user |
| `sudo adduser <name>` | Creates a user |
| `sudo usermod -aG <group> <user>` | Adds a user to a group |
| `passwd` | Changes your password |

Principle: **least privilege.** Run apps as a normal user, not as root.

---

## 8. Processes

| Command | What it does |
|---|---|
| `ps aux` | Lists all processes |
| `ps aux \| grep python` | Finds a process by name |
| `top` or `htop` | Live view of CPU and memory (`q` quits) |
| `kill <PID>` | Stops a process politely |
| `kill -9 <PID>` | Forces it to stop (last resort) |
| `pkill -f <name>` | Stops processes by name |
| `<command> &` | Runs in the background |
| `jobs`, `fg`, `bg` | Manage background jobs |
| `nohup <command> &` | Keeps running after you close the terminal |

---

## 9. Networking

| Command | What it does |
|---|---|
| `ping <host>` | Tests if a host replies (`Ctrl+C` stops) |
| `ip a` | Shows your network interfaces and IP addresses |
| `ss -tulpn` | Lists listening ports and which program owns them (use `sudo` to see programs) |
| `dig <domain>` | DNS lookup |
| `nc <host> <port>` | Opens a raw connection to a port (netcat) |
| `nc -zv <host> <port>` | Checks if a port is open |

### `curl`: test APIs from the terminal

| Command | What it does |
|---|---|
| `curl <url>` | GET request |
| `curl -i <url>` | Shows the response headers too |
| `curl -I <url>` | Headers only |
| `curl -s <url>` | Silent (no progress bar) |
| `curl -L <url>` | Follows redirects |
| `curl -o file.txt <url>` | Saves the response to a file |
| `curl -X POST <url> -H "Content-Type: application/json" -d '{"name":"test"}'` | POST with JSON |
| `curl -H "Authorization: Bearer <token>" <url>` | Sends a token |

`wget <url>` downloads a file.

> Only scan or probe machines you own or practice labs. Port scanning others' systems without permission can be illegal.

---

## 10. SSH and file transfer

```bash
ssh user@host                       # connect
ssh -p 2220 user@host               # custom port (lowercase -p)
ssh -i ~/.ssh/key user@host         # with a private key
```

Example from OverTheWire Bandit:
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

| Command | What it does |
|---|---|
| `ssh-keygen -t ed25519` | Creates a key pair (private + public) |
| `ssh-copy-id user@host` | Installs your public key on a server |
| `scp file user@host:/path/` | Copies a file **to** a server |
| `scp user@host:/path/file .` | Copies a file **from** a server |
| `scp -P 2220 ...` | Custom port for scp (**uppercase `-P`**) |

**Key rules**
- Your private key must be `chmod 600`, or SSH refuses it ("unprotected private key file").
- **Never share your private key**, and never commit it to GitHub. The public key (`.pub`) is safe to share.
- On real servers, prefer SSH keys over passwords.

---

## 11. Installing software (apt)

| Command | What it does |
|---|---|
| `sudo apt update` | Refreshes the package list |
| `sudo apt upgrade` | Installs available updates |
| `sudo apt install <pkg>` | Installs a package |
| `sudo apt remove <pkg>` | Removes a package |
| `sudo apt autoremove` | Removes unused dependencies |
| `apt search <word>` | Searches packages |

Habit: run `sudo apt update` before installing.

---

## 12. Services (systemd) and logs

| Command | What it does |
|---|---|
| `sudo systemctl status <service>` | Shows if a service is running |
| `sudo systemctl start <service>` | Starts it |
| `sudo systemctl stop <service>` | Stops it |
| `sudo systemctl restart <service>` | Restarts it |
| `sudo systemctl reload <service>` | Reloads the config without stopping |
| `sudo systemctl enable <service>` | Starts it automatically at boot |
| `sudo systemctl disable <service>` | Removes the automatic start |
| `sudo systemctl daemon-reload` | Re-reads service files after you edit them |
| `journalctl -u <service> -n 50` | Last 50 log lines of a service |
| `journalctl -u <service> -f` | Follows the logs live |

Other log locations: `/var/log/` (for example `/var/log/nginx/error.log`).

**When something breaks on a server, check in this order:**
```bash
sudo systemctl status <service>      # is it running?
journalctl -u <service> -n 50        # what do the logs say?
ss -tulpn                            # is it listening on the right port?
df -h                                # is the disk full?
```

---

## 13. Firewall (ufw)

```bash
sudo ufw allow OpenSSH      # ALWAYS allow SSH first, or you lock yourself out
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

---

## 14. Environment variables

| Command | What it does |
|---|---|
| `printenv` or `env` | Lists all variables |
| `echo $NAME` | Prints one variable |
| `export NAME=value` | Sets a variable for this session |
| `echo $PATH` | Shows where the shell looks for programs |
| `nano ~/.bashrc` | Put `export` lines here to make them permanent |
| `source ~/.bashrc` | Applies the changes |

Never put secrets in commands you type (they are saved in your history) or in scripts you commit.

---

## 15. Scheduling (cron)

```bash
crontab -e      # edit your scheduled jobs
crontab -l      # list them
```

Format: `minute hour day-of-month month day-of-week command`

```
0 8 * * *    /usr/bin/python3 /home/mazen/report.py >> /home/mazen/report.log 2>&1
*/15 * * * * /home/mazen/check.sh
```

- `*` means "every". `0 8 * * *` = every day at 08:00.
- Always use **absolute paths** inside cron jobs.

---

## 16. Archives

| Command | What it does |
|---|---|
| `tar -czf backup.tar.gz <dir>` | Creates a compressed archive |
| `tar -xzf backup.tar.gz` | Extracts a `.tar.gz` |
| `tar -xjf file.tar.bz2` | Extracts a `.tar.bz2` |
| `tar -tf file.tar.gz` | Lists the contents without extracting |
| `gzip file` / `gunzip file.gz` | Compress / decompress a single file |
| `bzip2 -d file.bz2` | Decompress a bzip2 file |
| `zip -r a.zip <dir>` / `unzip a.zip` | Zip files |

`tar` flags: `c` create, `x` extract, `z` gzip, `j` bzip2, `f` file name, `t` list.

---

## 17. System information

| Command | What it does |
|---|---|
| `df -h` | Free disk space |
| `du -sh <dir>` | Size of a directory |
| `free -h` | Memory usage |
| `uname -a` | Kernel and system info |
| `cat /etc/os-release` | Which Linux version |
| `uptime` | How long the system has been up and its load |
| `date` | Current date and time |

---

## 18. Keyboard shortcuts and history

| Shortcut | What it does |
|---|---|
| `Tab` | Autocompletes names (press twice for options) |
| `Up / Down` | Previous / next command |
| `Ctrl+R` | Search your command history |
| `Ctrl+C` | Stops the running command |
| `Ctrl+D` | Exits the shell (end of input) |
| `Ctrl+L` | Clears the screen |
| `Ctrl+A` / `Ctrl+E` | Jump to start / end of the line |
| `!!` | Repeats the last command (`sudo !!` repeats it with sudo) |
| `history` | Lists past commands |

---

## 19. Safety checklist

- `rm -rf` and `chmod -R` are the commands that cause the most disasters. Check the path first, and never run them with a variable that might be empty.
- Do not use `chmod 777`. Fix the real permission problem instead.
- Do not run `curl <url> | bash` from sources you do not trust.
- Do not store passwords, tokens or keys in scripts, shell history or Git. Use environment variables and `.env` files listed in `.gitignore`.
- Practice security tools only on your own machines or practice labs (like OverTheWire Bandit, TryHackMe or PortSwigger).
