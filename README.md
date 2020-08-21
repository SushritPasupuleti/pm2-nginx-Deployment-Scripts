# Prepare Production Environment for Deploypent using Nginx and pm2
 
## Nginx Setup
### 1. Install Nginx
```
sudo apt update
sudo apt install nginx
```

### 2. Adjust Firewall
Allow only communication over specifc ports

```
sudo ufw app list
```
Displays available application profiles
```
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

**Nginx Full**: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
**Nginx HTTP**: This profile opens only port 80 (normal, unencrypted web traffic)
**Nginx HTTPS**: This profile opens only port 443 (TLS/SSL encrypted traffic)

Enable a profile
```
sudo ufw allow 'Nginx HTTP'
```

Check Status
```
sudo ufw status
```
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

### 3. Check Nginx Server
```
systemctl status nginx
```

```
nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-04-20 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```

### 4. (Optional) Block all other Ports
Block all incoming Traffic
```
sudo ufw default deny incoming
```
```
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
```

```
sudo ufw allow in "Nginx HTTPS"
```
Enable Firewall with changes
```
sudo ufw enable
```

### 5. Configuring Nginx
Remove default configuration
```
sudo rm /etc/nginx/sites-enabled/default
```

Create a new site
```
sudo nano /etc/nginx/sites-available/[site-name]
```

```
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_set_header   X-Forwarded-For $remote_addr;
        proxy_set_header   Host $http_host;
        proxy_pass         "http://127.0.0.1:[port]";
    }
}
```

Create symlink with sites-enabled
```
sudo ln -s /etc/nginx/sites-available/[site-name] /etc/nginx/sites-enabled/[site-name]
```
Restart Nginx
```
sudo service nginx restart
```

## PM2 Setup

### 1. Install
```
sudo npm install pm2 -g
```

start app

```
pm2 start app.js
```

### PM2 Cheatsheet

### Fork mode

| Command                          | Description              |
| ---                              | ---                      |
| `pm2 start app.js --name my-api` | Start and name a process |

### Cluster mode

| Command                 | Description                                                        |
| ---                     | ---                                                                |
| `pm2 start app.js -i 0` | Will start maximum processes with LB depending on available CPUs	 |

### Listing

| Command          | Description                                         |
| ---              | ---                                                 |
| `pm2 list`       | Display all processes status                        |
| `pm2 jlist`      | Print process list in raw JSON                      |
| `pm2 prettylist` | Print process list in beautified JSON               |
| ---              | ---                                                 |
| `pm2 describe 0` | Display all information about a specific process	 |
| ---              | ---                                                 |
| `pm2 monit`      | Monitor all processes                               |

### Logs

| Command            | Description                               |
| ---                | ---                                       |
| `pm2 logs [--raw]` | Display all processes logs in streaming	 |
| `pm2 flush`        | Empty all log files                       |
| `pm2 reloadLogs`	 | Reload all logs                           |

### Actions

| Command           | Description                                    |
| ---               | ---                                            |
| `pm2 stop all`    | Stop all processes                             |
| `pm2 restart all` | Restart all processes                          |
| ---               | ---                                            |
| `pm2 reload all`  | Will 0s downtime reload (for NETWORKED apps)	 |
| ---               | ---                                            |
| `pm2 stop 0`      | Stop specific process id                       |
| `pm2 restart 0`   | Restart specific process id                    |
| ---               | ---                                            |
| `pm2 delete 0`    | Will remove process from pm2 list              |
| `pm2 delete all`  | Will remove all processes from pm2 list        |

### Misc

| Command                             | Description                                                    |
| ---                                 | ---                                                            |
| `pm2 reset <process>`               | Reset meta data (restarted time...)                            |
| `pm2 updatePM2`                     | Update in memory pm2                                           |
| `pm2 ping`                          | Ensure pm2 daemon has been launched                            |
| `pm2 sendSignal SIGUSR2 my-app`     | Send system signal to script                                   |
| ---                                 | ---                                                            |
| `pm2 start app.js --no-daemon`      | Run pm2 daemon in the foreground if it doesn't exist already	 |
| `pm2 start app.js --no-vizion`      | Skip vizion features (versioning control)                      |
| `pm2 start app.js --no-autorestart` | Do not automatically restart app                               |

## Git Sync
Ignores local changes on server
```
git reset --hard
git pull
```

## .env Updates
Modify .env in client as follows:
```
REACT_APP_API_URL=http://localhost:5000/ => REACT_APP_API_URL=http://148.72.208.218/api/
```

### Docker 

```
sudo docker run -t -d -p 1337:9000 --network="host" sushritlawliet/braggi
```

### ENV

Windows:

```shell
set NODE_ENV=production
```

Linux or other unix based system :

```shell
export NODE_ENV=production
```


## Mongo

### Admin user
```js
db.createUser(
  {
    user: "lawlieto",
    pwd: passwordPrompt(), // or cleartext password
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)
```