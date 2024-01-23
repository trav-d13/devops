# Remote Server
The following setup will allow for the creation of a remote server capable of hosting
a dynamic website. This is possible through **Nginx**, **Docker**, and **FastAPI**.

## Setup
The following setup process, encompasses the following dependencies/ tools:
- Nginx
- Docker
- Github
- FastAPI
- Certbot
- DNS Records

### Pre-setup
1. Purchase domain from [domain.com](https://www.domain.com/) or [godaddy.com](https://www.godaddy.com)
2. Rent a remote server. I recommend [Linode](https://cloud.linode.com/linodes). 
3. Modify your domain's DNS records:
    - Remove any any existing A record that may say **Parked**
    - Add a new A record, with the following information:
      - name: @
      - data/value: remote server IP address
      - TTL: 1/2 hour

### Docker
This section details the creation of a Docker images and its execution inside a docker container.
This section assumes that you have created a simple FastAPI server.

Use the following as a sample **Dockerfile**
```Dockerfile
# Import latest Python version
FROM python:latest

# Set the working directory to app/
WORKDIR /app

# Copy the current directory contents into /app
COPY . /app

# Install requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Specifies port on which container will listen at runtime
EXPOSE 8000

# Run FastAPI server, specifying port on which univorn will listen
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

1. Build the image
```bash
sudo docker build -t <IMAGE NAME> .
```

2. Run the container (detached)
```bash
# This maps the port 8000 on the host machine to port 8000 inside of the container
docker run -d -p 8000:8000 <IMAGE NAME>
```

The above steps should work locally before attempting to execute on the remote server

### Status 01
You can access your hosted website through **http** protocol. 
The majority of web-browsers will recognize this as an unsafe sight, as it does
not follow the https protocol. 

### Server Setup
#TODO Continue from here specifying nginx setup, and what should be expected



### 1. Install Server Software
- nginx
- git
- docker-compose
- application code

**CHECK**: IP address points to Nginx defaut page

### 2. DNS
Register Domain name and set A records to point to static IP of server hosting API

**CHECK**: domain name points to Nginx default page

### 3. OpenSSL Certbot 
SSL into server and run CertBot as per [instructions](https://certbot.eff.org/instructions)

If correctly installed should get this:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/drone-flying-online.space/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/drone-flying-online.space/privkey.pem
This certificate expires on 2023-04-15.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

### 4. Configuration file `/etc/nginx/sites-enabled/demo.conf`:

```
## Redirect http to https
server {
   listen 80;
   server_name example.yourdomain.com;
   return 307 https://example.yourdomain.com$request_uri;
 }
 
## Redirect www to https
server {
   listen 80;
   server_name www.example.yourdomain.com;
   return 307 https://example.yourdomain.com$request_uri;
}

 
server {
   listen 443 ssl;
   server_name example.yourdomain.com;
   ssl_certificate  /path/to/your/certificate;
   ssl_certificate_key  /path/to/your/certificate/key;
   ssl_prefer_server_ciphers on;

   ## Forward through original IP
   set_real_ip_from 0.0.0.0/0;
   real_ip_header X-Real-IP;
   real_ip_recursive on;

   location / {
        proxy_pass http://localhost:8080;

        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Host $remote_addr;
        proxy_set_header        X-Forwarded-Proto $scheme;
   }
}

```
Reload the Nginx server: `nginx -s reload`

**CHECK**: https://example.domain.com

### 5. Setup Firewall

Confirm ufw installed and disabled
```
ufw status
```

Reset all firewall rules
```
ufw reset
```

set defaults for incoming and outgoing
```
ufw default deny incoming
ufw default allow outgoing
```

enable SSH, https, http and nginx
```
ufw allow ssh
ufw allow 80
ufw allow 442
ufw allow 'Nginx Full'
```

Enable firewall and check 
```
ufw enable -y  && ufw status
```
