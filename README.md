# Configuration of my Ubuntu VM

## 1. Create SSH Key

```bash
mkdir -p ~/.ssh/dso
ssh-keygen -t ed25519
# Target path: ~/.ssh/dso/dso_ed25519
```

## 2. First login to the server with password

```bash
ssh NAME@IP
# -> Confirm with "yes"
# -> Enter password: ***********
logout
```

## 3. Add SSH key using `ssh-copy-id`

```bash
ssh-copy-id -i ~/.ssh/dso/dso_ed25519.pub NAME@IP
# -> Enter password: ***********

ssh -i ~/.ssh/dso/dso_ed25519 NAME@IP

# Check if everything is correct
ls -al ~/
ls -al ~/.ssh/
cat ~/.ssh/authorized_keys
```

## 4. Disable password authentication

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

## 5. Install & configure Nginx & Git, and set up an alternative HTML page

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

### Adjust Nginx configuration

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

### Example HTML file: `alternate-index.html`

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

## 6. Verify in browser

> Access via IP address:

```
http://IP
```

## 7. Configure Git and add SSH key to GitHub

```bash
git config --global user.name "cyborg-s"
git config --global user.email "EMAIL"

ssh-keygen -t ed25519 -C EMAIL
# Target path: ~/.ssh/git/git_ed25519
```

> Copy the contents of the file `git_ed25519.pub` into GitHub under  
> **Settings → SSH and GPG keys → New SSH key**
