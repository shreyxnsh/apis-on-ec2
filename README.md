# Host NodeJs REST APIs on AWS EC2
Host NodeJS API's on AWS EC2 instance using these simple steps.



## Commands

After connecting to your EC2 instance via a SSH client : 


Run these four commands to install all the required packages in your machine : NodeJS , NPM , Nginx

```bash
  sudo apt update
```

```bash
  sudo apt install nodejs
```

```bash
  sudo apt install npm
```

```bash
  sudo apt install nginx
```

```bash
  sudo apt update
```


Clone your project's github repository into the EC2 machine in a folder (ex : api-folder)
```bash
  git clone "https://github.com/your-repo-link"
```


Move to the api-folder directory
```bash
  cd api-folder
```
Navigate to the .env file to paste your environment variables
```bash
  nano .env
```

Install NVM (Node Version Manager)
```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

```bash
  export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
```

```bash
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

```bash
  nvm install v18
```
Install NPM packages of your project
```bash
  npm i
```

Run any of these as per your project's package.json
```bash
  npm run dev    /   npm start
```
Create the server code file for your domain (ex : testapi.com)
```bash
  sudo nano /etc/nginx/sites-available/testapi.com
```
Once you run the command above, you will enter an environment where you have to paste this code, MAKE SURE to change server_name and root folder directory
```bash
  server {
        listen 80;
        listen 443;
        server_name testapi.com;
        root /home/ubuntu/api-folder;
        index index.html index.htm;

        location / {
                proxy_pass             http://127.0.0.1:3000;
                proxy_read_timeout     60;
                proxy_connect_timeout  60;
                proxy_redirect         off;

                # Allow the use of websockets
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
         location /as {
        access_log off;
        add_header 'Content-Type' 'text/plain';
    }

}
```

After pasting this code save and exit it using : 
```bash
  Ctrl + S  --> Ctrl + C  --> Ctrl + X
```
Paste these commands as it is (change your domain name)
```bash
  sudo ln -s /etc/nginx/sites-available/testapi.com /etc/nginx/sites-enabled/testapi.com
```

```bash
  sudo rm -rf /etc/nginx/sites-enabled/default
```

Restart your nginx service after changing server code : 
```bash
  sudo service nginx restart
```


```bash
  systemctl status nginx.service
```
```bash
  sudo service nginx restart
```
Check if your nginx is completely healthy :
```bash
  sudo service nginx status
```
Install pm2 (Process Manager) globally in this machine
```bash
  sudo npm i -g pm2
```

Start your server : (must have in package.json -->  "start": "nodemon index)
```bash
  pm2 start npm -- start
```
Get the logs of your running server to know what's happening
```bash
  pm2 logs 0
```

Stop the server
```bash
  pm2 stop 0
```


# Get SSL Certificate for https protocol
Run these commands and change the nginx server configuration to accept 443 ssl requests.


Update and install certbot
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

```
Run the certbot
```bash
sudo certbot --nginx -d testapi.com
```
Nano server configuration
```bash
sudo nano /etc/nginx/sites-enabled/testapi.com
```

Update Server configuration 
```bash
  
server {
    listen 443 ssl;
    server_name testapi.com;

    ssl_certificate /etc/letsencrypt/live/testapi.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/testapi.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_read_timeout 60;
        proxy_connect_timeout 60;
        proxy_redirect off;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name testapi.com;
    return 301 https://$host$request_uri;
}
```
Check Server
```bash
sudo nginx -t
```
Restart nginx
```bash
sudo systemctl restart nginx
```
Auto renew certificate after 90 days
```bash
sudo certbot renew --dry-run
```
The above commands will allow https request and route http to https in browser but if you want to accept http requests then update the server configuration to this 
```bash
server {
    listen 80;
    server_name testapi.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl;
    server_name testapi.com;

    ssl_certificate /etc/letsencrypt/live/testapi.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/testapi.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```




