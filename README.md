# Install code-server (VSCode) on Raspberry Pi

You can use this guide to configure a Raspberry Pi to run VScode via code-server and access on a remote network on any device (iPads, yay!), mapped to a domain name of your choosing. 

## Resources
https://github.com/cdr/code-server/blob/master/doc/install.md
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04
https://www.digitalocean.com/community/tutorials/how-to-set-up-the-code-server-cloud-ide-platform-on-ubuntu-18-04

## Set up Raspberry Pi
Recommended model: at least 3B+, in case you need to use ethernet port/USB. Zero W is not recommended due to low memory & ARMV6

* Plug SD card into computer, install Raspbian via Raspberry Pi Imager (https://www.raspberrypi.org/downloads/)
* Make it headless if connecting via wifi (https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html)
* Eject, put SD card into Pi, power on
* Login: `ssh pi@internal-pi-ip`
* Change the password: `passwd`
* Update some Pi settings: `sudo raspi-config`
  * Interfacing Options - VNC - Enable
  * Advanced - Expand Filesystem 
* `sudo reboot`
* `sudo apt update`
* `sudo apt upgrade -y`  (will take a few minutes)
* Install VNC to access desktop remotely: `sudo apt install realvnc-vnc-server realvnc-vnc-viewer -y`
* Create non superuser - `sudo adduser yournewuser`

## Install node 12 & dependencies
There are apparently issues with other versions of node on ARMV6/V7 devices, so be sure to use node 12:
```
sudo apt-get install -y \
  build-essential \
  pkg-config \
  libx11-dev \
  libxkbfile-dev \
  libsecret-1-dev
```
* `curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -`
* `sudo apt-get install -y nodejs`
* Verify node install: `node -v`
* Verify npm: `npm -v`
* Install yarn: 
  * `curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
  * `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
  * `sudo apt-get update && sudo apt-get install yarn`

## Install code-server
* `yarn global add code-server --unsafe-perm`
  * This might take upwards of 10 minutes - the build takes a while
* Add yarn to $PATH: `export PATH=$PATH:~/.yarn/bin`  >>>>>>>FIND WAY TO ADD ON REBOOT
* `echo $PATH`
* `code-server --host 0.0.0.0`    >>>>>>>FIND WAY TO START ON REBOOT
* Open a new terminal window, SSH in, & get code-server password: `cat .config/code-server/config.yaml`
* Try logging in to codes-server via another machine's browser using: `pi_local_ip:8080`
> You can stop here if you only want to access on your local network!

## Install nginx & firewall
* `sudo apt update`
* `sudo apt install nginx -y`
* Verify nginx is working: `systemctl status nginx`
* `sudo apt-get install ufw -y`
* `sudo ufw allow ssh`
* no--sudo ufw allow 8080
* no--sudo ufw allow http?
* no--sudo ufw allow 80?
* `sudo ufw allow 'Nginx HTTP'`
* `sudo ufw enable`
* `sudo ufw reload`
* `sudo ufw status`

## Make accessible over the internet
* Forward port for Pi IP on your network to 8080
* Map custom domain A record (ie codeserver.mydomain.com) to your external-facing IP
* Update nginx config: `sudo nano /etc/nginx/nginx.conf`
  * Uncomment `server_names_hash_bucket_size` and update value to 64, save
* Create nginx code-server config: `sudo nano /etc/nginx/sites-available/code-server.conf`
```
server {
    listen 80;
    listen [::]:80;

    server_name codeserver.mydomain.com;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```
* Create symlink: `sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf`
* `sudo reboot`
* Validate: `sudo nginx -t`
* Restart nginx: `sudo systemctl restart nginx`

## Install Certbot (not working yet)
* `sudo apt install python-certbot-nginx -y`
* `sudo ufw allow https`
* `sudo ufw reload`
* Request a cert; provide your email: `sudo certbot --nginx -d codeserver.mydomain.com`
  * Select option 1 to disable https auto redirect (in case there are any issues)
