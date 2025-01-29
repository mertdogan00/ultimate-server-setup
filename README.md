# Webmin + 2FA
Webmin is a web-based interface for system administration for Unix. Follow these steps to install and configure it on your VPS.

### Steps:
```sh
curl -o webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
sh webmin-setup-repos.sh
apt-get install webmin --install-recommends
```

### Access Webmin:
After installation, access Webmin using your VPS IP address:
```
https://YOUR_VPS_IP_ADDRESS:10000
```

### Configure Firewall:
Run the following commands to allow Webmin through the firewall:
```sh
ufw allow 10000 >>> 2083
ufw allow 2083
```
Now, you can access Webmin using:
```
https://YOUR_VPS_IP_ADDRESS:2083
```

### Additional Security:
- **Two-Factor Authentication (2FA)** is added in Webmin for extra security.

---

## Authentication Setup
To enhance authentication security, install **Google Authenticator** for SSH:

### Install Google Authenticator:
```sh
sudo apt install libpam-google-authenticator
google-authenticator
```

### Configure SSH for 2FA:
Edit the PAM authentication module:
```sh
nano /etc/pam.d/sshd
```
Add the following line:
```
auth required pam_google_authenticator.so
```

### Update SSH Configuration:
```sh
nano /etc/ssh/sshd_config
```
Set:
```
ChallengeResponseAuthentication yes
```

### Restart SSH Service:
```sh
sudo systemctl restart sshd.service
```

---
**Enjoy your secured server setup!** 🔒🚀


---

## Notes:
- Ensure you save your Google Authenticator backup codes.
- Verify Webmin and SSH access before logging out.



## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details

