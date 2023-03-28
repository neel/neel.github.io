title: Installing gitea and woodpecker using binary packages (WIP)
author: Neel Basu (Sunanda Bose)
tags: []
categories: []
date: 2023-03-26 17:53:00
---
#### Create Users

```shell
$ sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git
$ sudo adduser --system --shell /bin/bash --gecos 'Woodpecker CI' --group --disabled-password --home /home/woodpecker woodpecker
$ sudo usermod -aG docker woodpecker
```

#### Binaries

```shell
$ wget https://dl.gitea.com/gitea/1.19.0/gitea-1.19.0-linux-amd64
$ sudo mv gitea-1.19.0-linux-amd64 /usr/local/bin/gitea
$ sudo chmod +x /usr/local/bin/gitea
$ wget https://github.com/woodpecker-ci/woodpecker/releases/download/v0.15.7/woodpecker-agent_0.15.7_amd64.deb
$ wget https://github.com/woodpecker-ci/woodpecker/releases/download/v0.15.7/woodpecker-cli_0.15.7_amd64.deb
$ wget https://github.com/woodpecker-ci/woodpecker/releases/download/v0.15.7/woodpecker-server_0.15.7_amd64.deb
$ sudo apt-get install ./woodpecker-*
```

#### Directories

```shell
$ sudo mkdir -p /var/lib/gitea/{custom,data,log}
$ sudo chown -R git:git /var/lib/gitea/
$ sudo chmod -R 750 /var/lib/gitea/
$ sudo mkdir /etc/gitea
$ sudo chown root:git /etc/gitea
$ sudo chmod 770 /etc/gitea
$ sudo mkdir -p /var/lib/woodpecker
$ sudo chown -R woodpecker:woodpecker /var/lib/woodpecker
$ sudo chmod -R 750 /var/lib/woodpecker/
$ sudo touch /etc/woodpecker.conf
$ sudo chmod 770 /etc/woodpecker.conf
```



## Setup

#### Database

```shell
$ sudo apt-get install postgresql postgresql-contrib
$ sudo -u postgres psql
postgres=# create database git;
postgres=# create user git with encrypted password 'pass';
postgres=# grant all privileges on database git to git;
postgres=# create database woodpecker;
postgres=# create user woodpecker with encrypted password 'pass';
postgres=# grant all privileges on database woodpecker to woodpecker;
postgres=# exit
```

#### nginx

Save the following as `sudo vim /etc/nginx/sites-available/gitea.conf`

```nginx
server {
    listen 80;
    server_name git.YOUR_DOMAIN.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;
        proxy_buffering off;
    }
}
```

Save the following as `sudo vim /etc/nginx/sites-available/woodpecker.conf`
```nginx
server {
    listen 80;
    server_name woodpecker.YOUR_DOMAIN.com;

    location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;

        proxy_pass http://127.0.0.1:8000;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;

        chunked_transfer_encoding off;
    }
}
```

enable both of these configs

```shell
sudo ln -s /etc/nginx/sites-available/gitea.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/woodpecker.conf /etc/nginx/sites-enabled/
```

Restart nginx service

```shell
sudo systemctl restart nginx
```

#### System Services

```shell
$ sudo wget https://raw.githubusercontent.com/go-gitea/gitea/main/contrib/systemd/gitea.service -P /etc/systemd/system/
```

Open `/etc/systemd/system/gitea.service` and uncoment postgresql lines.

Start the service

```shell
$ sudo systemctl start gitea
```

open `http://IP_ADDRESS:3000` and configure the database settings.

Open `/etc/gitea/app.ini` and and change `ROOT_URL` to `git.YOUR_DOMAIN.com`.

```shell
$ sudo systemctl restart gitea
```

Now visit `http://git.YOUR_DOMAIN.com/admin/applications`. 

Add Woodpecker application with Redirect URI `http://woodpecker.YOUR_DOMAIN.com/authorize` and copy the client id and the secret somewhere as we will need then in next steps.

Create file `/etc/systemd/system/woodpecker.service` and paste the following

```
[Unit]
Description=Woodpecker
Documentation=https://woodpecker-ci.org/docs/intro
Requires=network-online.target
After=network-online.target

[Service]
User=woodpecker
Group=woodpecker
EnvironmentFile=/etc/woodpecker.conf
ExecStart=/usr/local/bin/woodpecker-server
RestartSec=5
Restart=on-failure
SyslogIdentifier=woodpecker-server
WorkingDirectory=/var/lib/woodpecker

[Install]
WantedBy=multi-user.target
```

Write the following configuration in `/etc/woodpecker.conf`

