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
  Interfacing Options - VNC - Enable
  Advanced - Expand filesystem 
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
* `curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
* `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
* `sudo apt-get update && sudo apt-get install yarn`

## Install code-server
* `yarn global add code-server --unsafe-perm` (this might take upwards of 10 minutes - build step takes a while)
* Add yarn to $PATH: `export PATH=$PATH:~/.yarn/bin`  >>>>>>>FIND WAY TO ADD ON REBOOT
* `echo $PATH`
* Get code-server password: `cat .config/code-server/config.yaml`
* `code-server --host 0.0.0.0`    >>>>>>>FIND WAY TO START ON REBOOT
You can stop here if you only want to access on your local network!

## Install nginx & firewall
* `sudo apt update`
* `sudo apt install nginx`
* `systemctl status nginx`
* `sudo apt-get install ufw -y`
* `sudo ufw allow 'Nginx HTTP'`
* `sudo ufw allow ssh`
* --sudo ufw allow 8080
* --sudo ufw allow http?
* --sudo ufw allow 80?
* `sudo ufw allow 'Nginx HTTP'`
* `sudo ufw enable`
* `sudo ufw reload`
* `sudo ufw status`

## Make accessible over the internet
* Forward port for Pi IP on network for 8080
* Map new domain A record (ie codeserver.xxxxx.com) to external IP
* `sudo nano /etc/nginx/nginx.conf`
* Uncomment server_names_hash_bucket_size and update to 32, save
* Create nginx config: `sudo nano /etc/nginx/sites-available/code-server.conf`
```
server {
    listen 80;
    listen [::]:80;

    server_name code-server.your-domain;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```
* Create symlink: `sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf`
* Validate: `sudo nginx -t`
* Restart nginx: `sudo systemctl restart nginx`

## Install Certbot
* sudo apt install python-certbot-nginx
* sudo ufw allow https
* sudo ufw reload

****add remaining instructions

