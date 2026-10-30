# Kali Linux Command Cheatsheet

## Before you start

Open a terminal and check your environment:

```bash
whoami                 # Current username
id                     # User ID, group ID, and group memberships
pwd                    # Current directory
echo "$HOME"           # Home directory
hostname               # Computer name
cat /etc/os-release    # Linux distribution details
ip -br address         # Network interfaces and IP addresses
```

Throughout this guide, `$HOME` refers to your home directory. For example, if your username is `kali`, `$HOME` normally points to `/home/kali`.

Create a folder for the lab and another one for evidence:

```bash
mkdir -p "$HOME/linux_lab/evidence"
cd "$HOME/linux_lab"
```

Take screenshots from your own VM. The sample output in this guide is only an example, not proof that a command was run on your machine.

### Useful terminal shortcuts

| Shortcut | What it does |
|---|---|
| `Tab` | Autocompletes commands and paths |
| `Ctrl+C` | Stops a running command |
| `Ctrl+L` | Clears the terminal screen |
| `Ctrl+R` | Searches command history |
| `↑` | Brings back the previous command |
| `q` | Quits `man`, `less`, or another pager |

## Quick command reference

| Task | Command | Notes |
|---|---|---|
| Show current directory | `pwd` | Print working directory |
| List files | `ls -lah` | Includes hidden files |
| Go home | `cd ~` | Switch to your home directory |
| Go up one directory | `cd ..` | Parent directory |
| Create a directory | `mkdir -p linux_lab` | `-p` also creates missing parents |
| Create an empty file | `touch target1.txt` | Does not clear an existing file |
| Write to a file | `printf 'text\n' > file.txt` | `>` overwrites existing content |
| Append to a file | `printf 'text\n' >> file.txt` | `>>` adds content |
| Read a file | `cat file.txt` | Prints file content |
| Copy a file | `cp -i a.txt b.txt` | Prompts before overwrite |
| Rename or move | `mv -i a.txt b.txt` | Prompts before overwrite |
| Remove a file | `rm -i file.txt` | Prompts before removal |
| Set owner-only read/write | `chmod 600 file.txt` | Owner `rw-`, group and others `---` |
| Check file permissions | `ls -l file.txt` | Shows mode, owner, and group |
| Refresh package lists | `sudo apt-get update` | Does not upgrade installed packages |
| Upgrade installed packages | `sudo apt-get upgrade -y` | Approves prompts automatically |
| Install a package | `sudo apt-get install nmap` | Uses configured APT repositories |
| Check a service | `sudo systemctl status ssh` | Shows SSH service status |
| Show listening TCP ports | `sudo ss -ltnp` | Useful when troubleshooting scans |
| Read help | `command --help` or `man command` | Check options before using a tool |

---

# Part 1: Navigation and file management

**Assignment:** Create `linux_lab` under `/home/<username>/`, make three empty files, write a note, change its permissions, and provide screenshots.

### 1. Create the lab directory

```bash
mkdir -p "$HOME/linux_lab"
cd "$HOME/linux_lab"
pwd
```

Expected directory: `/home/kali/linux_lab` if the username is `kali`. Your path may be different.

### 2. Create three empty files

```bash
touch target1.txt target2.txt notes.txt
ls -lah
```

`touch` creates the file if it does not exist. If it already exists, `touch` updates its timestamps without clearing the content.

### 3. Write text to notes.txt

```bash
printf 'testing notes\n' > notes.txt
cat notes.txt
```

Expected content:

```text
testing notes
```

Use `>` to replace existing content. Use `>>` to append another line. `printf` is useful when you want consistent formatting.

### 4. Change file permissions

Only the file owner should be able to read and write `notes.txt`:

```bash
chmod 600 notes.txt
```

The permission value `600` means:

| Digit | Applies to | Permissions |
|---|---|---|
| `6` | Owner | Read and write (`rw-`) |
| `0` | Group | No permissions (`---`) |
| `0` | Others | No permissions (`---`) |

This is standard Unix file permission behavior. A privileged administrator may still be able to access the file.

### 5. Verify the permissions

```bash
ls -l notes.txt
stat -c '%a %A %n' notes.txt
```

Example output:

