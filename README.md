# Ubuntu VM Setup with SSH & Nginx

This guide outlines the basic configuration of an Ubuntu VM for secure SSH access using key-based authentication, as well as the installation and configuration of the Nginx web server. It also covers Git setup and linking an SSH key to GitHub.

Ideal as a starting point for personal server projects or for reuse in similar setups.

---


# Table of Contents

- [Prerequisites](#prerequisites)
- [Quickstart](#quickstart)
- [1. Create SSH Key](#1-create-ssh-key)
- [2. First login to the server with password](#2-first-login-to-the-server-with-password)
- [3. Add SSH key using `ssh-copy-id`](#3-add-ssh-key-using-ssh-copy-id)
- [4. Disable password authentication](#4-disable-password-authentication)
- [5. Install & configure Nginx & Git](#5-install--configure-nginx--git)
  - [Adjust Nginx configuration](#adjust-nginx-configuration)
  - [Example HTML file](#example-html-file-alternate-indexhtml)
- [6. Verify in browser](#6-verify-in-browser)
- [7. Configure Git and add SSH key to GitHub](#7-configure-git-and-add-ssh-key-to-github)

---

## Prerequisites
Before starting, make sure you have the following:

- ✅ A running Ubuntu server (e.g. on a cloud provider or local VM)

- ✅ Root or sudo access to the server

- ✅ A local machine with SSH installed

- ✅ A GitHub account (for optional Git integration)

Optional but recommended:

-  Familiarity with editing configuration files (nano, vim, or similar)

-  Understanding of SSH key-based authentication

---

## Quickstart
> [!NOTE]
> Minimal steps to get SSH login running securely:

1. **Generate SSH Key**  
```bash
mkdir -p ~/.ssh/dso
```
```bash
ssh-keygen -t ed25519 -f ~/.ssh/dso/dso_ed25519
```

2. **Copy SSH key to server**  
```bash
ssh-copy-id -i ~/.ssh/dso/dso_ed25519.pub "<username>@<your_vm_ip>"
```

3. **Initial login with password (only once)**  
```bash
ssh "<username>@<your_vm_ip>"
```

4. **Login using SSH key**  
```bash
ssh -i ~/.ssh/dso/dso_ed25519 "<username>@<your_vm_ip>"
```

---

## 1. Create SSH Key
> [!NOTE]
> Generate a new SSH key for secure, passwordless authentication.

```bash
mkdir -p ~/.ssh/dso
```
```bash
ssh-keygen -t ed25519
```
> Target path: ~/.ssh/dso/dso_ed25519

---

## 2. First login to the server with password
> [!NOTE]
> Login manually to initialize the SSH trust and test access.

```bash
ssh "<username>@<your_vm_ip>"
```

> -> Confirm with "yes"
> -> Enter password: ***********

```bash
logout
```

---

## 3. Add SSH key using `ssh-copy-id`
> [!NOTE]
> Upload your public SSH key to the server to allow key-based login.

```bash
ssh-copy-id -i ~/.ssh/dso/dso_ed25519.pub "<username>@<your_vm_ip>"
```
> -> Enter password: ***********

```bash
ssh -i ~/.ssh/dso/dso_ed25519 "<username>@<your_vm_ip>"
```
> Check if everything is correct

```bash
ls -al ~/.ssh/
```

```bash
cat ~/.ssh/authorized_keys
```

---

## 4. Disable password authentication
> [!NOTE]
> Enhance server security by disabling password login completely.

```bash
sudo nano /etc/ssh/sshd_config
```
> Change:
> #PasswordAuthentication yes
> to:
```bash
PasswordAuthentication no
```

```bash
sudo systemctl restart ssh.service
```

```bash
logout
```

### Test:

```bash
ssh -i ~/.ssh/dso/dso_ed25519 "<username>@<your_vm_ip>"
```
> should work

```bash
logout
```

```bash
ssh -o PubKeyAuthentication=no "<username>@<your_vm_ip>"
```
> should fail


---

## 5. Install & configure Nginx & Git, and set up an alternative HTML page
> [!NOTE]
> Set up Nginx as a web server and prepare a custom index page. Also install Git for version control.

**Update all packages**
```bash
sudo apt update
```

**Install nginx**
```bash
sudo apt install nginx -y
```

**Install git**
```bash
sudo apt install git -y
```

```bash
systemctl status nginx
```

**Create directory and HTML file**
```bash
sudo mkdir -p /var/www/html
```

```bash
sudo touch /var/www/html/alternate-index.html
```

---

### Adjust Nginx configuration
> [!NOTE]
> Replace the default Nginx config to serve a custom HTML file.

```bash
sudo nano /etc/nginx/sites-enabled/default
```

**Edit content to:**

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index alternate-index.html index.html index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### Example HTML file: `alternate-index.html`
> [!NOTE]
> This file will be served when accessing the server via browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome by DSO</title>
</head>
<body>
    <h1>Hello, Nginx!</h1>
    <p>I have configured my Nginx Server on Ubuntu.</p>
</body>
</html>
```

```bash
sudo service nginx restart
```

---

## 6. Verify in browser
> [!NOTE]
> Access the server’s IP to confirm Nginx is working.

```
http://<your_vm_ip>
```

---

## 7. Configure Git and add SSH key to GitHub
> [!NOTE]
> Set Git user info and upload a new SSH key to GitHub for secure repo access.

```bash
git config --global user.name "<your_github_name"
```

```bash
git config --global user.email "<your_github_email>"
```

```bash
ssh-keygen -t ed25519 -C "<your_email>"
```
> Target path: ~/.ssh/git/git_ed25519

> [!NOTE]
> Copy the contents of the file `git_ed25519.pub` into GitHub under  
> **Settings → SSH and GPG keys → New SSH key**

---
