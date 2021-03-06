# Operations
This guide describes how to serve this application using Gunicorn and Nginx on Ubuntu 16.04.

Thanks to [Digital Ocean's tutorial here](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04) for some of the basic setup steps.

## Ops Files

Name | Description
------------ | -------------
flower-classifier-web.service | The systemd service file to install on your server. Runs Gunicorn to serve your app. Waits for requests on a socket file that Nginx will pass requests to.
flower-classifier-web-nginx | The Nginx configuration file. Passes web requests to the socket that Gunicorn is waiting on.

## Server Setup Steps

#### Update Package Index
```
sudo apt update
sudo apt upgrade
```

#### Install Packages from Package Index
```
sudo apt install python3-pip python3-dev nginx
```

#### Install Virtualenv
```
sudo pip3 install virtualenv
```

#### Create Project Dir & Virtual Env
```
git clone https://github.com/gregdferrell/flower-classifier-web.git ~/flower-classifier-web
virtualenv ~/flower-classifier-web/env
source ~/flower-classifier-web/env/bin/activate
pip install -r ~/flower-classifier-web/server/requirements.txt
deactivate
```

#### Configure Project
```
# Download checkpoint file or configure your app_config.ini to download it
sudo wget -O ~/flower-classifier-web/server/checkpoint.pth [YOUR-DOWNLOAD-URL]

cp ~/flower-classifier-web/server/app_config_template.ini ~/flower-classifier-web/server/app_config.ini
vi ~/flower-classifier-web/server/app_config.ini
    state.dict.file.path = checkpoint.pth
    state.dict.download.url = [YOUR-DOWNLOAD-URL]
```

#### Setup Permissions
```
# The following assumes your user is named "ubuntu"
sudo chown -R ubuntu:www-data ~/flower-classifier-web
```

#### Setup Gunicorn to Server Your App on Startup
```
sudo cp ~/flower-classifier-web/ops/flower-classifier-web.service /etc/systemd/system/flower-classifier-web.service
sudo systemctl start flower-classifier-web
sudo systemctl enable flower-classifier-web
```

#### Setup Nginx Reverse Proxy
```
sudo cp ~/flower-classifier-web/ops/flower-classifier-web-nginx /etc/nginx/sites-available/flower-classifier-web
sudo vi /etc/nginx/sites-available/flower-classifier-web
    -- Replace [YOUR-SERVER-IP]

sudo ln -s /etc/nginx/sites-available/flower-classifier-web /etc/nginx/sites-enabled
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default_backup
sudo nginx -t
sudo systemctl restart nginx
```