```
WOODPECKER_OPEN=true
WOODPECKER_HOST=http://woodpecker.YOUR_DOMAIN.com
WOODPECKER_GITEA=true
WOODPECKER_GITEA_URL=http://git.YOUR_DOMAIN.com
WOODPECKER_GITEA_CLIENT=COPIED_CLIENT_ID
WOODPECKER_GITEA_SECRET=COPIED_SECRET
WOODPECKER_GITEA_SKIP_VERIFY=true
WOODPECKER_DATABASE_DRIVER=postgres
WOODPECKER_DATABASE_DATASOURCE=postgres://woodpecker:password@127.0.0.1:5432/woodpecker?sslmode=disable
GODEBUG=netdns=go
```
The last environment variable is added because there is a bug and that seting that seems to be an [workaround](https://github.com/woodpecker-ci/woodpecker/issues/1497#issuecomment-1364312746). Start woodpecker service.

```shell
$ sudo systemctl start woodpecker
```

Now visit `http://woodpecker.YOUR_DOMAIN.com`. it should redirect you to `http://git.YOUR_DOMAIN.com` for authentication. Ideally it should be done by now. But the problem is it will crash due to this [issue](https://github.com/woodpecker-ci/woodpecker/issues/1576). The workaround is to change `WOODPECKER_GITEA_URL` to `http://woodpecker.YOUR_DOMAIN.com:3000`. For some reason, woodpecker server crashes when gitea is behind nginx proxy.

> If woodpecker crashes then do the following to get the backtrace
>
> ```shell
> $ sudo -i
> # set -o allexport
> # source /etc/woodpecker.conf
> # set +o allexport
> # /usr/local/bin/woodpecker-server
> ```


Now if the woodpecker server is working then we can proceed further.
We need to change the gitea port because `woodpecker-agent` listens to 3000 which will already be in use by gitea.
So, open `/etc/gitea/app.ini` and change the `HTTP_PORT` to `4000`.
Open the nginx configuration file `/etc/nginx/sites-enabled/gitea.conf` and change `3000` to `4000`.
Now restart both services and check whether everything is working by opening `git.YOUR_DOMAIN.com`.
```
$ sudo systemctl restart gitea
$ sudo systemctl restart nginx
```
Install `docker` if you don't have it installed already.

```
$ sudo apt-get install docker docker-compose
$ sudo docker login
```

Generate a secret using `openssl rand -hex 32` and copy that generated secret into `/etc/woodpecker.conf` as shown below.
```
...
WOODPECKER_AGENT_SECRET=COPIED_SECRET
```

Then save the following file as `/etc/systemd/system/woodpecker-agent.service`
```
[Unit]
Description=Woodpecker
Documentation=https://woodpecker-ci.org/docs/intro
Requires=network-online.target
After=network-online.target

[Service]
User=woodpecker
Group=woodpecker
EnvironmentFile=/etc/woodpecker-agent.conf
ExecStart=/usr/local/bin/woodpecker-agent
RestartSec=5
Restart=on-failure
SyslogIdentifier=woodpecker-agent
WorkingDirectory=/var/lib/woodpecker

[Install]
WantedBy=multi-user.target
```

Following is the configuration file `/etc/woodpecker-agent.conf`

```
WOODPECKER_SERVER=localhost:9000
WOODPECKER_BACKEND=docker
WOODPECKER_AGENT_SECRET=COPIED_SECRET
```

Now start the `woodpecker-agent` service and check it's status

```
$ sudo systemctl start woodpecker-agent
$ sudo systemctl status woodpecker-agent
```

For my case I got some errors

```
Mar 27 15:39:01 git systemd[1]: Started Woodpecker.
Mar 27 15:39:01 git woodpecker-agent[19259]: {"level":"error","error":"rpc error: code = Unknown desc = invalid agent token","time":"2023-03-27T15:39:01Z","message":"grpc error: done(): code: Unk>
Mar 27 15:39:01 git woodpecker-agent[19259]: {"level":"error","error":"rpc error: code = Unknown desc = invalid agent token","time":"2023-03-27T15:39:01Z","message":"pipeline done with error"}
Mar 27 15:39:01 git systemd[1]: woodpecker-agent.service: Deactivated successfully.
```

Which implies that the `woodpecker-server` needs to be restarted. So restart that and restart `woodpecker-agent` too. Now it should be running.

```
$ sudo systemctl restart woodpecker
$ sudo systemctl restart woodpecker-agent
$ sudo systemctl status woodpecker-agent
● woodpecker-agent.service - Woodpecker
     Loaded: loaded (/etc/systemd/system/woodpecker-agent.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-03-27 15:41:01 UTC; 2s ago
       Docs: https://woodpecker-ci.org/docs/intro
   Main PID: 19344 (woodpecker-agen)
      Tasks: 8 (limit: 4660)
     Memory: 3.6M
        CPU: 21ms
     CGroup: /system.slice/woodpecker-agent.service
             └─19344 /usr/local/bin/woodpecker-agent

Mar 27 15:41:01 git systemd[1]: Started Woodpecker.
```

The Woodpecker CI should be up and running
