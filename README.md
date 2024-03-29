# Production-API-IN-Flask

## Introduction
This guide provides step-by-step instructions to deploy a Flask application in a production environment using Nginx as a reverse proxy and Gunicorn as the WSGI server.

## Installation and Setup

# Install Required Dependencies

```bash
<server command> install python3-venv
<server command> install nginx
```
# Create Project Directory and Python Virtual Environment

```bash
mkdir ~/project
cd ~/project
python3 -m venv <env-name>
```
# Activate Virtual Environment

```bash
# Linux
source <env-name>/bin/activate

# Windows
<env-name>/Scripts/activate.bat
```
# Install dependencies

```python
pip install wheel gunicorn flask
```
# Next create python file main.py

```bash
nano ~/project/main.py
```
# Add the following code

```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def welcome():
    return 'Welcome to your first API'

if __name__=='__main__':
    app.run(host='0.0.0.0')
```
# Save and close the file
# To verify run with this command
# Run it with run option in your ide or use command line

```bash    
cd ~/project
python3 main.py
```
# If everything goes well you will see something like the following

```bash
* Serving Flask app 'main'
* Debug mode: off
* Running on all addresses.
  WARNING: This is a development server. Do not use it in a production deployment.
* Running on http://192.168.0.2:5000/ 
(Press CTRL+C to quit)
```
# Press CTRL+C to close the application
# Next we will create the WSGI entry Point to serve the application via Gunicorn

#Create wsgi file

```bash
nano ~/project/wsgi.py
```
# Add the following lines

```python
from main import app

if __name__=='__main__':
   app.run()
```
# For your required or other services Check wsgi official website
# Run the following command in your terminal

```bash 
cd ~/project/gunicorn --bind 0.0.0.0:5000 wsgi:app
```
# If all goes well you will see some thing like

```bash
[2024-03-28 10:37:15 +0000] [23362] [INFO] Starting gunicorn <version>
[2024-03-28 10:37:15 +0000] [23362] [INFO] Listening at: http://0.0.0.0:5000 (23362)
[2024-03-28 10:37:15 +0000] [23362] [INFO] Using worker: sync
[2024-03-28 10:37:15 +0000] [23364] [INFO] Booting worker with pid: 23364
```
# Press CTRL+C to close the application
# Deactivate the virtual Environment

```bash
deactivate
```
# Next we will create systemd services file for flask application

```bash
nano /etc/systemd/system/<name>.service
```
#Add the following lines

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
# Save and close the file
# Set the ownershiP and permission to flask project

```bash
chown -R root:ww-data /root/project
chmod -R 775 /root/project
```
# Next reload the systemd daemon with

```bash
systemctl daemon-reload
```
# Next start the <name>.service and enable it with start at system reboot

```bash
systemctl start flask
systemctl enable flask
```
# Next verify the status

```bash
systemctl status flask
```
# The output will be something like

```bash
● flask.service - Gunicorn instance to serve Flask
     Loaded: loaded (/etc/systemd/system/flask.service; enabled; preset: enabled)
     Active: active (running) since Wed 2024-03-27 16:16:57 UTC; 13h ago
   Main PID: 164345 (gunicorn)
      Tasks: 7 (limit: 2309)
     Memory: 405.3M
        CPU: 19.547s
     CGroup: /system.slice/flask.service
             ├─164345 /root/baby_cry/baby_env/bin/python3 /root/baby_cry/baby_env/bin/gunicorn --bi>
             └─164348 /root/baby_cry/baby_env/bin/python3 /root/baby_cry/baby_env/bin/gunicorn --bi>

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
# Next configure Nginx as a Reverse Proxy for flask Application
# Create an Nginx virtual host configuration file with port 80.

```bash
nano /etc/nginx/conf.d/nginx.conf
```
# Add the following lines

```plaintext
server {
        listen 80;
        server_name _;
        location / {
                include proxy_params;
                proxy_pass http://172.0.0.1:5000/;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded_proto $scheme;
                proxy_set_header X-Forwarded-Host $host;
                proxy_set_header X_Forwarded-Prefix /;

        }
}
```
# Save and close it
# Check for any syntax error

```bash
nginx -t
```
# You will have to see this

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
# Finally restart the nginx

```bash
systemctl restart nginx
```
# Check status to see like this

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
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 nginx[161724]: 2024/03/27 12:31:23 [warn] 161724#161724:>
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 nginx[161725]: 2024/03/27 12:31:23 [warn] 161725#161725:>
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 nginx[161725]: 2024/03/27 12:31:23 [warn] 161725#161725:>
Mar 27 12:31:23 ubuntu-s-1vcpu-2gb-sfo3-01 systemd[1]: Started nginx.service - A high performance w>
```
# Your Flask application is now successfully deployed in a production environment with Nginx and Gunicorn.
# For further services or configurations, please visit the official websites of Flask, Nginx, and Gunicorn.

