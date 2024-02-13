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

**CHECK**: IP address points to Nginx defaut page. This requires you to set up Nginx on your remote server first and
adjust the A records as above.

### Docker
This section details the creation of a Docker image and its execution inside a docker container.
Assuming you have created a simple FastAPI server, please execute the below steps:

Use this **Dockerfile** as a template and adapt it to your use case:
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

The above steps should work locally before attempting to execute on the remote server.


### Server Setup
1. Access your remote server using the ssh.
2. Download Docker to the remote server `pip install docker.io`
3. Clone your GitHub repository containing your project.
4. Build and run (detached) the docker container.

### Nginx setup
1. Create a nginx config file (replace $NAME$ with a file name)
```
nano /etc/nginx/sites-available/$NAME$
```
2. Copy the following information and save it.
This allows for the redirection of http to https.

```
server {
   listen 80;
   server_name $DOMAIN$;
   
   location / {
      proxy_pass http://127.0.0.0:8000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote-addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
   }
}
```
3. Create a symbolic link to the config file
```
sudo ln -s /etc/nginx/sites-available/$NAME$ etc/nginx/sites-available/default
```
4. Remove the default nginx config file
```
sudo rm  /etc/nginx/sites-available/default
```
5. Restart nginx
```
sudo service nginx restart
```

**Check:** When visiting your domain in browser, your home page should be visible (not secured)

### Certbot
1. Follow the Certbot installation instructions available at [Certbot]()
2. Execute certbot and follow the instructions in terminal
```
certbot --nginx -d $DOMAIN NAME$
```

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

**Check:** https://example.domain.com.
Once Certbot instructions has been completed, visiting your domain in terminal should show the site is secured.


### Setup Firewall

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
