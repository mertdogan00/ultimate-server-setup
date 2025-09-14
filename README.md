# 🛡️ All-in-One Server Setup: Admin Panel, SSL, Proxy, Security & More

A full-featured, production-ready deployment guide for [Webmin](https://www.webmin.com/).  
Includes SSL, Nginx reverse proxy, real IP, 2FA, port customization, UFW, user safety, and even fix suggestions for common pitfalls.

---

## 📚 Table of Contents


- [🔧 Installation](#-installation)
- [🧩 Required Packages & Fixes](#-required-packages--fixes)
- [🚪 First Login to Webmin](#-first-login-to-webmin)
- [🌐 Access via Domain or IP](#-access-via-domain-or-ip)
- [🔥 Firewall Setup (UFW)](#-firewall-setup-ufw)
- [🌀 Change Webmin Port](#-change-webmin-port)
  - [Via GUI](#via-gui)
  - [Via miniserv.conf](#via-miniservconf)
- [🔐 Enable 2FA](#-enable-2fa)
  - [Terminal-Based](#terminal-based)
  - [Webmin GUI](#webmin-gui)
- [🌍 Setup Nginx Reverse Proxy](#-setup-nginx-reverse-proxy)
- [🛡️ Show Real Client IPs](#️-show-real-client-ips)
- [🔁 Redirect Settings](#-redirect-settings)
- [🔐 Trusted Referrers](#-trusted-referrers)
- [🔑 Reset Webmin Password](#-reset-webmin-password)
- [🎨 Webmin Custom CSS](#-webmin-custom-css)
- [📜 Let's Encrypt SSL Certificates](#-lets-encrypt-ssl-certificates)
- [📁 Important File Paths](#-important-file-paths)
- [🤝 Contributing](#-contributing)
- [📜 License](#-license)

---

## 🔧 Installation

```bash
curl -o webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
sh webmin-setup-repos.sh
sudo apt update
sudo apt install webmin --install-recommends
```

---

## 🧩 Required Packages & Fixes

👉 For a full manual Linux server setup (ZSH, Docker, hostname, firewall, more), see:  
**[mertdogan00/server-manual-setup](https://github.com/mertdogan00/server-manual-setup)**

Install essential tools:

```bash
sudo apt install nginx certbot python3-certbot-nginx \
libhtml-parser-perl libauthen-pam-perl libio-pty-perl libnet-ssleay-perl \
fail2ban
```

Enable and start services:

```bash
sudo systemctl enable --now nginx
sudo systemctl enable --now fail2ban
sudo systemctl enable --now webmin
```

---

## 🚪 First Login to Webmin

- Access via `https://your-server-ip:10000`
- Browser may show a warning → click **Advanced > Continue**
- Default username: `root`
- Password: your server's root password

---

## 🌐 Access via Domain or IP

- IP access shows self-signed cert warning  
- Domain access with SSL needs proper setup  
- For HTTPS on a custom domain, use Let's Encrypt with Certbot  
  👉 Full guide: **[certbot-self-hosted-ssl](https://github.com/mertdogan00/certbot-self-hosted-ssl)**

---

## 🔥 Firewall Setup (UFW)

Enable and allow required ports:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 10000
sudo ufw allow 2083
sudo ufw enable
sudo ufw status numbered
```

To remove a rule:

```bash
sudo ufw delete [number]
```

---

## 🌀 Change Webmin Port

### Via GUI

- `Webmin > Webmin Configuration > Ports and Addresses`
- Set new port (e.g. 2083), save, restart

### Via `miniserv.conf`

```bash
sudo nano /etc/webmin/miniserv.conf
# Change:
port=10000
# To:
port=2083
sudo systemctl restart webmin
```

---

## 🔐 Enable 2FA

### Terminal-Based

```bash
sudo apt install libpam-google-authenticator
google-authenticator
```

Then:

```bash
sudo nano /etc/pam.d/sshd
auth required pam_google_authenticator.so

sudo nano /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes

sudo systemctl restart sshd
```

---

### Webmin GUI

- Go to: `Webmin > Webmin Configuration > Two-Factor Authentication`
- Choose: TOTP
- Save
- Then: `Webmin > Webmin Users > [username] > Two-Factor Authentication`
- Scan QR & save backup

---

## 🌍 Setup Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name admin.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name admin.example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass https://127.0.0.1:2083/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 🛡️ Show Real Client IPs

In `/etc/webmin/miniserv.conf`:

```
trust_real_ip=127.0.0.1
```

Then:

```bash
sudo systemctl restart webmin
```

---

## 🔁 Redirect Settings

- GUI path: `Webmin > Webmin Configuration > Web Server Options`
- Set:
  - Redirect host: `admin.example.com`
  - Redirect port: `443`
  - ✅ Enable SSL redirection

---

## 🔐 Trusted Referrers

- GUI path: `Webmin > Webmin Configuration > Trusted Referrers`
- Add:
  
  ```
  admin.example.com
  ```

---

## 🔑 Reset Webmin Password

```bash
sudo /usr/share/webmin/changepass.pl /etc/webmin root NEW_PASSWORD
```

---

## 🎨 Webmin Custom CSS
Webmin allows full interface customization. You can directly paste your CSS code into the theme file.

Edit:
```
sudo nano /etc/webmin/authentic-theme/styles.css
```

Add your custom CSS at the bottom:
```css
body {
    background-color: #1e1e1e !important;
}

.login-header {
    color: #ffffff !important;
}
```

✅ Save and refresh your browser to apply the changes instantly.

🎯 Alternatively, copy the provided CSS file to Webmin:
```
sudo cp webmin-custom.css /etc/webmin/authentic-theme/webmin-custom.css
```

Add to the bottom of styles.css:
```css
@import url('webmin-custom.css');
```

🌐 [👉 Download My Custom Webmin CSS](https://github.com/mertdogan00/ultimate-server-setup/blob/main/webmin-custom.css)


---


## 📜 Let's Encrypt SSL Certificates

Need wildcard, DNS challenge, or `certonly` SSL setup?

👉 Visit: **[mertdogan00/certbot-self-hosted-ssl](https://github.com/mertdogan00/certbot-self-hosted-ssl)**

Covers:
- DNS & HTTP challenges
- wildcard certs
- cert-only mode
- automated renewal and hooks

---

## 📁 Important File Paths

| File                             | Purpose                            |
|----------------------------------|------------------------------------|
| `/etc/webmin/miniserv.conf`     | Webmin main config                 |
| `/etc/webmin/config`            | GUI & redirect settings            |
| `/etc/webmin/users/`            | Per-user 2FA and ACLs              |
| `/var/webmin/`                  | Sessions, cache, runtime data      |
| `/etc/letsencrypt/live/`        | SSL certificate files              |

---

## 🤝 Contributing

Got ideas or improvements? Fork and PR are welcome!

---

## 📜 License

MIT License.  
See the [LICENSE](LICENSE) file for full terms.