```text
-rw------- 1 kali kali 14 Oct 30 16:00 notes.txt
600 -rw------- notes.txt
```

Look for `-rw-------`. The username, date, and other details will vary.

### 6. Capture evidence

Run these commands together so the screenshot shows the folder, permissions, and file content:

```bash
pwd
ls -l notes.txt
cat notes.txt
ls -l target1.txt target2.txt
```

### Check your work

```bash
cd "$HOME/linux_lab"
test -f target1.txt && echo 'target1: OK'
test -f target2.txt && echo 'target2: OK'
test "$(cat notes.txt)" = 'testing notes' && echo 'notes content: OK'
test "$(stat -c %a notes.txt)" = '600' && echo 'permissions: OK'
```

Extra commands worth trying:

```bash
file notes.txt
wc -c notes.txt
cp -i notes.txt notes-backup.txt
diff -u notes.txt notes-backup.txt
```

Before moving on, make sure you can explain the difference between `>` and `>>`, and between `chmod 600` and `chmod 644`.

---

# Part 2: Package management and Nmap

**Assignment:** Update and upgrade packages, install Nmap, check its version, scan localhost, and explain the results of additional Nmap commands.

### 1. Update and upgrade packages

The assignment asks for:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

What each part does:

| Part | Meaning |
|---|---|
| `apt-get update` | Downloads the latest package index from configured repositories |
| `&&` | Runs the next command only when the first command succeeds |
| `apt-get upgrade` | Upgrades eligible installed packages |
| `-y` | Automatically answers yes to confirmation prompts |

You can run the commands separately if you want clearer screenshots:

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

Kali's rolling-release documentation also recommends `apt full-upgrade` for normal system maintenance. For this assignment, use the requested command. Review upgrades before running them on an important machine, and take a VM snapshot when possible.

To capture the terminal output in text files, use this approach **instead of** running the same update twice:

```bash
set -o pipefail
sudo apt-get update 2>&1 | tee "$HOME/linux_lab/evidence/apt-update.txt"
sudo apt-get upgrade -y 2>&1 | tee "$HOME/linux_lab/evidence/apt-upgrade.txt"
```

A result such as `0 upgraded` may simply mean your packages are already up to date. Record what you actually see.

### 2. Install Nmap

```bash
sudo apt-get install nmap
apt-cache policy nmap
```

`apt-cache policy nmap` shows the installed version and the candidate version from your configured repositories. The newest repository version is not necessarily the newest upstream release.

### 3. Check the Nmap version

```bash
command -v nmap
nmap --version
```

Take a screenshot showing the command and version output.

### 4. Scan localhost

```bash
nmap -sV 127.0.0.1
```

| Argument | Meaning |
|---|---|
| `nmap` | Network Mapper |
| `-sV` | Attempts to identify the service and its version |
| `127.0.0.1` | IPv4 loopback address, your own machine |

You may see a table containing `PORT`, `STATE`, `SERVICE`, and `VERSION`.

| State | How to read it |
|---|---|
| `open` | A service is accepting connections on that port |
| `closed` | No service is listening on that port at scan time |
| `filtered` | A filter or network control may be preventing a clear result |

Finding an open port does not automatically mean you have found a vulnerability. Nmap's service detection is a best-effort identification and can be inaccurate.

If there are no open ports, that is still a valid result. Save a screenshot with the scan summary.

### 5. Explore other Nmap options

Run these against localhost and compare the output:

```bash
nmap 127.0.0.1
nmap -sV 127.0.0.1
nmap -p 22,80,443 127.0.0.1
nmap -p- 127.0.0.1
nmap -sT -p 22,80 127.0.0.1
nmap -oN "$HOME/linux_lab/evidence/nmap-localhost.txt" -sV 127.0.0.1
```

| Option | What to look for |
|---|---|
| Default scan | Which common ports are checked and their states |
| `-sV` | Whether a service name or version is identified |
| `-p 22,80,443` | Results for only the specified ports |
| `-p-` | All 65,535 TCP ports; may take longer |
| `-sT` | TCP connect scan |
| `-oN` | Saves normal-format output for your report |

For the exploration section, include at least two different commands. Explain what changed and why. Avoid scanning systems outside your lab.

