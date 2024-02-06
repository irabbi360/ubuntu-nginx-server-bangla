# Setup Guide for Nginx SSL with Let's Encrypt

Guide for setting up Nginx on Ubuntu with Universal Firewall (ufw) and SSL with Let's Encrypt

## Prerequisites
To follow this tutorial, you will need:
- One Ubuntu 20.04 server set up by following this initial server setup for Ubuntu 20.04 tutorial, including a sudo-enabled non-root user and a firewall.
- A registered domain name. This tutorial will use `example.com` throughout. You can purchase a domain name from Namecheap, get one for free with Freenom, or use the domain registrar of your choice.
- Both of the following DNS records set up for your server. If you are using DigitalOcean, please see our DNS documentation for details on how to add them.
  - An A record with `example.com` pointing to your server’s public IP address.
  - An A record with `www.example.com` pointing to your server’s public IP address.
- Nginx installed by following How To Install Nginx on Ubuntu 20.04. Be sure that you have a server block for your domain. This tutorial will use `/etc/nginx/sites-available/example.com` as an example.

## Step 1 — Installing Certbot

```bash
# Add the certbot
sudo apt install certbot python3-certbot-nginx
```
Certbot is now ready to use, but in order for it to automatically configure SSL for Nginx, we need to verify some of Nginx’s configuration.

## Step 2 — Confirming Nginx’s Configuration
Find the existing `server_name` line and replace it with your domain name. It is important that you use the same server names that you will use with Let's Encrypt. These must match or certbot will not know which server block in the configuration file to update.
```bash
sudo nano /etc/nginx/sites-available/example.com
```
Find the existing server_name line. It should look like this:
```bash
        "/etc/nginx/sites-available/example.com"
...
server_name example.com www.example.com;
...
```
If it does, exit your editor and move on to the next step.

Then save the file, quit your editor, and verify the syntax of your configuration edits:
```bash
sudo nginx -t
```
If you get an error, reopen the server block file and check for any typos or missing characters. Once your configuration file’s syntax is correct, reload Nginx to load the new configuration:
```bash
sudo systemctl reload nginx
```
Certbot can now find the correct server block and update it automatically.

Next, let’s update the firewall to allow HTTPS traffic.

## Step 3 — Set up UFW / Allowing HTTPS Through the Firewall
You can see the current setting by typing:
```bash
sudo ufw status
```
It will probably look like this, meaning that only HTTP traffic is allowed to the web server:
```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

To additionally let in HTTPS traffic, allow the Nginx Full profile and delete the redundant Nginx HTTP profile allowance:

```bash
# Note that if you are using a non-standard port you can specify a port number
# sudo ufw allow 2222

sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```
Check status of ufw with `sudo ufw status`, you should get output similar to this
```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

Next, let’s run Certbot and fetch our certificates.

## Step 4 — Obtaining an SSL Certificate
Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:
```bash
sudo certbot --nginx -d example.com -d www.example.com
```
This runs `certbot` with the `--nginx` plugin, using `-d` to specify the domain names we’d like the certificate to be valid for.

If this is your first time running `certbot`, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.

If the installation runs successfully, `certbot` will ask you if you want to redirect all traffic to https or not. You should select the option to redirect all traffic to https.
```
Output
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Certificates have now been registered, installed, and loaded. Try loading your site using https.
```
Output
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-08-18. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Let’s finish by testing the renewal process.

## Step 5 - Set up SSL Cert Auto Renewal

Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The `certbot` package we installed takes care of this for us by adding a systemd timer that will run twice a day and automatically renew any certificate that’s within thirty days of expiration.

You can query the status of the timer with `systemctl`:

```bash
sudo systemctl status certbot.timer
```
```
Output
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Mon 2020-05-04 20:04:36 UTC; 2 weeks 1 days ago
    Trigger: Thu 2020-05-21 05:22:32 UTC; 9h left
   Triggers: ● certbot.service
```

To test the renewal process, you can do a dry run with `certbot`:
```bash
sudo certbot renew --dry-run
```

Should there be no errors, everything is in order. Certbot will take care of renewing your certificates and reloading Nginx as needed to apply the changes. In the event of a failure in the automated renewal process, you’ll receive an email alert from Let’s Encrypt at the address you provided, notifying you of an impending certificate expiration.