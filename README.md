# LDP6 — Linux Disposable Platform (6-Hour)

[![GitHub Actions](https://img.shields.io/badge/Powered%20by-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Ubuntu 24.04](https://img.shields.io/badge/OS-Ubuntu%2024.04%20LTS-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Tailscale](https://img.shields.io/badge/Network-Tailscale-242424?logo=tailscale&logoColor=white)](https://tailscale.com/)

A fully automated, cloud-hosted Ubuntu 24.04 desktop environment that spins up in under 2 minutes via GitHub Actions. Access it securely from anywhere through a private Tailscale tunnel and a VNC connection — no cloud bills, no infrastructure management, no trace left behind.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [One-Time Setup](#one-time-setup)
- [Launching the Environment](#launching-the-environment)
- [Connecting via VNC](#connecting-via-vnc)
- [Included Software](#included-software)
- [Important Limitations](#important-limitations)
- [FAQ](#faq)

---

## Overview

LDP6 provisions a **disposable Linux desktop** on GitHub's infrastructure. When you trigger the workflow:

1. A fresh Ubuntu 24.04 runner boots in GitHub Actions.
2. The environment installs a full XFCE4 desktop, browser, and dev tools.
3. A Tailscale node joins **your private network only** — no public exposure.
4. You connect via VNC and have a full graphical Linux environment for up to **6 hours**.
5. When the timer ends (or you close the workflow), everything is wiped automatically.

**Use cases:** Isolated testing, sandboxed browsing, running untrusted scripts, CI debugging, temporary build environments, and more.

---

## Architecture

```
┌──────────────────────────────────────────────┐
│              GitHub Actions Runner            │
│                                              │
│  ┌─────────────┐     ┌──────────────────┐   │
│  │  Ubuntu 24  │────▶│  VNC Server      │   │
│  │  XFCE4      │     │  :5901           │   │
│  └─────────────┘     └──────────────────┘   │
│         │                                    │
│  ┌──────▼──────┐                             │
│  │  Tailscale  │  ◀── Ephemeral Node         │
│  │  Agent      │      (auto-removes itself)  │
│  └─────────────┘                             │
└──────────────────────────────────────────────┘
         │  Private Tailscale Mesh (WireGuard)
         ▼
┌─────────────────┐
│   Your Device   │
│   VNC Viewer    │
│   Tailscale     │
└─────────────────┘
```

Traffic flows exclusively over your Tailscale mesh (end-to-end encrypted WireGuard). The runner is **never exposed to the public internet**.

---

## Prerequisites

Install both tools on your local machine before your first session.

| Tool | Purpose | Download |
|------|---------|----------|
| **Tailscale** | Creates the secure private tunnel to the runner | [tailscale.com/download](https://tailscale.com/download) |
| **VNC Viewer** | Connects to the remote desktop (port 5901) | [RealVNC](https://www.realvnc.com/en/connect/download/viewer/) · [TigerVNC](https://tigervnc.org/) |

> **Note:** Tailscale must be running on your device and logged into the **same account** whose auth key you add to GitHub Secrets.

---

## One-Time Setup

This setup takes about 5 minutes and only needs to be done once.

### Step 1 — Generate a Tailscale Auth Key

1. Go to the [Tailscale Admin Console → Keys](https://login.tailscale.com/admin/settings/keys).
2. Click **Generate auth key** and configure it:
   - ✅ **Reusable** — so you can launch multiple sessions without generating a new key each time.
   - ✅ **Ephemeral** — the node automatically removes itself from your Tailscale account when the session ends.
3. Copy the generated key (it starts with `tskey-auth-...`).

### Step 2 — Add the Key to GitHub Secrets

1. In this repository, navigate to **Settings → Secrets and variables → Actions**.
2. Click **New repository secret**.
3. Set the name to `TAILSCALE_AUTH_KEY` and paste the key as the value.
4. Click **Add secret**.

Your key is now stored securely. GitHub Actions will inject it into the workflow at runtime without exposing it in logs.

---

## Launching the Environment

1. Navigate to the **Actions** tab of this repository.
2. Select **Linux-Disposable-2026** from the workflow list in the left sidebar.
3. Click the **Run workflow** dropdown → **Run workflow**.
4. Wait approximately **1–2 minutes** for the environment to fully initialize.

The workflow log will confirm when the desktop is ready and display the connection address.

---

## Connecting via VNC

Once the workflow is running, connect using these credentials:

| Field | Value |
|-------|-------|
| **Host** | `gh-linux` |
| **Port** | `5901` |
| **Full Address** | `gh-linux:5901` |
| **Password** | `password` |
| **Resolution** | 1920 × 1080 (pre-configured) |

> **Tip:** If `gh-linux` doesn't resolve, check the running workflow's logs for the Tailscale-assigned IP address and use that directly (e.g., `100.x.x.x:5901`).

**Connection steps (RealVNC example):**
1. Open VNC Viewer.
2. Enter `gh-linux:5901` in the address bar.
3. Accept the unencrypted VNC warning (the transport is secured by Tailscale/WireGuard).
4. Enter `password` when prompted.

---

## Included Software

The environment is pre-configured out of the box — no additional installation required for common tasks.

| Category | Software |
|----------|----------|
| **Operating System** | Ubuntu 24.04 LTS |
| **Desktop Environment** | XFCE4 (lightweight, fast) |
| **Browser** | Google Chrome (latest stable) |
| **Version Control** | Git |
| **Build Tools** | build-essential, curl |
| **Monitoring** | htop |

All packages are installed from official Ubuntu repositories during workflow startup.

---

## Important Limitations

| Constraint | Detail |
|-----------|--------|
| **6-Hour TTL** | GitHub Actions enforces a hard 6-hour limit. The runner — and all data on it — is permanently destroyed at the 360-minute mark. |
| **No persistence** | Files saved to the desktop do not survive session end. Use `git push`, cloud storage (S3, Drive, etc.), or `scp` to preserve your work before the timer runs out. |
| **Cold start** | Each session starts from a clean slate. There is no carry-over from previous sessions. |
| **Single user** | Only devices on your Tailscale network can reach the runner. It is not accessible to anyone else. |
| **GitHub fair use** | This workflow uses GitHub-hosted runners. Using it for crypto mining, DDoS, or other abuse violates [GitHub's Terms of Service](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service). |

---

## FAQ

**Q: Can I extend the session beyond 6 hours?**  
A: No. The 6-hour limit is a hard constraint imposed by GitHub Actions and cannot be bypassed. Re-trigger the workflow to start a fresh session.

**Q: Is my traffic private?**  
A: Yes. All traffic between your device and the runner travels over Tailscale's WireGuard-based mesh. The VNC port is never exposed to the public internet.

**Q: What happens to my files when the session ends?**  
A: They are permanently deleted when the GitHub Actions runner is torn down. Always push code to a remote repository or upload files to external storage before your session expires.

**Q: Can I install additional software?**  
A: Yes. You have full `sudo` access within the session. Install anything you need via `apt`, `snap`, or other package managers. Changes persist for the duration of the session only.

**Q: The address `gh-linux` isn't resolving — what do I do?**  
A: Open the running workflow log and look for the line where Tailscale prints the assigned IP address. Use that IP directly in your VNC client (e.g., `100.x.x.x:5901`).


---

<p align="center"><b>Built for developers who need a clean slate, fast.</b></p>

<h1 align="center"><b>Built by @Jack111I</b></h1>