### Optional: create a local web service to scan

If there are no open ports on localhost, start a basic Python HTTP server.

Terminal A:

```bash
cd "$HOME/linux_lab"
python3 -m http.server 8000 --bind 127.0.0.1
```

Terminal B:

```bash
curl -I http://127.0.0.1:8000
nmap -sV -p 8000 127.0.0.1
```

Stop the Python server with `Ctrl+C` in Terminal A. This gives you a service you can inspect without relying on an external target.

---

# Part 3: Kali Linux tools

## 1. Wireshark: capture network traffic

**Assignment:** Capture packets for 30 to 60 seconds, save the capture, and identify HTTP or DNS packets.

1. Launch Wireshark from the application menu or run `wireshark`.
2. Select the network interface carrying your test traffic. For traffic sent to `127.0.0.1`, choose `lo` (loopback).
3. Start packet capture and let it run for 30 to 60 seconds.
4. Generate some traffic in another terminal.
5. Stop the capture and choose **File > Save As**. Save it as a `.pcapng` file.
6. Use display filters to find the packets you need.

For a predictable HTTP example, start the local server from Part 2:

Terminal A:

```bash
cd "$HOME/linux_lab"
python3 -m http.server 8000 --bind 127.0.0.1
```

Terminal B:

```bash
curl http://127.0.0.1:8000/
```

Wireshark display filters:

```text
http
dns
tcp.port == 8000
ip.addr == 127.0.0.1
```

To generate DNS traffic, try:

```bash
nslookup example.com
dig example.com
```

DNS packets might not appear on `lo` if the resolver uses another interface, cache, or encrypted DNS. Regular HTTPS traffic will not expose readable HTTP payloads. The localhost HTTP example is easier for this exercise.

If Wireshark cannot capture packets, you can use `tcpdump` to capture loopback traffic and open the file in Wireshark later:

```bash
sudo tcpdump -i lo -nn -w "$HOME/linux_lab/evidence/loopback-capture.pcap" tcp port 8000
```

Stop it after 30 to 60 seconds with `Ctrl+C`. Make sure your normal user can read the saved file. Do not run the full Wireshark GUI as root by default.

**Evidence to collect:** The selected interface, packet list, applied filter, packet details, and saved `.pcapng` or `.pcap` filename. Captures can contain sensitive data, so only capture traffic you are allowed to inspect.

## 2. Metasploit: browse available modules

**Assignment:** Open Metasploit and search for exploit modules. You do not need to execute an exploit.

Start Metasploit:

```bash
msfconsole
```

At the `msf6 >` prompt, try:

```text
help
search exploit
show exploits
search type:exploit name:apache
info <module_path_from_search_results>
exit
```

`search exploit` looks for matching modules. `show exploits` lists exploit modules, while `info` displays details about a selected module. Replace the placeholder in `info` with an actual module path from your results.

In your report, pick one module and describe its name, purpose, and supported platform if shown. Do not assume a target is vulnerable just because a module exists.

**Evidence to collect:** The Metasploit prompt, `search exploit` results, and optionally one `info` output.

## 3. Install and check OpenSSH Server

**Assignment:** Install an SSH server on your lab machine.

```bash
sudo apt-get update
sudo apt-get install openssh-server
sudo systemctl start ssh
sudo systemctl status ssh --no-pager
sudo ss -ltnp | grep ':22'
```

The last command looks for a TCP listener on port 22. If your SSH server uses another port, adjust the check.

Check the SSH client and server configuration:

```bash
ssh -V
sudo sshd -t
```

The SSH server should be exposed only as needed for your own lab. Use an isolated VM network and appropriate firewall rules. If you cannot confirm isolation, do not expose the service to other machines.

## 4. Add a test user

Create a separate account for the exercise:

```bash
sudo adduser labhydra
id labhydra
```

Set a password used only for this temporary lab account. Do not reuse a real password. Follow the password prompts from `adduser`.

Check that the account can log in manually before using Hydra:

```bash
ssh labhydra@127.0.0.1
```

Enter the test password. If login works, type `exit` to return to your original terminal.

If login fails, check the account credentials, SSH service, and password authentication configuration. Do not weaken the SSH settings on a production system for a lab exercise.

