# Install Gitea Git service on Ubuntu 22.04|20.04|18.04|16.04

## Step 1: Create a git system user

```
sudo adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
```
## Step 2: Install MariaDB database server

```
sudo apt update
sudo apt -y install mariadb-server
```

##### Create a database for Gitea.
```
sudo mysql -u root -p
CREATE DATABASE gitea;
GRANT ALL PRIVILEGES ON gitea.* TO 'gitea'@'localhost' IDENTIFIED BY "giteapassword";
FLUSH PRIVILEGES;
QUIT;
```

## Step 3: Install Gitea on Ubuntu
```
curl -fsSL https://github.com/go-gitea/gitea/releases/download/v1.19.4/gitea-1.19.4-linux-amd64 -O /home/ubuntu 

chmod +x gitea-*-linux-amd64

sudo mv gitea-*-linux-amd64 /usr/local/bin/gitea
```
###### You can confirm version installed using.

gitea --version

#### Create the required directory structure.
```
sudo mkdir -p /etc/gitea /var/lib/gitea/{custom,data,indexers,public,log}

sudo chown git:git /var/lib/gitea/{data,indexers,log}

sudo chmod 750 /var/lib/gitea/{data,indexers,log}

sudo chown root:git /etc/gitea

sudo chmod 770 /etc/gitea
```

#### Create a systemd service unit
sudo vim /etc/systemd/system/gitea.service

#### Configure the file to set User, Group and WorkDir.
```
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
After=mysql.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
#LimitMEMLOCK=infinity
#LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web -c /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
# If you want to bind Gitea to a port below 1024 uncomment
# the two values below
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
### Reload systemd and restart service
```
sudo systemctl daemon-reload
sudo systemctl restart gitea
```

#### enable the service to start on boot

sudo systemctl enable gitea

### status
systemctl status gitea

## Step 4: Configure Nginx Proxy

sudo apt -y install nginx
#### If ufw is enabled, allow http and https ports.
```
for i in http https; do
 sudo ufw allow $i
done
```
#### Create Nginx configuration file for Gitea

sudo vim /etc/nginx/conf.d/gitea.conf

#### Paste below data into the file created.

```
erver {
    listen 80;
    server_name git.example.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```
### Note:
You can now access Gitea through your IP or domain on port 3000. In the initial configuration, the database password is set to "gitea," as mentioned in the above steps. You can change it if you wish. After that, you can create a root user from the admin account settings and start using the application.