## THIS CONFIGURATION EMPHASIZE ON MERND DEPLOYMENT USING NGNIX FOR CLIENT,ADMIN AND BACKEND APPLICATIONS

## Initial Setup Configuration

### Check if there is an appache server and delete it

```
systemctl stop apache2
```

```
systemctl disable apache2
```

```
apt remove apache2
```

to delete related dependencies:

```
apt autoremove
```

### Cleaning and updating server

```
apt clean all && sudo apt update && sudo apt dist-upgrade
```

```
rm -rf /var/www/html
```

### Installing nginx

```
apt install nginx
```

### Installing and configure Firewall [ note that this is optional ]

```
apt install ufw
```

```
ufw enable
```

```
ufw allow "Nginx Full"
```

## First Page

#### Delete the default server configuration- it is recommendable to delete the default and setup your own

```
 rm /etc/nginx/sites-available/default
```

```
 rm /etc/nginx/sites-enabled/default
```

#### First configuration

```
 nano /etc/nginx/sites-available/motumag
```

```
server {
  listen 80;

  location / {
        root /var/www/motumag;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
}

```

```
ln -s /etc/nginx/sites-available/motumag /etc/nginx/sites-enabled/motumag

```

##### Write your fist message

```
nano /var/www/motumag/index.html

```

##### Start Nginx and check the page

```
systemctl start nginx
```

## Uploading Apps Using Git

```
apt install git
```

```
mkdir motumag
```

```
cd motumag
```

```
git clone <your repository>
```

## Nginx Configuration for new apps

```
nano /etc/nginx/sites-available/motumag
```

```
location /api {
        proxy_pass http://backendip:port;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
```

##### If you check the location /api you are going to get "502" error which is good. Our configuration works. The only thing we need to is running our app

```
apt install nodejs
```

```
apt install npm
```

```
cd api
```

```
npm install
```

```
nano .env
```

##### Copy and paste your env file

```
node index.js
```

#### But if you close your ssh session here. It's gonna kill this process. To prevent this we are going to need a package which is called `pm2`

```
npm i -g pm2
```

Let's create a new pm2 instance

```
pm2 start --name api index.js
```

```
pm2 startup ubuntu
```

## React App Deployment

```
cd ../client
```

```
nano .env
```

Paste your env file.

```
npm i
```

Let's create the build file

```
npm run build
```

Right now, we should move this build file into the main web file

```
rm -rf /var/www/motumag/*
```

```
mkdir /var/www/motumag/client
```

```
cp -r build/* /var/www/motumag/client
```

Let's make some server configuration

```
 location / {
        root /var/www/motumag/client/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }

```

### Adding Domain

1 - Make sure that you created your A records on your domain provider website.

2 - Change your pathname from Router

3 - Change your env files and add the new API address

4 - Add the following server config

```
server {
 listen 80;
 server_name motumag.com www.motumag.com;

location / {
 root /var/www/motumag/client;
 index  index.html index.htm;
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection 'upgrade';
 proxy_set_header Host $host;
 proxy_cache_bypass $http_upgrade;
 try_files $uri $uri/ /index.html;
}
}

server {
  listen 80;
  server_name api.motumag.com;
  location / {
    proxy_pass http://backendip:port;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }
}

server {
  listen 80;
  server_name admin.motumag.com;
  location / {
    root /var/www/motumag/admin;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }
}
```

## SSL Certification

```
apt install certbot python3-certbot-nginx
```

Make sure that Nginx Full rule is available

```
ufw status
```

```
certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. To set a timer to validate automatically:

```
systemctl status certbot.timer
```
