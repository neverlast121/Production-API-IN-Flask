# Production-API-IN-Flask

## Introduction
This guide provides step-by-step instructions to deploy a Flask application in a production environment using Nginx as a reverse proxy and Gunicorn as the WSGI server.

## Installation and Setup

### Install Required Dependencies

```bash
<server command> install python3-venv
<server command> install nginx
```

### Create Project Directory and Python Virtual Environment

```bash
mkdir ~/project
cd ~/project
python3 -m venv <env-name>
```
`<env-name>` is the name of the virtual environment.

### Activate Virtual Environment

```bash
# Linux
source <env-name>/bin/activate

# Windows
<env-name>/Scripts/activate.bat
```

### Install Dependencies

```bash
pip install wheel gunicorn flask
```

### Create Python File main.py

```bash
nano ~/project/main.py
```

### Add the Following Code

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def welcome():
    return 'Welcome to your first API'

if __name__=='__main__':
    app.run(host='0.0.0.0')
```

### Save and Close the File

### Verify by Running the Application

```bash    
cd ~/project
python3 main.py
```

### Expected Output

```bash
* Serving Flask app 'main'
* Debug mode: off
* Running on all addresses.
  WARNING: This is a development server. Do not use it in a production deployment.
* Running on http://192.168.0.2:5000/ 
(Press CTRL+C to quit)
```

### Press CTRL+C to Close the Application

### Create the WSGI Entry Point

```bash
nano ~/project/wsgi.py
```

### Add the Following Lines

```python
from main import app

if __name__=='__main__':
   app.run()
```

### Run the Following Command in Your Terminal

```bash 
cd ~/project
gunicorn --bind 0.0.0.0:5000 wsgi:app
```

### Expected Output

```bash
[2024-03-28 10:37:15 +0000] [23362] [INFO] Starting gunicorn <version>
[2024-03-28 10:37:15 +0000] [23362] [INFO] Listening at: http://0.0.0.0:5000 (23362)
[2024-03-28 10:37:15 +0000] [23362] [INFO] Using worker: sync
[2024-03-28 10:37:15 +0000] [23364] [INFO] Booting worker with pid: 23364
```

### Press CTRL+C to Close the Application

### Deactivate the Virtual Environment

```bash
deactivate
```

### Create systemd Service File for Flask Application

```bash
nano /etc/systemd/system/<name>.service
```

### Add the Following Lines

```plaintext
[Unit]
Description= <describe it>
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/project
Environment="PATH=/root/project/<env-name>/bin"
ExecStart=/root/project/<env-name>/bin/gunicorn --bind 0.0.0.0:5000 wsgi:app

[Install]
WantedBy=multi-user.target
```

### Save and Close the File

### Set Ownership and Permission to Flask Project

```bash
chown -R root:www-data /root/project
chmod -R 775 /root/project
```

### Reload the systemd Daemon

```bash
systemctl daemon-reload
```

### Start and Enable the Service

```bash
systemctl start <name>
systemctl enable <name>
```

### Verify the Status

```bash
systemctl status <name>
```

### Expected Output

```bash
● flask.service - Gunicorn instance to serve Flask
     Loaded: loaded (/etc/systemd/system/flask.service; enabled; preset: enabled)
     Active: active (running) since Wed 2024-03-27 16:16:57 UTC; 13h ago
   Main PID: 164345 (gunicorn)
      Tasks: 7 (limit: 2309)
     Memory: 405.3M
        CPU: 19.547s
     CGroup: /system.slice/flask.service
             ├─164345 /root/project/<env-name>/bin/python3 /root/project/<env-name>/bin/gunicorn --bi>
             └─164348 /root/project/<env-name>/bin/python3 /root/project/<env-name>/bin/gunicorn --bi>

Mar 27 22:45:09 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:09 +0000] [164348] [>
Mar 27 22:45:12 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:12 +0000] [164348] [>
Mar 27 22:45:17 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:17 +0000] [164348] [>
Mar 27 22:45:29 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:29 +0000] [164348] [>
Mar 27 22:45:30 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:30 +0000] [164348] [>
Mar 27 22:45:41 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:41 +0000] [164348] [>
Mar 27 22:45:43 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:43 +0000] [164348] [>
Mar 27 22:45:50 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:45:50 +0000] [164348] [>
Mar 27 22:46:13 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:46:13 +0000] [164348] [>
Mar 27 22:46:29 ubuntu-s-1vcpu-2gb-sfo3-01 gunicorn[164348]: [2024-03-27 22:46:29 +0000] [164348] [>
```

### Configure Nginx as a Reverse Proxy

### Create an Nginx Virtual Host Configuration File

```bash
nano /etc/nginx/conf.d/nginx.conf
```

### Add the Following Lines

```plaintext
server {
        listen 80;
        server_name _;
        location / {
                include proxy_params;
                proxy_pass http://127.0.0.1:5000/;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-Host $host;
                proxy_set_header X-Forwarded-Prefix /;
        }
}
```

### Save and Close the File

### Check for Syntax Errors

```bash
nginx -t
```

### Expected Output

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Restart Nginx

```bash
systemctl restart nginx
```

### Check Nginx Status

```bash
systemctl status nginx
```

### Expected Output

```bash
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Wed 2024-03-27 12:31:23 UTC; 17h ago
       Docs: man:nginx(8)
   Main PID: 161726 (nginx)
      Tasks: 2 (limit: 2309)
     Memory: 2.0M
        CPU: 149ms
     CGroup: /system.slice/nginx.service
             ├─161726 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─161727 "nginx: worker process"

Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 systemd[1]: Starting nginx.service - A high performance >
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 nginx[161724]: 2024/03/27 12:31:23 [warn] 161724#161724:>
Mar 27 12:31:23 ubuntu-s-1

vcpu-2gb-sfo3-01 nginx[161725]: 2024/03/27 12:31:23 [warn] 161725#161725:>
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 systemd[1]: Started nginx.service - A high performance w>
```

Your Flask application is now successfully deployed in a production environment with Nginx and Gunicorn. 
Your API will be http://<your-ip-address>:5000 for following all these steps as it is.
For further services or configurations, please visit the official websites of Flask, Nginx, and Gunicorn.
