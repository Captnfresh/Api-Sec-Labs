# Kali Linux API Hacking Lab Setup Guide
> A detailed walkthrough of setting up a professional API security testing environment on Kali Linux using VirtualBox.

**Author:** Adebowale  
**Context:** HAWD (Hacking APIs With Determination) Bootcamp — Lab Setup  
**OS:** Kali Linux 2024.4 (VirtualBox)  
**Date:** April 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1 — Update Kali Linux](#step-1--update-kali-linux)
4. [Step 2 — Create a Dedicated Hacking User](#step-2--create-a-dedicated-hacking-user)
5. [Step 3 — Install Tools](#step-3--install-tools)
   - [Burp Suite](#burp-suite)
   - [Postman](#postman)
   - [jwt_tool](#jwt_tool)
   - [Kiterunner](#kiterunner)
   - [Arjun](#arjun)
   - [OWASP ZAP](#owasp-zap)
   - [Wfuzz](#wfuzz)
6. [Step 4 — Configure Burp Suite](#step-4--configure-burp-suite)
   - [Install Jython](#install-jython)
   - [Install Autorize Extension](#install-autorize-extension)
   - [Install FoxyProxy](#install-foxyproxy)
   - [Import Burp Certificate into Firefox](#import-burp-certificate-into-firefox)
   - [Import MITMweb Certificate into Firefox](#import-mitmweb-certificate-into-firefox)
7. [Step 5 — Download Wordlists](#step-5--download-wordlists)
8. [Tool Reference Guide](#tool-reference-guide)
9. [Troubleshooting](#troubleshooting)

---

## Overview

This guide documents the complete setup of a professional API security testing lab on Kali Linux. The environment is used for ethical hacking, API penetration testing, and learning the OWASP API Security Top 10.

By the end of this guide you will have the following tools installed and configured:

| Tool | Purpose |
|---|---|
| Burp Suite | Intercept and manipulate HTTP/HTTPS traffic |
| Postman | Craft and send API requests directly |
| jwt_tool | Attack and forge JWT tokens |
| Kiterunner | Discover hidden API endpoints |
| Arjun | Discover hidden API parameters |
| OWASP ZAP | Automated vulnerability scanning |
| Wfuzz | Fuzz inputs to find weaknesses |

---

## Prerequisites

- VirtualBox installed on your host machine ([Download here](https://www.virtualbox.org/wiki/Downloads))
- Kali Linux VM downloaded and running ([Download here](https://www.kali.org/get-kali/#kali-virtual-machines))
- Minimum **8GB RAM** allocated to the VM
- Minimum **80GB storage** allocated to the VM
- Internet connection

> **Note:** This guide uses VirtualBox as the hypervisor. VMware Workstation Pro is an alternative. If you are on Apple Silicon, use QEMU with the Kali ARM64 image.

---

## Step 1 — Update Kali Linux

Before installing anything, update the system to make sure all packages are current. Open a terminal and run the following commands one at a time:

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

**What each command does:**

- `apt update` — refreshes the list of available packages from Kali's repositories
- `apt upgrade` — upgrades all currently installed packages to their latest versions
- `apt dist-upgrade` — performs a more thorough upgrade including handling dependency changes

> **Important:** This process can take a long time depending on your internet speed and how outdated your Kali installation is. Do not interrupt it. If your VM crashes mid-upgrade (which can happen), see the [Troubleshooting](#troubleshooting) section.

---

## Step 2 — Create a Dedicated Hacking User

It is good practice to create a separate user account for your hacking activities rather than using the default `kali` account. This keeps your environment organised and follows the principle of least privilege.

Run the following commands one at a time:

```bash
# Create the new user with a home directory
sudo useradd -m hapihacker

# Add the user to the sudo group so it can run admin commands
sudo usermod -a -G sudo hapihacker

# Set zsh as the default shell (Kali's preferred shell)
sudo chsh -s /bin/zsh hapihacker

# Set a strong password for the new user
sudo passwd hapihacker
```

**What each command does:**

- `useradd -m` — creates the user and their home directory at `/home/hapihacker`
- `usermod -a -G sudo` — grants the user admin privileges without removing existing group memberships
- `chsh -s /bin/zsh` — sets zsh as the shell, which has better autocomplete and display features than bash
- `passwd` — prompts you to set a password; choose something strong

Now switch to the new user:

```bash
su - hapihacker
```

You will see the terminal prompt change from `kali@kali` to `hapihacker@kali`. All tool installations from this point should be done as `hapihacker`.

---

## Step 3 — Install Tools

All manually installed tools go into the `/opt` directory. This is the Linux convention for third party software that is not managed by the system package manager.

```bash
cd /opt
```

---

### Burp Suite

Burp Suite comes pre-installed with Kali Linux. Verify it is available by running:

```bash
burpsuite
```

If the application launches you are good. If not, install it with:

```bash
sudo apt-get install burpsuite -y
```

> **What is Burp Suite?** Burp Suite is a web proxy that sits between your browser and the internet. It intercepts every HTTP/HTTPS request before it reaches its destination, allowing you to read, modify, and replay traffic. It is the primary tool for manual API security testing.

---

### Postman

Postman is not pre-installed. Install it with this single command:

```bash
sudo wget https://dl.pstmn.io/download/latest/linux64 -O postman-linux-x64.tar.gz && sudo tar -xvzf postman-linux-x64.tar.gz -C /opt && sudo ln -s /opt/Postman/Postman /usr/bin/postman
```

**What this command does:**
- Downloads the latest Postman Linux package
- Extracts it into `/opt`
- Creates a symlink so you can launch it by just typing `postman`

Verify the installation:

```bash
postman
```

Postman will open and ask you to create a free account. Sign up and log in.

> **What is Postman?** Postman is an API client that lets you craft and send HTTP requests directly to an API without needing a browser or application. It gives you full control over request headers, body, authentication, and parameters — essential for API testing.

---

### jwt_tool

Navigate to `/opt` and clone the repository:

```bash
cd /opt
sudo git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool
python3 -m pip install -r requirements.txt --break-system-packages
```

Create a symlink so you can run it from anywhere:

```bash
sudo chmod +x jwt_tool.py
sudo ln -s /opt/jwt_tool/jwt_tool.py /usr/bin/jwt_tool
```

Verify the installation:

```bash
jwt_tool
```

You should see the jwt_tool banner and usage instructions.

> **What is jwt_tool?** jwt_tool is a toolkit for attacking JSON Web Tokens (JWTs). It automates attacks such as the `alg:none` bypass, weak secret brute forcing, and token forgery — all common vulnerabilities in API authentication.

---

### Kiterunner

Kiterunner is written in Go, so Go must be installed first:

```bash
sudo apt install golang-go -y
```

Verify Go is installed:

```bash
go version
```

Now clone and build Kiterunner:

```bash
cd /opt
sudo git clone https://github.com/assetnote/kiterunner.git
cd kiterunner
sudo make build
```

> **Note:** The `make build` command compiles the Go source code into an executable binary. This may take a few minutes.

Create a symlink:

```bash
sudo ln -s /opt/kiterunner/dist/kr /usr/bin/kr
```

Verify the installation:

```bash
kr version
```

> **What is Kiterunner?** Kiterunner is an API-specific recon tool that discovers hidden endpoints by brute forcing common API paths against a target. Unlike general web fuzzing tools, it understands API structures and uses API-specific wordlists.

---

### Arjun

```bash
cd /opt
sudo git clone https://github.com/s0md3v/Arjun.git
pip3 install arjun --break-system-packages
```

Verify the installation:

```bash
arjun -h
```

> **What is Arjun?** Arjun discovers hidden HTTP parameters within API endpoints. While Kiterunner finds hidden endpoints, Arjun finds hidden parameters within those endpoints — for example, undocumented fields like `?admin=true` or `?debug=true` that developers left in production.

---

### OWASP ZAP

```bash
sudo apt install zaproxy -y
```

Once installed, launch ZAP:

```bash
zaproxy
```

When ZAP opens, navigate to **Manage Add-ons** (`Ctrl+U`) and apply updates for:
- **Fuzzer**
- **OpenAPI Support**

> **What is OWASP ZAP?** ZAP (Zed Attack Proxy) is an automated security scanner built by the Open Web Application Security Project. Unlike Burp Suite which requires manual operation, ZAP can automatically crawl a target and test for vulnerabilities including SQL injection, XSS, broken authentication, and more.

---

### Wfuzz

Wfuzz comes pre-installed with Kali. Verify it is available:

```bash
wfuzz -h
```

> **What is Wfuzz?** Wfuzz is a web fuzzer that replaces marked positions in a URL or request with values from a wordlist. It is used to discover hidden endpoints, valid user IDs, injection points, and more. More flexible than Kiterunner but less API-specific.

---

## Step 4 — Configure Burp Suite

### Install Jython

Jython is a Python interpreter for Java that is required by the Autorize extension.

1. Visit [https://www.jython.org/download.html](https://www.jython.org/download.html)
2. Download the **Jython Standalone JAR** file
3. Save it to your Downloads folder

Now add it to Burp Suite:

1. Open Burp Suite
2. Go to **Extensions → Extensions Settings**
3. Under **Python Environment**, click **Select file**
4. Navigate to your Downloads folder and select the Jython `.jar` file

---

### Install Autorize Extension

1. In Burp Suite go to **Extensions → BApp Store**
2. Search for **Autorize**
3. Click **Install**

> **What is Autorize?** Autorize is a Burp Suite extension that automatically tests for broken access control (BOLA/IDOR) vulnerabilities. It intercepts requests made by one user and replays them with another user's token, reporting whether unauthorised access is possible.

---

### Install FoxyProxy

FoxyProxy is a Firefox extension that routes browser traffic through a proxy of your choice.

1. Open Firefox
2. Navigate to [https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)
3. Click **Add to Firefox** and install it

Now configure two proxies:

1. Click the FoxyProxy fox icon in the Firefox toolbar
2. Click **Options**
3. Click **Add** and fill in the following for the first proxy:

| Field | Value |
|---|---|
| Title | BurpSuite |
| Hostname | 127.0.0.1 |
| Port | 8080 |

4. Click **Add** again for the second proxy:

| Field | Value |
|---|---|
| Title | Postman |
| Hostname | 127.0.0.1 |
| Port | 5555 |

5. Click **Save**

---

### Import Burp Certificate into Firefox

This allows Firefox to trust Burp Suite as a certificate authority so it can intercept HTTPS traffic without throwing security errors.

1. Make sure Burp Suite is running
2. In FoxyProxy select **BurpSuite** to route Firefox traffic through Burp
3. In Firefox navigate to `http://burpsuite`
4. Click **CA Certificate** to download the certificate
5. In Firefox open **Settings → search "certificates" → View Certificates**
6. Click the **Authorities** tab → **Import**
7. Select the downloaded certificate file
8. Tick **Trust this CA to identify websites**
9. Click **OK**

**Why this is necessary:** Burp Suite acts as a man-in-the-middle between your browser and the internet. When it intercepts HTTPS traffic it presents its own certificate instead of the website's. Without this step Firefox would reject that certificate as untrusted and block all HTTPS traffic through Burp.

---

### Import MITMweb Certificate into Firefox

MITMweb is a separate proxy tool used later in the course for converting captured traffic into API documentation.

1. Close Burp Suite (both tools cannot use port 8080 simultaneously)
2. Run MITMweb from the terminal: `mitmweb`
3. In FoxyProxy make sure **BurpSuite** proxy is still selected (port 8080)
4. In Firefox navigate to `http://mitm.it`
5. Under the **Firefox** section click **Get mitmproxy-ca-cert.pem**
6. Follow the same certificate import process as above (Settings → Certificates → Authorities → Import)

---

## Step 5 — Download Wordlists

Wordlists are used by tools like Kiterunner and Wfuzz to brute force endpoints and parameters.

Navigate to `/opt` and download both wordlist collections:

```bash
cd /opt

# SecLists — general purpose wordlists used across all pentesting tools
sudo wget -c https://github.com/danielmiessler/SecLists/archive/master.zip -O SecList.zip \
&& sudo unzip SecList.zip \
&& sudo rm -f SecList.zip

# Hacking-APIs — wordlists specifically built for API pentesting
sudo wget -c https://github.com/hAPI-hacker/Hacking-APIs/archive/refs/heads/main.zip -O HackingAPIs.zip \
&& sudo unzip HackingAPIs.zip \
&& sudo rm -f HackingAPIs.zip
```

> **SecLists** is a community maintained collection of wordlists covering usernames, passwords, URLs, API paths, fuzzing payloads and more. **Hacking-APIs** is a collection built specifically for API security testing by the HAWD bootcamp team.

---

## Tool Reference Guide

A quick reference for when to use each tool during a pentest engagement:

```
Phase 1 — Recon
├── Kiterunner   → discover hidden API endpoints
└── Arjun        → discover hidden parameters within endpoints

Phase 2 — Scanning
├── OWASP ZAP    → automated vulnerability scan
└── Wfuzz        → fuzz specific inputs and parameters

Phase 3 — Exploitation
├── Burp Suite   → intercept, modify and replay requests manually
├── Postman      → craft and send specific API requests
└── jwt_tool     → attack JWT authentication tokens

Phase 4 — Reporting
└── Document all findings with evidence
```

---

## Troubleshooting

### VM crashed mid-upgrade (black screen on boot)

If VirtualBox crashes during a `sudo apt upgrade` the package manager may be left in a broken state. Boot into a terminal session using `Ctrl+Alt+F2` and run:

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install -y
sudo reboot
```

### Package lock error — "held by process XXXX"

Another apt process is running in the background. Find and kill it:

```bash
sudo lsof /var/lib/dpkg/lock-frontend
sudo kill <PID>
```

Then retry your command.

### Kiterunner build fails — "go: not found"

Go is not installed. Install it first:

```bash
sudo apt install golang-go -y
```

Then retry `sudo make build` inside the kiterunner directory.

### Burp certificate not working — HTTPS sites blocked in Firefox

Make sure you:
1. Had Burp Suite running when you visited `http://burpsuite`
2. Had FoxyProxy set to BurpSuite when downloading the certificate
3. Imported the certificate under the **Authorities** tab (not Personal)
4. Ticked **Trust this CA to identify websites**

---

*This documentation is part of my HAWD API Security Bootcamp learning journey. For the full write-up and weekly updates visit my [Medium profile](#).*
