# Install code-server (VSCode) on Raspberry Pi

## Resources
https://github.com/cdr/code-server/blob/master/doc/install.md
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04
https://www.digitalocean.com/community/tutorials/how-to-set-up-the-code-server-cloud-ide-platform-on-ubuntu-18-04

## Set up Raspberry Pi
Recommended model: at least 3B+, in case you need to use ethernet port/USB. Zero W is not recommended due to low memory & ARMV6

* Plug SD card into computer, install Raspbian via Raspberry Pi Imager (https://www.raspberrypi.org/downloads/)
* Make it headless if connecting via wifi
* Put SD card into Pi, power on
* ssh pi@internal-pi-ip
* passwd
* sudo raspi-config
  Interfacing Options - VNC - Enable
  Advanced - Expand filesystem 
* sudo reboot
* sudo apt update
* sudo apt upgrade  (will take a few minutes)
* sudo apt install realvnc-vnc-server realvnc-vnc-viewer

## Install node 12 & dependencies
sudo apt-get install -y \
  build-essential \
  pkg-config \
  libx11-dev \
  libxkbfile-dev \
  libsecret-1-dev

curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -
sudo apt-get install -y nodejs
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
node -v
npm -v

## Install code-server
yarn global add code-server
-- get code-server password
code-server


doesnt work:
mkdir ~/code-server
cd ~/code-server
export PATH=$PATH:/usr/local/bin
sudo npm install -g code-server --unsafe-perm
code-server 

Install nginx:
sudo apt update
sudo apt install nginx
systemctl status nginx

Install ufw & firewall rules:
sudo apt-get install ufw -y
sudo ufw allow 'Nginx HTTP'
sudo ufw reload
sudo ufw status

Map new domain A record (ie codeserver.xxxxx.com) to external IP

Create nginx config:
sudo nano /etc/nginx/sites-available/code-server.conf

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

Create symlink: sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf
Validate: sudo nginx -t
sudo systemctl restart nginx

Install Certbot:
sudo apt install python-certbot-nginx
sudo ufw allow https
sudo ufw reload
****add remaining instructions








didnt work:

curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -
sudo apt-get install -y nodejs
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn

mkdir ~/code-server
cd ~/code-server
wget http://69.195.146.38/code-server/code-server-deftdawg-raspbian-9-vsc1.41.1-linux-arm-built.tar.bz2
bzip2 -dc code-server-deftdawg-raspbian-9-vsc1.41.1-linux-arm-built.tar.bz2 | tar xvf -
cp -r code-server-deftdawg-raspbian-9-vsc1.41.1-linux-arm-built/ code-server
sudo cp -r code-server/ /usr/lib/code-server
sudo ln -s /usr/lib/code-server /usr/bin/code-server
sudo nano /lib/systemd/system/code-server.service

[Unit]
Description=code-server
After=nginx.service

[Service]
Type=simple
Environment=PASSWORD=your_password
ExecStart=/usr/bin/code-server --bind-addr 127.0.0.1:8080 --user-data-dir /var/lib/code-server --auth password
Restart=always

[Install]
WantedBy=multi-user.target

sudo mkdir /var/lib/code-server
sudo systemctl start code-server
sudo systemctl status code-server
