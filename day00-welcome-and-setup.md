# Day 0 — Welcome & Environment Setup

*Adaps Fresher Training | Linux → Vertica → Grafana → Capstone*

> 🕘 **This is Hour 0 of Day 1, done live in class.** You're walking in with a stock Windows 11 laptop and nothing installed — that's expected. We do this together, as a group, before touching anything about the filesystem or Linux commands. Budget ~45–60 minutes for this block; then we roll straight into the rest of Day 1.

> 🎓 **Trainer note:** Slot this at the very start of Day 1's morning session, ahead of the FHS/navigation content. `wsl --install` + restart + first-boot is the main time sink — use that dead time to talk through the "why this order" and "how these notes work" sections out loud rather than sitting in silence. Anyone who hits a snag (antivirus blocking WSL, BIOS virtualization disabled, etc.) should be pulled aside 1:1 so the rest of the batch isn't blocked.

---

## 👋 Welcome

You're about to spend 10 days going from "never touched a terminal" to "built and demoed a live analytics dashboard, end to end." No fluff, no theory-only slides — every day is hands-on.

**The quick facts:**

| | |
|---|---|
| Duration | 10 working days, full-day, in-classroom |
| Format | Explain → demo → you build it → we review |
| Batch size | 5 of you, 1 trainer, all on your own machines |
| Path | Linux (Days 1–4) → Vertica (Days 5–7) → Grafana (Days 8–9) → Capstone (Day 10) |

**Why this order?** Grafana is basically useless without real data underneath it. Vertica is a pain to install and load if you can't navigate a filesystem or run a package manager. So: Linux first (the ground floor), Vertica next (the data), Grafana last (making that data look good). By Day 10 you'll load fresh data into Vertica *and* build a dashboard on it, solo, no hand-holding.

No prior Linux, database, or dashboarding experience needed. If you can write a basic `if` condition and a `for` loop in any language, you're already ahead.

---

## 📖 How these notes work

Every day's `.md` file follows the same shape — treat it as your single reference, not just class notes:

1. **Recap** — 3 lines on what you did yesterday
2. **Concepts** — what you're learning today, explained plainly
3. **Hands-on** — guided walkthroughs with commands you run *with* the trainer
4. **Lab** — the "now you do it alone" exercise
5. **Copy-paste reference** — every command/query used that day, in one block, for revision later
6. **Tomorrow** — a 2-line preview

Bookmark these. You'll come back to Day 1's command reference on Day 9, guaranteed.

---

## ✅ What you're starting with

### Hardware (your machine)
- Windows 11, **x86_64 (Intel/AMD)** — not ARM, not Apple Silicon. Vertica has zero ARM support, full stop.
- 16 GB RAM minimum (you're running Linux + a database *inside* Windows — it adds up)
- 20–30 GB free disk space
- Working internet (this block involves downloads — classroom Wi-Fi permitting)

### Software (we install this together, right now)
- WSL2 with Ubuntu — your actual Linux environment
- Vertica Community Edition — installed Day 5 (licensing is in progress on the trainer's end — more on that when we get there)
- Grafana OSS — installed Day 8
- DBeaver Community Edition — a GUI for talking to Vertica, installed Day 6

Nothing here costs money. Everything is free/open-source.

---

## 🛠️ Setting up WSL2 + Ubuntu

Follow along step by step — don't jump ahead even if you're quick. The `wsl --install` step needs a restart, and we want everyone restarting and relaunching at roughly the same time so the trainer can walk the room during the slow parts (downloads, first boot) instead of firefighting five different stages at once.

### Step 1: Enable WSL2

Open **PowerShell as Administrator** (right-click Start → Terminal (Admin)) and run:

```powershell
wsl --install -d Ubuntu
```

This does three things in one shot: enables the Windows features WSL2 needs, downloads the WSL2 Linux kernel, and installs the latest Ubuntu LTS.

**Restart your machine** when it asks you to. This step is not optional — skipping the restart causes weird half-broken installs.

### Step 2: First launch

After restart, open **Ubuntu** from the Start menu (or type `wsl` in PowerShell). First launch takes a minute to finish setup, then asks you to create a **UNIX username and password**.

> 💡 This username/password is *separate* from your Windows login. Pick something you won't forget — you'll type this password a lot with `sudo`.

### Step 3: Update everything

Inside your new Ubuntu terminal:

```bash
sudo apt update && sudo apt upgrade -y
```

Grab a coffee — this can take a few minutes on first run.

### Step 4: Turn on systemd

Vertica and Grafana both run as **systemd services**. Ubuntu-on-WSL2 doesn't enable systemd by default, so we switch it on manually.

```bash
sudo nano /etc/wsl.conf
```

Paste this in (if the file already has content, add the `[boot]` section):

```ini
[boot]
systemd=true
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano). Now restart WSL2 — **from PowerShell, not inside Ubuntu**:

```powershell
wsl --shutdown
wsl
```

### Step 5: Verify systemd is alive

Back inside Ubuntu:

```bash
ps -p 1
```

**Expected output:**
```
    PID TTY          TIME CMD
      1 ?        00:00:00 systemd
```

If it says `systemd` — you're done. If it says `init` or anything else, systemd didn't take; double-check your `/etc/wsl.conf` and repeat Step 4.

### Step 6: Install a few basics

A handful of tools we'll lean on from Day 1 onward:

```bash
sudo apt install -y curl wget gnupg2 ca-certificates dialog openssh-server
```

---

## 🧪 Environment checklist — confirm before we move on

Run these and make sure your output roughly matches. **Don't move to the next section of Day 1 until this passes** — everything from here on assumes a working, systemd-enabled Ubuntu:

```bash
whoami                 # your UNIX username
lsb_release -a          # confirms Ubuntu version
ps -p 1                 # must show systemd
pwd                      # should be /home/<your-username>
```

If all four run clean, raise your hand so the trainer can do a quick visual check, then we continue straight into the rest of Day 1.

---

## 🎯 Ground rules for the next 10 days

- **Type the commands, don't copy-paste blindly.** Muscle memory > clipboard memory. (The copy-paste blocks are for *revision*, not for skipping the typing during class.)
- **Break things.** Seriously. A local WSL2 sandbox is the safest place on earth to mess up `chmod`. That's how this sticks.
- **Ask "why", not just "how".** Every command here has a reason it exists — ask if it's not obvious.
- **Keep a running notes file.** Doesn't need to be fancy — a scratch `.txt` of "commands that confused me" pays off by Day 10.

---

## 👀 Up next — still today: Linux Fundamentals I

Environment's live, so now the real Day 1 content starts: navigating the filesystem with `cd`/`ls`/`tree`, understanding what `/etc`, `/var`, and `/home` actually mean, and creating/moving/deleting files like it's second nature. Head to `day01.md` — by end of today, the terminal stops feeling like a foreign country.

Let's go. 🚀
