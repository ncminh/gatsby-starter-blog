---
title: Linux Server Config
date: "2022-11-18T22:12:03.284Z"
description: "Linux Server Config"
---

Install Postgresql

```shell
# install
sudo apt-get install -y postgresql postgresql-contrib

# access
sudo -i -u postgres
```

Config for remote access
```shell
sudo vi /etc/postgresql/12/main/postgresql.conf
# listen_addresses = '*'

sudo vi /etc/postgresql/12/main/pg_hba.confg
# TYPE  DATABASE    USER    ADDRESS     METHOD
host    all         all     0.0.0.0/0   md5
```

Install .NET Core Ubuntu 22

```shell
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```

```shell
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```

If we want into issues like system does not regconize dotnet command, try to reinstall
```shell
    # Removed all .NET packages
    sudo apt remove 'dotnet*'
    sudo apt remove 'aspnetcore*'

    # Deleted PMC repository from APT, by deleting the repo .list file
    sudo rm /etc/apt/sources.list.d/microsoft-prod.list

    sudo apt update

    # install the SDK again
    sudo apt-get update && sudo apt-get install -y dotnet-sdk-6.0
```

NGINX Configuration for .NET, config NGINX default: /etc/nginx/sites-available/default
```shell
##################################################################################### START NGINX CONFIG
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

# This location block fixed assets issue.
    	location ~* /(css|js|lib) {
        	root /var/www/html/wwwroot;
    	}

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
#		try_files $uri $uri/ =404;
		proxy_pass http://localhost:5000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection keep-alive;
		proxy_set_header Host $http_host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwared-Proto $scheme;
	}
}

##################################################################################### END NGINX CONFIG
```

Setup service auto run on start up.
		sudo vi /etc/systemd/system/kestrel-[APP_NAME].service
```shell

######################################################################## START CONFIG KESTREL STARTUP SERVICE
[Unit]
Description=[APPNAME] App running on Ubuntu
[Service]
WorkingDirectory=/var/www/html
ExecStart=/usr/bin/dotnet /var/www/html/[WEB_FILE].dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-[APP_NAME]-prod
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
[Install]
WantedBy=multi-user.target
######################################################################## END KESTREL CONFIG.

```

Enable service startup.
```shell
    cd /etc/systemd/system/ && sudo systemctl enable kestrel-[APP_NAME].service
```

Dotnet EF migration commands.
```shell
    dotnet ef database update -s [path to project file ] -- --environment Production
```