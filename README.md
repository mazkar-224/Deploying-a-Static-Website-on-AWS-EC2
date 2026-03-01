# 🚀 Static Website Deployment on AWS EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)
![Apache](https://img.shields.io/badge/Web_Server-Apache-red?logo=apache)
![Linux](https://img.shields.io/badge/OS-Red_Hat_Linux-lightgrey?logo=redhat)
![Git](https://img.shields.io/badge/VCS-Git-black?logo=git)
![License](https://img.shields.io/badge/License-MIT-blue)

> End-to-end deployment of a static website on AWS EC2 (Red Hat Linux) using Apache web server, configured with Security Groups, SSH key-pair authentication, and GitHub as the file transfer mechanism.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Phase 1 — Push Code to GitHub](#phase-1--push-code-to-github)
- [Phase 2 — Launch EC2 Instance](#phase-2--launch-ec2-instance)
- [Phase 3 — Connect via SSH](#phase-3--connect-via-ssh)
- [Phase 4 — Configure the Server](#phase-4--configure-the-server)
- [Phase 5 — Deploy Website Files](#phase-5--deploy-website-files)
- [Phase 6 — Start Apache & Go Live](#phase-6--start-apache--go-live)
- [Common Issues & Fixes](#-common-issues--fixes)
- [Next Steps](#-next-steps)

---

## 📖 Project Overview

This project demonstrates how to deploy a static HTML website to a cloud server using AWS EC2. It covers the complete workflow from local development to a publicly accessible live URL — including server provisioning, network security configuration, SSH access, and Apache web server setup.

**Tech Stack:**

| Tool | Purpose |
|---|---|
| AWS EC2 | Cloud virtual machine to host the website |
| Red Hat Linux | Operating system on the EC2 instance |
| Apache (httpd) | Web server that serves the static files |
| Git & GitHub | Version control and file transfer to the server |
| SSH | Secure remote access to the EC2 instance |
| AWS Security Groups | Firewall rules controlling inbound/outbound traffic |

---

## 🏗 Architecture

```
Your Local Machine
       │
       │  git push
       ▼
   GitHub Repo
       │
       │  git clone (over internet)
       ▼
  AWS EC2 Instance  ◄──── SSH Access (port 22, .pem key)
  (Red Hat Linux)
       │
       │  files copied to /var/www/html/
       ▼
  Apache Web Server
       │
       │  HTTP (port 80)
       ▼
  Public Internet  ──► Users access via EC2 Public IP
```

---

## ✅ Prerequisites

Before you begin, make sure you have:

- [ ] An **AWS account** (free tier is sufficient — [Sign up here](https://aws.amazon.com/free/))
- [ ] A **GitHub account** with your static website files in a repository
- [ ] A terminal with **SSH client** installed (Linux/Mac: built-in | Windows: use Git Bash or PuTTY)
- [ ] Basic familiarity with Linux commands (`cd`, `ls`, `cp`)

---

## Phase 1 — Push Code to GitHub

If your website files are not already on GitHub, follow these steps from your local machine.

**1. Navigate to your project folder:**
```bash
cd /path/to/your/project
```

**2. Initialize a Git repository:**
```bash
git init
git add .
git commit -m "initial commit - added website files"
```

**3. Connect to your GitHub remote and push:**
```bash
git remote add origin https://github.com/<your-username>/<your-repo-name>.git
git push -u origin master
```

> 💡 `origin` is just a short alias for your GitHub repo URL so you don't have to type it every time.

---

## Phase 2 — Launch EC2 Instance

**1. Log in to AWS Console** → Search for **EC2** → Click **Launch Instance**

**2. Configure the instance with these settings:**

**Name:** Give your instance a meaningful name (e.g., `my-website-server`)

**AMI (Operating System):**
- Select **Red Hat Enterprise Linux** (free tier eligible)

**Instance Type:**
- Select **t2.micro** (1 vCPU, 1GB RAM — free tier eligible)

**Key Pair (SSH Authentication):**
- Click **Create new key pair**
- Give it a name (e.g., `my-website-key`)
- Select **`.pem`** format (for Linux/Mac) or **`.ppk`** for PuTTY (Windows)
- Click **Create key pair** — the file will auto-download
- ⚠️ **Store this file safely. You cannot download it again. Without it, you cannot access your server.**

**Network Settings (Security Group):**

Click **Edit** and add the following inbound rules:

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | My IP *(recommended)* or Anywhere | Remote server access |
| HTTP | TCP | 80 | Anywhere (0.0.0.0/0) | Public website access |

> 🔒 **Security tip:** Set SSH source to **My IP** instead of Anywhere. This ensures that even if your private key is compromised, only connections from your IP are allowed in.

**Auto-assign Public IP:**
- Make sure this is **Enabled** — without it your server won't have a public address

**Storage:**
- Default 10 GB EBS SSD is sufficient for a static website

**3. Click Launch Instance.**

AWS will provision your server. It takes about 30–60 seconds to reach the **Running** state.

**4. Note your Public IP:**

Go to **EC2 → Instances** → click your instance → copy the **Public IPv4 address**. You'll need this throughout the setup.

---

## Phase 3 — Connect via SSH

**1. Open your terminal and navigate to where your `.pem` file is stored:**
```bash
cd ~/Downloads
```

**2. Restrict the key file permissions (required — SSH will refuse to connect otherwise):**
```bash
chmod 400 my-website-key.pem
```

**3. SSH into your EC2 instance:**
```bash
ssh -i "my-website-key.pem" ec2-user@<your-public-ip>
```

- Replace `<your-public-ip>` with the IP you copied from the AWS console
- Type `yes` when asked to confirm the fingerprint
- You are now inside your cloud server ✅

---

## Phase 4 — Configure the Server

**1. Switch to root user** (required for installing packages and system changes):
```bash
sudo su -
```

**2. Update all system packages:**
```bash
yum update -y
```
> The `-y` flag auto-confirms all prompts so you don't have to type yes repeatedly.

**3. Install Git:**
```bash
yum install git -y
```

**Verify installation:**
```bash
git --version
```

---

## Phase 5 — Deploy Website Files

**1. Clone your GitHub repository onto the server:**
```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
```

**2. Navigate into the cloned folder:**
```bash
cd <your-repo-name>
```

**3. If your files are on the `master` branch, switch to it:**
```bash
git checkout master
git branch   # verify you're on the right branch
```

**4. Install Apache web server:**
```bash
yum install httpd -y
```

> ℹ️ Apache only serves files placed in `/var/www/html/`. Files anywhere else won't be accessible publicly. This is Apache's **document root** — think of it as the public folder of your web server.

**5. Copy your website files to Apache's document root:**
```bash
# Copy the main HTML file
cp index.html /var/www/html/

# Copy folders using -r flag (recursive, for directories)
cp -r assets/ /var/www/html/
cp -r images/ /var/www/html/
cp -r errors/ /var/www/html/
```

> ⚠️ Always use the `-r` flag when copying folders. Without it, the `cp` command only works on single files and will throw an error for directories.

**6. Verify everything is in place:**
```bash
ls /var/www/html/
```

You should see `index.html` and all your asset folders listed.

---

## Phase 6 — Start Apache & Go Live

**1. Start the Apache web server:**
```bash
systemctl start httpd
```

**2. Enable Apache to auto-start on server reboot:**
```bash
chkconfig httpd on
```

**3. Verify Apache is running:**
```bash
systemctl status httpd
```
You should see `active (running)` in green.

**4. Open your browser and visit:**
```
http://<your-public-ip>
```

Your website should be live and accessible to anyone on the internet 🎉

---

## 🐛 Common Issues & Fixes

**Website not loading in browser:**
- Make sure you're using `http://` not `https://` — Apache is only configured for HTTP (port 80) in this setup. Modern browsers sometimes auto-redirect to HTTPS.
- Confirm port 80 is allowed in your EC2 Security Group inbound rules.

**SSH connection refused or timeout:**
- Check that port 22 is open in your Security Group.
- Confirm you're using the correct `.pem` file and public IP.
- Ensure `chmod 400` has been applied to the key file.

**Permission denied when running yum/systemctl:**
- You need to be root. Run `sudo su -` first.

**Git clone not working:**
- Confirm Git is installed: `git --version`
- Check that your repo URL is correct and the repo is public.

**Files copied but website still not showing:**
- Confirm Apache is running: `systemctl status httpd`
- Confirm files are in `/var/www/html/` and not a subfolder inside it.

---

## 🔭 Next Steps

Once comfortable with this setup, here's how to level up:

- **Add HTTPS** — Get a free SSL certificate using Let's Encrypt + Certbot
- **Host a Node.js or Python app** — Install Node.js (`yum install nodejs`) or Python and run a dynamic application
- **Containerize with Docker** — Package your app into a Docker container for portable deployments
- **Automate with Terraform** — Replace the manual EC2 setup with Infrastructure as Code
- **Set up CI/CD** — Auto-deploy on every GitHub push using Jenkins or GitHub Actions
- **Use a Domain Name** — Point a custom domain to your EC2 public IP via Route 53 or any DNS provider

---

## 👤 Author

**Your Name**
[LinkedIn](https://linkedin.com) · [GitHub](https://github.com)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
