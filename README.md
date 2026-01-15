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

---

- Download & Install TigerVNC: https://sourceforge.net/projects/tigervnc/

---

Recommended lighter daily usage (optional):

```bash
kex --sl
```

---

## Extra: Error Cases

If you encounter errors when using Win‑KeX or related tools, review the specific cases below for common causes and step-by-step recovery instructions. Follow the checklist in each subsection before reporting bugs or seeking help.

### 5.1 Error Case 1: TigerVNC abort (critical)

**What this error really means**

Win-KeX (via TigerVNC Viewer) may show:

<div align="center">

![](https://imgur.com/whmNxXu.png)

</div>

This message can be either normal behavior or a real KeX failure. The key is **when** it happens.

#### Case A — 10053 appears after you stop KeX / shutdown WSL (normal)

This is **expected** if you:

- Shut down Kali (e.g., `sudo poweroff`)
- Stop KeX (`kex --stop`)
- Shut down WSL (`wsl --shutdown`)
- While the TigerVNC Viewer window is still open

**What’s happening**

- The VNC server stopped intentionally
- Windows detects the connection is gone
- TigerVNC reports it as 10053 (notification, not a crash)

**Correct action**

- Click **No** (do not reconnect)
- Close TigerVNC Viewer
- Do nothing else

**Do NOT**

- Click “Reconnect”
- Run recovery commands
- Reinstall Win-KeX

#### Case B — 10053 appears while Kali is running (real problem)

This is the critical case. Typical symptoms:

- KeX disconnects immediately after launch
- “Reconnect” loops endlessly
- The GUI never stays open

Common causes:

- Corrupted/stale VNC socket files
- Broken KeX user config
- Interrupted KeX startup
- Security software interfering on the Windows side

**Recovery checklist (ONLY for Case B)**

Run these in order from a fresh Kali shell:

1. Stop KeX cleanly

```bash
kex --stop
```

2. Kill any ghost VNC / KeX processes

```bash
pkill Xtigervnc
pkill kex
```

3. Remove broken VNC display files

```bash
rm -rf ~/.vnc
```

4. Remove KeX user configuration

```bash
rm -rf ~/.config/kex
```

5. Reset the KeX password

```bash
kex --passwd
```

- When asked `Viewonly password (y/n)?` answer: `n`
- Use a 6–8 character password

6. Start KeX in the most stable mode

```bash
kex --win --force
```

**Important rule (do NOT skip)**

If 10053 is happening during a real crash, don’t click “Reconnect” in TigerVNC.
Close the viewer and restart KeX cleanly after completing the checklist.

**Quick decision table**

<div align="center">

| Situation                           | What to do                                                            |
| ----------------------------------- | --------------------------------------------------------------------- |
| 10053 after shutdown / `kex --stop` | Click **No**, close TigerVNC (normal)                                 |
| 10053 during KeX startup            | Click **YES (once)**, Run the recovery checklist above                |
| 10061 (connection refused)          | Click **No**, Restart KeX and/or run `wsl --shutdown`, then try again |
| GUI opens normally                  | Ignore past errors                                                    |

</div>

---

### 5.2 Error Case 2: Xfce notification daemon warning (non-critical)

<div align="center">

![](https://imgur.com/DWDVshC.png)

</div>

The “Wayland compositor does not support required protocol wlr-layer-shell” notice is purely cosmetic. It appears because the Xfce notification daemon briefly probes Wayland protocols while the rest of KeX runs on X11. The session keeps working despite the message, but you can silence it.

**Optional: permanently suppress the popup**

1. `mkdir -p ~/.config/autostart`
2. `cp /etc/xdg/autostart/xfce4-notifyd.desktop ~/.config/autostart/`
3. `nano ~/.config/autostart/xfce4-notifyd.desktop` and change (or add) `Hidden=false` → `Hidden=true`.
4. Save and exit the editor.
5. Restart the KeX GUI with `kex --stop` followed by `kex --win`.

Once those lines are updated, the notification daemon will stay quiet for good.

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
