# WSL2 + Kali Linux + Win-KeX (GUI) Installation Guide (Beginner-Friendly)

This notebook-style guide walks you through installing **WSL2**, **Kali Linux**, and **Win-KeX** (Kali GUI) on Windows, including recommended prompt answers and common fixes.

---

## 0) Prerequisites (Do these first)

### 0.1 Virtualization must be enabled
**How to check**
- Open **Task Manager → Performance → CPU**
- Look for: **Virtualization: Enabled**

<div align="center">

![](https://imgur.com/Z5pW4W4.png)
</div>

If it’s disabled, enable virtualization in your **BIOS/UEFI** settings.

---

### 0.2 Install the latest graphics driver (important for GPU acceleration)
If you have **NVIDIA**, the recommended option is **Studio Driver**:
- Supports **WSL GPU acceleration**, **CUDA development**, and creative apps
- Occasional gaming still works fine (only very new games may miss day-one optimizations)

**Check current driver date**
- **Task Manager → Performance → GPU → Driver date**

<div align="center">

![](https://imgur.com/UXHWyuo.png)
</div>

---

### 0.3 Enable required Windows Features
Open:
- Press the **Windows key** → search **“Turn Windows features on or off”**

Enable:
1. **Virtual Machine Platform**
2. **Windows Subsystem for Linux**

<div align="center">

![](https://imgur.com/PDDBRJG.png)
</div>
Restart Windows if prompted.

---

## 1) Step 1 — Update WSL (PowerShell as Admin)

Open **PowerShell (Admin)** and run:

```powershell
wsl --update
wsl --version
```

Check WSL status:

```powershell
wsl
```

If no distro is installed yet, it will tell you no distro exists.

---

## 2) Step 2 — Install Kali Linux

List available WSL distros:

```powershell
wsl.exe --list --online
```

Install Kali:

```powershell
wsl --install -d kali-linux
```

---

## 3) Step 3 — Set Kali as default + start it

Set default distro:

```powershell
wsl --setdefault kali-linux
```

Start Kali using either command:

```powershell
wsl
```

or:

```powershell
bash
```

---

## 4) Step 4 — Update packages inside Kali

Inside the Kali WSL terminal, run:

```bash
sudo apt update
```

---

## Extra: Fix Kali mirror / `apt update` errors (if needed)

If `sudo apt update` shows mirror/network errors:

### 4.1 Edit Kali sources
```bash
sudo nano /etc/apt/sources.list
```

### 4.2 Replace everything with:
```text
deb https://ftp.halifax.rwth-aachen.de/kali kali-rolling main contrib non-free non-free-firmware
```

Then run:
```bash
sudo apt update
```

---

## 5) Step 5 — Install Win-KeX (Kali GUI for WSL)

Inside Kali:

```bash
sudo apt install kali-win-kex
```

Start KeX:
```bash
kex --win
```

---

## KeX “Viewonly password (y/n)?” popup

**Answer: `n`**

Why `n` is correct:
- `y` = view-only (cannot click or type)
- `n` = full control (keyboard + mouse)

For pentesting/labs/tools, you need full control.

What happens next:
- KeX server starts
- Windows KeX window launches automatically
- Kali desktop appears

Recommended lighter daily usage (optional):
```bash
kex --sl
```

---

## Quick KeX control commands

```bash
kex --status    # check running status
kex --stop      # stop KeX
kex --win       # full desktop
kex --sl        # seamless mode
```

---


Stop KeX:
```bash
kex --stop
```

---

## 6) Step 6 — (Optional) Install big Kali toolkit

In the KeX Linux terminal (or Kali terminal), run:

```bash
sudo apt install kali-linux-large
```

## Installer Prompts (Recommended answers + why)

### Prompt: Install Kismet “setuid root”?
Choose: **Yes**

<div align="center">

![](https://imgur.com/8MwsvoU.png)
</div>

Why **Yes** is correct:
- Kismet needs privileges for:
  - Monitor mode
  - Packet capture
  - WiFi analysis workflows
- Selecting Yes:
  - Sets permissions safely
  - You don’t need to run Kismet as root every time
  - Typical/recommended on Kali

---

### Prompt: “Should non-superusers be able to capture packets?”
Choose: **Yes**

<div align="center">

![](https://imgur.com/htxIEyF.png)
</div>

Why **Yes** is right:
- Lets you run Wireshark/Tshark without `sudo`
- Safer than running full Wireshark as root
- Recommended setup

What happens:
- Uses group `wireshark`
- Capture handled by `dumpcap` (minimal privileged component)

**IMPORTANT after installation (don’t skip)**

Check group membership:
```bash
groups
```

If you see `wireshark`:
- You’re done.

If you do **NOT** see `wireshark`, run:
```bash
sudo usermod -aG wireshark $USER
```

Then restart WSL:
```powershell
wsl --shutdown
```

Verify again (reopen Kali):
```bash
groups
```

You should see:
- `wireshark`

---

### Prompt: Run `sslh`: select `inetd` or `standalone`?
Select: **standalone**

<div align="center">

![](https://imgur.com/NkwsQPk.png)
</div>

Why standalone:
- More stable and faster for pentesting/tools
- Doesn’t spawn a new process per connection
- Recommended for Kali usage

Rule of thumb:
- Pentesting/labs/tools → **standalone**
- Rare/minimal server traffic → **inetd**

---

## Tips: Stopping KeX vs stopping WSL

### 1) Stop only the KeX GUI (keep WSL distro running)
```bash
kex --stop
```

Effect:
- Stops Win-KeX server and closes KeX window(s)
- Kali WSL instance keeps running

---

### 2) Close your current shell/desktop session
Inside Kali:
```bash
exit
```
(or logout)

Effect:
- Closes that session
- WSL may still remain running in the background

---

### 3) Stop (terminate) only Kali from Windows (recommended to fully stop just Kali)
PowerShell:
```powershell
wsl --terminate kali-linux
```

Shorthand:
```powershell
wsl -t kali-linux
```

Effect:
- Immediately ends that distro process (stops services, closes shells)

---

### 4) Shutdown the entire WSL VM (all distros)
PowerShell:
```powershell
wsl --shutdown
```

Effect:
- Stops WSL2 utility VM and all running distros
- Good after big installs or to free resources

---

## Error Case 1: `kex` not found + `dpkg` interrupted

Example:
```bash
kex --win
```

You may see:
- `Command 'kex' not found, but can be installed with: sudo apt install kali-win-kex`
- During install/reinstall:
  - `Error: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem.`

Meaning:
- The package system was left unfinished, so `apt` is blocked until repaired.

### Step 1 — Fix broken dpkg state (MANDATORY)
Run inside Kali:
```bash
sudo dpkg --configure -a
```

### Step 2 — Fix broken dependencies (if any)
```bash
sudo apt --fix-broken install
```

### Step 3 — Update packages list
```bash
sudo apt update
```

### Step 4 — Reinstall Win-KeX cleanly
```bash
sudo apt install --reinstall kali-win-kex
```

### Step 5 — Start KeX
```bash
kex --win
```

Note:
- First run will ask you to set a **KeX password**.
