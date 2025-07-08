# Ubuntu VM Setup with SSH & Nginx

This guide outlines the basic configuration of an Ubuntu VM for secure SSH access using key-based authentication, as well as the installation and configuration of the Nginx web server. It also covers Git setup and linking an SSH key to GitHub.

Ideal as a starting point for personal server projects or for reuse in similar setups.

---

## Prerequisites
Before starting, make sure you have the following:

- ✅ A running Ubuntu server (e.g. on a cloud provider or local VM)

- ✅ Root or sudo access to the server

- ✅ A local machine with SSH installed

- ✅ A GitHub account (for optional Git integration)

- ✅ Basic understanding of the Linux terminal and file structure

Optional but recommended:

-  Familiarity with editing configuration files (nano, vim, or similar)

-  Understanding of SSH key-based authentication

---

# Table of Contents

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

## Quickstart

Minimal steps to get SSH login running securely:

1. **Generate SSH Key**  
   ```bash
   mkdir -p ~/.ssh/dso
   ssh-keygen -t ed25519 -f ~/.ssh/dso/dso_ed25519
   ```

2. **Copy SSH key to server**  
   ```bash
   ssh-copy-id -i ~/.ssh/dso/dso_ed25519.pub NAME@IPAdresse
   ```

3. **Initial login with password (only once)**  
   ```bash
   ssh NAME@IPAdresse
   ```

4. **Login using SSH key**  
   ```bash
   ssh -i ~/.ssh/dso/dso_ed25519 NAME@IPAdresse
   ```

---

## 1. Create SSH Key

> Generate a new SSH key for secure, passwordless authentication.

```bash
mkdir -p ~/.ssh/dso
ssh-keygen -t ed25519
# Target path: ~/.ssh/dso/dso_ed25519
```

---

## 2. First login to the server with password

> Login manually to initialize the SSH trust and test access.

```bash
ssh NAME@IP
# -> Confirm with "yes"
# -> Enter password: ***********
logout
```

---

## 3. Add SSH key using `ssh-copy-id`

> Upload your public SSH key to the server to allow key-based login.

```bash
ssh-copy-id -i ~/.ssh/dso/dso_ed25519.pub NAME@IP
# -> Enter password: ***********

ssh -i ~/.ssh/dso/dso_ed25519 NAME@IP

# Check if everything is correct
ls -al ~/
ls -al ~/.ssh/
cat ~/.ssh/authorized_keys
```

---

## 4. Disable password authentication

> Enhance server security by disabling password login completely.

```bash
sudo nano /etc/ssh/sshd_config
# Change:
# PasswordAuthentication yes
# to:
PasswordAuthentication no

sudo systemctl restart ssh.service
logout
```

### Test:

```bash
ssh -i ~/.ssh/dso/dso_ed25519 NAME@IP  # should work

logout

ssh -o PubKeyAuthentication=no NAME@IP  # should fail
```

---

## 5. Install & configure Nginx & Git, and set up an alternative HTML page

> Set up Nginx as a web server and prepare a custom index page. Also install Git for version control.

```bash
# Update all packages
sudo apt update

# Install nginx
sudo apt install nginx -y

# Install git
sudo apt install git -y

systemctl status nginx

# Create directory and HTML file
sudo mkdir -p /var/www/html
sudo touch /var/www/html/alternate-index.html
```

---

### Adjust Nginx configuration

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

> Access the server’s IP to confirm Nginx is working.

```
http://IPAdress
```

---

## 7. Configure Git and add SSH key to GitHub

> Set Git user info and upload a new SSH key to GitHub for secure repo access.

```bash
git config --global user.name "cyborg-s"
git config --global user.email "EMAIL"

ssh-keygen -t ed25519 -C EMAIL
# Target path: ~/.ssh/git/git_ed25519
```

> Copy the contents of the file `git_ed25519.pub` into GitHub under  
> **Settings → SSH and GPG keys → New SSH key**

---
