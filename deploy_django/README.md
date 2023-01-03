# Django Deployment Instruction

This project aims to show deployment process of Django in easy way and organized way. This project won't cover [Django Deployment Checklist](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/). It will only cover deployment process of Django.

For this deployment, we will use [Django](https://www.djangoproject.com/) and [Gunicorn](https://gunicorn.org/) as WSGI server. We will use [Nginx](https://www.nginx.com/) as reverse proxy and [Supervisor](http://supervisord.org/) as process manager.

You can use `Linux Service` instead of Supervisor. But, I prefer Supervisor because it is easy to use and manage.

## Prerequisites

- [Django](https://www.djangoproject.com/) project
- [Gunicorn](https://gunicorn.org/) installed
- [Nginx](https://www.nginx.com/) installed
- [Supervisor](http://supervisord.org/) installed

Moving forward, we will assume that you have already installed all the prerequisites and have a Django project.

## Pre-checklist

1. Make sure you have Python installed. You can check it by running `python --version` or `python3 --version` in your terminal. If you don't have Python installed, you can install it from [here](https://www.python.org/downloads/). For ubuntu, you can install it by running `sudo apt-get install python3-pip python3-dev libpq-dev -y` in your terminal. It will install with all the required packages.

2. Make sure you update your `pip` and machine apt packages. You can do it by running `pip install --upgrade pip` and `sudo apt update && sudo apt upgrade -y` in your terminal.

3. Install [Gunicorn](https://gunicorn.org/) by running `pip install gunicorn` in your terminal.

4. Install [Nginx](https://www.nginx.com/) by running `sudo apt install nginx -y` in your terminal.

5. Install [Supervisor](http://supervisord.org/) by running `sudo apt install supervisor -y` in your terminal.

That's it for pre-checklist. Now, let's move to the deployment process.

## Deployment Process

### 1. Create a Supervisor configuration file

Create a file named `supervisor_django.conf` in `/etc/supervisor/conf.d/` directory.

You can do it by running

```bash
sudo nano /etc/supervisor/conf.d/supervisor_django.conf
```

in your terminal. Paste the following code in the file.

```ini
[program:deploy_django]
directory=/home/ubuntu/deploy_django
environment=HOME="/home/ubuntu",USER="ubuntu" # Replace with your user name
user=ubuntu # Replace with your user name

# Replace with your virtual environment path
command=/home/ubuntu/deploy_django/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/deploy_django/deploy_django.sock deploy_django.wsgi:application
autostart=true
autorestart=true

stdout_logfile=/var/log/deploy_django/deploy_django.log
stderr_logfile=/var/log/deploy_django/deploy_django.log
logfile_maxbytes=1000000 # 1MB
```

Replace `deploy_django` with your Django project name. Replace `/home/ubuntu/deploy_django` with your Django project directory path. Replace `ubuntu` with your user name.

### 2. Create a log directory

Create a directory named `deploy_django` in `/var/log/` directory.

You can do it by running

```bash
sudo mkdir /var/log/deploy_django
```

in your terminal.

### 3. Start Supervisor

Start Supervisor by running

```bash

sudo supervisorctl reread 

sudo supervisorctl update
```

in your terminal.

### 4. Create a Nginx configuration file

Create a file named `nginx_django.conf` in `/etc/nginx/sites-available/` directory.

You can do it by running

```bash
sudo nano /etc/nginx/sites-available/nginx_django.conf
```

in your terminal. Paste the following code in the file.

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/deploy_django;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/deploy_django/deploy_django.sock;
    }
}
```

Replace `example.com` with your domain name. Replace `deploy_django` with your Django project name. Replace `/home/ubuntu/deploy_django` with your Django project directory path.

Run

```bash
sudo ln -s /etc/nginx/sites-available/nginx_django.conf /etc/nginx/sites-enabled
```

in your terminal. It will create a symbolic link of `nginx_django.conf` file in `/etc/nginx/sites-enabled/` directory.

After that, run

```bash
sudo nginx -t
```

in your terminal. It will check your Nginx configuration file for any error.

If there is no error, run

```bash
sudo systemctl restart nginx
```

in your terminal. It will restart Nginx.

By now, you have successfully deployed your Django project. You can check it by visiting your domain name in your browser.

Troubleshooting

If you are getting `502 Bad Gateway` error, you can check your Nginx error log by running

```bash
sudo tail -f /var/log/nginx/error.log
```

doing this, you will get the error log in your terminal. You can check it and fix the error.

if the error is `Permission denied`, you can fix it by running

```bash
sudo chgrp www-data /home/ubuntu/
chmod g+x /home/ubuntu/
chmod g+r /home/ubuntu/
sudo systemctl restart nginx
```

what this command does is, it gives permission to `www-data` group to access `/home/ubuntu/` directory. After that, it restarts Nginx.

## Conclusion

In this tutorial, we have learned how to deploy a Django project on Ubuntu >=20.04. We have used Gunicorn, Nginx, and Supervisor to deploy our Django project.

If you have any questions, feel free to ask me in my [LinkedIn](https://www.linkedin.com/in/chapimenge/) or [Twitter](https://twitter.com/chapimenge3) account.

Enjoy coding!