## 5. Hydra: test a local SSH account

**Assignment:** Use Hydra to demonstrate password guessing against the test account on your own localhost SSH service.

This example uses one lab username, four password candidates, and one concurrent task. Keep the SSH server isolated from external access.

For this example, assume you set the test account password to `LabDemo-2026!`. If you chose a different password, replace that entry in the wordlist.

```bash
printf 'wrong-demo-1\nwrong-demo-2\nLabDemo-2026!\nwrong-demo-3\n' \
  > "$HOME/linux_lab/lab-passwords.txt"
chmod 600 "$HOME/linux_lab/lab-passwords.txt"
```

Check that Hydra is installed:

```bash
hydra -h | head -n 20
```

Install it if needed:

```bash
sudo apt-get install hydra
```

Run the localhost test:

```bash
hydra -l labhydra \
  -P "$HOME/linux_lab/lab-passwords.txt" \
  -t 1 -W 2 -f \
  ssh://127.0.0.1
```

| Option | Meaning |
|---|---|
| `-l labhydra` | Single lab username |
| `-P FILE` | Small password list |
| `-t 1` | One concurrent task |
| `-W 2` | Connection response wait setting in seconds |
| `-f` | Stops after finding one valid login |
| `ssh://127.0.0.1` | SSH service on your own VM |

If Hydra reports a matching login and password, it means a password from the list worked for the test account. It does not prove that OpenSSH has a software vulnerability.

If the test fails, confirm that manual SSH login works and the password in the file matches the test account. Avoid increasing the rate or expanding the target scope.

Once you have captured your evidence, clean up:

```bash
rm -i "$HOME/linux_lab/lab-passwords.txt"
sudo systemctl stop ssh
# Optional, if you no longer need the test user:
# sudo deluser --remove-home labhydra
```

Only stop SSH if it was started for this lab and you do not need it for remote administration. Never run the cleanup command through a remote SSH session you still need.

---

# Troubleshooting

| Problem | What to check |
|---|---|
| `Permission denied` when creating files | Run `pwd` and `ls -ld .`. Work inside `$HOME/linux_lab`. |
| `notes.txt` is missing | Check `pwd` and `ls -lah`. You may be in another directory. |
| File permissions are not `600` | Run `chmod 600 notes.txt` and `stat -c '%a %n' notes.txt`. |
| `Unable to locate package` | Run `sudo apt-get update` and check the configured repositories. |
| APT reports a lock | Another APT or dpkg process may be running. Do not delete lock files blindly. |
| DNS lookup fails during APT update | Check the VM network, `ip route`, and `getent hosts kali.org`. |
| Nmap finds no open ports | Check listeners using `sudo ss -ltnp`, or start the local test server on port 8000. |
| Nmap shows `filtered` | Check firewall and filtering rules. A filtered port is not necessarily open. |
| Wireshark shows no packets | Use the correct interface. For `127.0.0.1`, use `lo`. |
| Wireshark shows only TLS | HTTPS is encrypted. Use the local HTTP server to see plaintext HTTP. |
| Wireshark cannot capture | Check capture permissions or use `tcpdump` in your lab VM. |
| `msfconsole: command not found` | Check with `command -v msfconsole`. Install the package if necessary. |
| `systemctl` does not work | Check whether Kali is running with systemd: `ps -p 1 -o comm=`. |
| SSH says `Connection refused` | Check `sudo systemctl status ssh` and listening ports. |
| SSH says `Permission denied` | Check credentials, account status, and SSH password authentication settings. |
| Hydra finds no matching password | Test one manual SSH login, then check the local password list and service status. |

---
## References

- [Existing class cheatsheet](https://github.com/mrdzer0/sinaubareng/blob/main/Dibimbing/KaliConfiguration/cheatsheet.md)
- [Kali Linux: Updating Kali](https://www.kali.org/docs/general-use/updating-kali/)
- [Debian apt-get manual](https://manpages.debian.org/apt-get)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Metasploit Documentation](https://docs.metasploit.com/)
- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)

The main goal is to understand what each command does and explain the output you actually get. You do not need a successful scan or password match for every step to be useful, but you should be able to explain what happened.
