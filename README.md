# 🌐 Webmin + 2FA Setup Guide

**Webmin** is a web-based interface for system administration on Unix-like systems. This guide will help you install Webmin, enable Two-Factor Authentication (2FA), and ensure secure access through your firewall and SSH.

---

## 🧱 Installation Steps

```sh
curl -o webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
sh webmin-setup-repos.sh
sudo apt update
sudo apt install webmin --install-recommends
```

---

## 🌐 Access Webmin

Once installed, open Webmin in your browser using your VPS IP address:
```
https://YOUR_VPS_IP_ADDRESS:10000
```

---

## 🔥 Configure Firewall

Open the required ports to allow Webmin traffic:

```sh
sudo ufw allow 10000
sudo ufw allow 2083
```

Optionally, you can forward port `10000` to `2083` if desired via firewall/NAT.

Then access Webmin from:

```
https://YOUR_VPS_IP_ADDRESS:2083
```

---

## 🔒 Additional Security: Enable 2FA (Google Authenticator)

### 📦 Install Google Authenticator

```sh
sudo apt install libpam-google-authenticator
google-authenticator
```

Follow the prompts and save your backup codes.

---

### ⚙️ Configure PAM for SSH

Edit the PAM config:
```sh
sudo nano /etc/pam.d/sshd
```
Add this line:
```
auth required pam_google_authenticator.so
```

---

### ⚙️ Configure SSH Server

Edit your SSH config:
```sh
sudo nano /etc/ssh/sshd_config
```

Ensure the following lines are present:
```
ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes
```

Then restart the SSH service:
```sh
sudo systemctl restart sshd
```

---

## 🧰 Fix Webmin Nginx Module Error (HTML::Entities)

If you encounter this error:
```
Can't locate HTML/Entities.pm in @INC...
```

Install the missing Perl module:

```sh
sudo apt update
sudo apt install libhtml-parser-perl
```

This resolves missing dependency issues for the **Webmin Nginx module**.

---

## 📝 Notes

- Save your Google Authenticator backup codes.
- Before logging out, always test SSH and Webmin login to avoid lockout.
- Use port `2083` only if you specifically remap it — otherwise `10000` is default.

---

## 🪪 License

This project is licensed under the MIT License.  
See the [LICENSE](LICENSE) file for details.
