# 🚀 Intelligent Infrastructure Monitoring System using OpenClaw & AWS EC2

> Deploy an AI-powered monitoring dashboard — from a **single prompt** — that surpasses traditional tools like Prometheus & Grafana with autonomous security testing, vulnerability detection, and real-time reporting.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Part 1 — Launch & Secure Your AWS EC2 Instance](#part-1--launch--secure-your-aws-ec2-instance)
- [Part 2 — Connect to Your Server](#part-2--connect-to-your-server)
- [Part 3 — Install OpenClaw](#part-3--install-openclaw)
- [Part 4 — Integrate with Telegram](#part-4--integrate-with-telegram)
- [Part 5 — Deploy the Monitoring Dashboard via Prompt](#part-5--deploy-the-monitoring-dashboard-via-prompt)
- [Security Best Practices](#security-best-practices)
- [Results & Capabilities](#results--capabilities)
- [Contributing](#contributing)

---

## Project Overview

This project demonstrates deploying **OpenClaw** — an AI-powered gateway orchestration platform — on an AWS EC2 instance to autonomously build and run a live infrastructure monitoring dashboard.

**What makes this different from Prometheus & Grafana?**

| Feature | Prometheus + Grafana | OpenClaw |
|---|---|---|
| Setup | Manual configuration | Single prompt |
| Security Testing | ❌ Not included | ✅ Autonomous |
| Vulnerability Detection | ❌ Manual plugins | ✅ Auto-generated reports |
| AI Intelligence | ❌ None | ✅ Built-in |
| Multi-channel Messaging | ❌ Limited | ✅ Telegram, WhatsApp & more |

---

## Architecture

```
Your Machine
     │
     │  SSH (Port 22 - Key Pair Only)
     ▼
┌─────────────────────────────────┐
│         AWS EC2 Instance        │
│         (Ubuntu 22.04)          │
│                                 │
│  ┌──────────────────────────┐   │
│  │       OpenClaw Engine    │   │
│  │  - Gateway Orchestration │   │
│  │  - AI Model Integration  │   │
│  │  - Persistent Memory     │   │
│  │  - Skills / Plugins      │   │
│  └──────────┬───────────────┘   │
│             │                   │
│  ┌──────────▼───────────────┐   │
│  │   Monitoring Dashboard   │   │
│  │  - CPU / RAM Tracking    │   │
│  │  - Internet Speed        │   │
│  │  - Security Reports      │   │
│  │  - Live Logs             │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘
     │
     │  Telegram Bot Integration
     ▼
  Your Phone / Desktop
```

---

## Prerequisites

Before starting, make sure you have:

- An **AWS Account** (Free Tier works)
- A **Telegram account** (for bot integration)
- Basic knowledge of Linux terminal commands
- SSH client installed on your machine (`ssh` on Mac/Linux, PuTTY or Windows Terminal on Windows)

---

## Part 1 — Launch & Secure Your AWS EC2 Instance

### Step 1.1 — Log in to AWS Console

1. Go to [https://aws.amazon.com/console](https://aws.amazon.com/console)
2. Sign in to your account
3. In the top-right corner, select your preferred **region** (e.g., `ap-south-1` for Mumbai)

---

### Step 1.2 — Launch an EC2 Instance

1. In the search bar, type **EC2** and click on it
2. Click **"Launch Instance"**
3. Fill in the following settings:

| Setting | Value |
|---|---|
| Name | `openclaw-monitor` |
| AMI | Ubuntu Server 22.04 LTS (Free Tier eligible) |
| Instance Type | `t2.micro` (Free Tier) or `t3.small` for better performance |
| Key Pair | Create a new key pair → name it `openclaw-key` → download the `.pem` file |

> ⚠️ **Keep your `.pem` file safe. You cannot download it again.**

---

### Step 1.3 — Configure Security Group (Critical Step)

Under **Network Settings**, click **"Edit"** and configure the following rules:

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | **Your IP only** (`My IP`) | Secure server access |
| Custom TCP | TCP | 8080 | Your IP only | OpenClaw Dashboard |
| Custom TCP | TCP | 3000 | Your IP only | Monitoring UI |

> 🔒 **Never set Source to `0.0.0.0/0` for SSH.** Always restrict to your own IP address.

To find your IP: visit [https://whatismyip.com](https://whatismyip.com) and use `YOUR_IP/32` format.

---

### Step 1.4 — Configure Storage

- Set root volume to **20 GB** (gp3 type — faster and cheaper than gp2)
- Enable **"Delete on termination"** for cost savings on test environments

---

### Step 1.5 — Set Up IAM Role for EC2 (Least Privilege)

1. Go to **IAM → Roles → Create Role**
2. Select **EC2** as the trusted entity
3. Attach only the required policies:
   - `CloudWatchAgentServerPolicy` — for sending metrics
   - `AmazonSSMManagedInstanceCore` — for Systems Manager access (optional)
4. Name the role: `openclaw-ec2-role`
5. Go back to your EC2 instance → **Actions → Security → Modify IAM Role** → attach `openclaw-ec2-role`

---

### Step 1.6 — Set Up AWS Budget Alert

1. Go to **AWS Billing → Budgets → Create Budget**
2. Select **"Cost Budget"**
3. Set monthly budget amount (e.g., `$10`)
4. Add alert: notify at **80%** of budget via email
5. Click **Create Budget**

> 💡 This ensures you never get surprise charges during testing.

---

### Step 1.7 — Launch the Instance

1. Review all settings
2. Click **"Launch Instance"**
3. Wait 1–2 minutes for the instance state to show **"Running"**
4. Copy your **Public IPv4 Address** from the instance details

---

## Part 2 — Connect to Your Server

### On Mac / Linux

```bash
# Move to the folder where you downloaded the .pem file
cd ~/Downloads

# Fix permissions on the key file (required)
chmod 400 openclaw-key.pem

# Connect to your EC2 instance
ssh -i openclaw-key.pem ubuntu@YOUR_PUBLIC_IP
```

### On Windows (PowerShell)

```powershell
# Fix permissions
icacls openclaw-key.pem /inheritance:r /grant:r "$($env:USERNAME):(R)"

# Connect
ssh -i openclaw-key.pem ubuntu@YOUR_PUBLIC_IP
```

Replace `YOUR_PUBLIC_IP` with the IPv4 address from your EC2 dashboard.

---

## Part 3 — Install OpenClaw

### Step 3.1 — Update the Server

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3.2 — Install Dependencies

```bash
# Install required packages
sudo apt install -y curl wget git python3 python3-pip nodejs npm unzip

# Verify installations
python3 --version
node --version
npm --version
```

### Step 3.3 — Install OpenClaw

```bash
# Clone OpenClaw from the official repository
git clone https://github.com/openclaw/openclaw.git

# Navigate into the directory
cd openclaw

# Install dependencies
npm install

# Or using the install script (if available)
curl -fsSL https://openclaw.io/install.sh | bash
```

> 📌 Visit the [OpenClaw official website](https://lnkd.in/diQ2qR7N) for the latest installation method.

### Step 3.4 — Configure OpenClaw

```bash
# Copy the example config
cp config.example.json config.json

# Open and edit the config
nano config.json
```

Set the following fields in `config.json`:

```json
{
  "server": {
    "host": "0.0.0.0",
    "port": 8080
  },
  "ai": {
    "model": "your-preferred-model",
    "api_key": "YOUR_AI_API_KEY"
  },
  "memory": {
    "enabled": true,
    "persistence": "file"
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

Save and exit with `Ctrl+X → Y → Enter`.

### Step 3.5 — Start OpenClaw

```bash
# Start OpenClaw
npm start

# Or run in background (recommended)
nohup npm start > openclaw.log 2>&1 &

# Check it's running
curl http://localhost:8080/health
```

---

## Part 4 — Integrate with Telegram

### Step 4.1 — Create a Telegram Bot

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Enter a name: `OpenClaw Monitor`
4. Enter a username: `openclaw_your_name_bot`
5. Copy the **API Token** provided (e.g., `7123456789:AAF...`)

### Step 4.2 — Add Token to OpenClaw Config

```bash
nano ~/openclaw/config.json
```

Paste your bot token into the `telegram.token` field and save.

### Step 4.3 — Restart OpenClaw

```bash
# Find the running process
ps aux | grep openclaw

# Kill and restart
pkill -f "npm start"
nohup npm start > openclaw.log 2>&1 &
```

### Step 4.4 — Test the Connection

1. Open Telegram and find your bot
2. Send `/start`
3. You should receive a welcome message from OpenClaw ✅

---

## Part 5 — Deploy the Monitoring Dashboard via Prompt

This is where the magic happens. With OpenClaw running and Telegram connected, send a **single prompt** to deploy your entire monitoring dashboard.

### Step 5.1 — Send the Deployment Prompt

Open your Telegram bot and send:

```
Build and deploy a live monitoring dashboard on this server. 
Include real-time CPU usage, RAM usage, internet speed tracking, 
security vulnerability checks, system logging, and generate a 
security recommendations report. Deploy it and keep it running.
```

### Step 5.2 — What OpenClaw Does Automatically

Within minutes, OpenClaw will autonomously:

- ✅ Check system security vulnerabilities
- ✅ Set up CPU and RAM usage monitoring
- ✅ Run internet speed analysis
- ✅ Configure live logging
- ✅ Generate security recommendations report
- ✅ Deploy a live web dashboard
- ✅ Keep everything running in real-time

### Step 5.3 — Access Your Dashboard

Once deployed, open your browser and go to:

```
http://YOUR_PUBLIC_IP:3000
```

You should see your **live monitoring dashboard** — fully autonomous, no manual configuration needed.

---

## Security Best Practices

Follow these rules to keep your server hardened:

```bash
# 1. Restrict SSH access — never leave port 22 open to 0.0.0.0
# Update your Security Group to allow only your IP

# 2. Disable password authentication — key pair only
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd

# 3. Set strict file permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 4. Block unnecessary open ports
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 3000/tcp
sudo ufw deny 80/tcp
sudo ufw deny 443/tcp
sudo ufw status

# 5. Take a snapshot/backup before major changes
# Go to EC2 → Instances → Actions → Image and Templates → Create Image
```

**Additional Security Checklist:**

- [ ] Conduct regular security audits using OpenClaw's built-in security scanner
- [ ] Restrict all access via SSH tunnels only
- [ ] Maintain regular EC2 snapshots / AMI backups
- [ ] Block all unnecessary open ports (HTTP/HTTPS if not needed)
- [ ] Enforce strict Linux file permissions on all config files
- [ ] Deny access to control-plane tools wherever possible
- [ ] Rotate your IAM keys and API tokens regularly

---

## Results & Capabilities

After completing this setup, your system will:

| Capability | Details |
|---|---|
| **CPU Monitoring** | Real-time tracking with alerts |
| **RAM Monitoring** | Usage graphs and threshold warnings |
| **Internet Speed** | Live upload/download speed analysis |
| **Security Scanning** | Automated vulnerability detection |
| **Security Reports** | Structured, auto-generated reports |
| **Live Logging** | System-wide log aggregation |
| **Telegram Alerts** | Instant notifications for critical events |

> 💡 **Compared to Prometheus & Grafana:** OpenClaw delivers a comparable monitoring experience but adds AI-powered security intelligence, automated vulnerability reporting, and zero-config deployment — all from a single prompt.

---

## Contributing

Feel free to fork this repository, open issues, or submit pull requests.  
For setup help or configuration questions, feel free to **DM on LinkedIn**.

For more security information, visit the **[OpenClaw Official Website](https://lnkd.in/diQ2qR7N)**.

---

> Built and tested on AWS EC2 — Ubuntu 22.04 LTS | OpenClaw | Telegram Integration
