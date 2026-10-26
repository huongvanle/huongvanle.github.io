---
layout: post
title: Running Claude Science on Windows, then and now
date: 2026-10-26 09:00:00 +0900
description: From a WSL workaround during the beta to a native Windows app, and when the Linux route still makes sense
tags: claude-science windows wsl tools
categories: tools
---

[Claude Science](https://claude.com/product/claude-science) is Anthropic's research application for scientists. It runs locally on your own computer rather than as a website: it sets up its own Python and R environments, connects to scientific databases and tools, and asks before it reads one of your folders or runs code. Results come back as artifacts you can open and download. It needs a Claude Pro, Max, Team or Enterprise plan.

## The beta: Linux and macOS only

When I first tried the beta in July 2026, it ran only on macOS and Linux. To make it easier for Windows users to try it, I tested whether the Linux version would run under Windows Subsystem for Linux (WSL 2). It did, and I shared the steps as a [short guide on LinkedIn](https://lnkd.in/p/gbxNGCmd) on 14 July.

## Now: a native Windows app

Claude Science now ships a Windows installer. You download it from the product page, open it, and it installs for your user account, adds itself to the Start menu and opens in its own window. No WSL, no terminal. For most Windows users this is the way to go, and it replaces my earlier guide.

A few differences on Windows are worth knowing before you start:

- Claude runs shell commands with PowerShell. Running git inside cells, Command Prompt scripts and changing folders in PowerShell need a one-time Windows permission. Python and R cells work without it.
- You can only grant access to folders on local NTFS or ReFS drives, not network shares, mapped drives or most USB sticks.
- The **Model endpoints** option under **Settings > Compute** (NVIDIA BioNeMo NIM) is not available in the Windows app.

## When WSL still makes sense

The Linux version under WSL is still documented and supported. I would still pick it if:

- my pipeline depends on Linux-only command-line tools,
- I want Model endpoints (BioNeMo NIM), which the Windows app doesn't offer, or
- I want the same environment as the Linux servers I run jobs on.

Projects in the two setups are kept separately: work you create under WSL stays in WSL, and the Windows app keeps its own.

## A correction to my July guide

My original guide used **Ubuntu 22.04**. Claude Science's sandbox now needs bubblewrap 0.8.0 or later, and Ubuntu 22.04 ships an older version, so use **Ubuntu 24.04 or newer**. The short version of the current WSL setup:

```powershell
# PowerShell, as Administrator
wsl --install -d Ubuntu-24.04
```

```bash
# Inside Ubuntu
sudo apt update && sudo apt install -y curl bubblewrap socat
curl -fsSL https://claude.ai/install-claude-science.sh | bash
. ~/.profile
claude-science serve --port 8765 --no-browser
```

Open the printed `http://127.0.0.1:8765` link in a Windows browser and sign in. If the link stops working, `claude-science url` prints a fresh one.

For the official, up-to-date steps, see [Get started](https://claude.com/docs/claude-science/run-on-windows). My original WSL walkthrough is [on LinkedIn](https://lnkd.in/p/gbxNGCmd); if you follow it, use Ubuntu 24.04 as noted above.
