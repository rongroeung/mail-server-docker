# Install A Mail Server Using Mailcow With Docker
## I. Add DNS Record for Mail Server
### 1. Add DNS Record

| Type       | Host             | Value                               |
| ---------- | ---------------- | ----------------------------------- |
| A          | mail             | 38.47.39.132                        |
| CNAME      | autodiscover     | mail.crossroadscambodia.church      |
| CNAME      | autoconfig       | mail.crossroadscambodia.church      |

### 2. Add MX Record

| Type    | Host     | Value                               | Priority   |
| ------- | -------- | ----------------------------------- | ---------- |
| MX      | @        | mail.crossroadscambodia.church      | 10         |

## II. Install mailcow-dockerized in Docker
### 1. Clone mailcow from GitHub
#### >>> Clone mailcow into the `/opt` folder.
```
cd /opt/
git clone https://github.com/mailcow/mailcow-dockerized
```

### 2. Generate your configuration file and follow the steps in the script.
```
cd /opt/mailcow-dockerized/
./generate_config.sh
```

### 3. Enter your mailserver FQDN (this is your mailserver hostname, not your domain name)
```
mail.crossroadscambodia.church
```

### 4. Select Timezone 
#### >>> Just press `Enter` to select your default timezone `[Asia/Phnom_Penh]`

### 5. Select Branch
#### >>> Choose number `1` to select master branch 

### 6. Modify `mailserver.conf`
#### >>> And change `HTTP_PORT` and `HTTPS_PORT` to your desired port
```
HTTP_PORT=4442
HTTP_BIND=

HTTPS_PORT=4443
HTTPS_BIND=
```
#### >>> If you start "mailcow" it will automatically generate and request a letsencrypt certificate for your domains. If you don't want that, but instead use your own certificate you need to modify the `mailserver.conf` and change the line to:
```
SKIP_LETS_ENCRYPT=y
```

### 7. Start mailcow services
```
cd /opt/mailcow-dockerized/
docker compose up -d
```

### **Notice:
#### >>> Command to stop mailcow services
```
cd /opt/mailcow-dockerized/
docker compose down
```

#### >>> Command to check mailcow services status
```
cd /opt/mailcow-dockerized/
docker compose ps
```

## III. Configure Nginx HTTPS Proxy for Mail Service
### 1. Add Configuration File for Mail Service
#### >>> Create a cr-mail-service.conf on path /etc/nginx/conf.d with the following content:
```
server {
    listen 4444 ssl;
    server_name crossroadscambodia.church;

    ssl_certificate /opt/https-httpd/fullchain.pem;
    ssl_certificate_key /opt/https-httpd/privkey.pem;

    location / {
        proxy_pass http://192.168.10.111:4442;  # URL of mail service
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
#### >>> Replace `4444` with your desired https port.
#### >>> Replace `crossroadscambodia.church` with your actual domain name.
#### >>> Replace `http://192.168.10.111:4442` with your mail service url.
#### >>> Update `/opt/https-httpd/fullchain.pem` and `/opt/https-httpd/privkey.pem` with the paths to your SSL certificate and key files.
#### >>> If you don't have SSL certificates, you may generate using this guideline: `https://github.com/rongroeung/apache-httpd-https?tab=readme-ov-file#i-generate-ssltls-certificate`

### 2. Test Nginx Configuration
#### >>> Before restarting Nginx, it's a good practice to test the configuration for syntax errors:
```
nginx -t
```
#### >>> If the test is successful, you should see: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok.

### 3. Restart Nginx
#### >>> After verifying the configuration, restart Nginx for the changes to take effect:
```
systemctl restart nginx
```

## IV. Setup Mail Domain and Mailbox
### 1. Login to Admin Dashboard
#### >>> When all services are started successfully, you can now login to the admin dashboard and configure your domain, mailboxes, aliases, etc.
#### >>> HTTP Mail Admin Dashboard: `http://crossroadscambodia.church:4442/`
#### >>> HTTPS Mail Admin Dashboard: `https://crossroadscambodia.church:4444/`
#### >>> The default username is `admin`, and the password is `moohoo`

### 2. Setup Mail Domain
#### >>> You need to setup your domain first at `E-Mail -> Configuration -> Domains -> Add domain`.

### 3. Setup Mailbox
#### >>> If you want to configure your mailboxes, you can add them at `E-Mail -> Configuration -> Mailboxes -> Mailboxes -> Add mailbox`.

### 4. Adding DKIM, DMARC, and SPF Records in DNS Record (Optional)
#### >>> To enhance the security and deliverability of your emails, it’s important to add DKIM, DMARC, and SPF records to your DNS configuration.

| Type    | Host             | Value                                                                                                  |
| ------- | ---------------- | ------------------------------------------------------------------------------------------------------ |
| TXT     | @                | v=spf1 mx a -all                                                                                       |
| TXT     | _dmarc           | v=DMARC1;p=none;rua=mailto:dmarc@crossroadscambodia.church;ruf=mailto:dmarc@crossroadscambodia.church; |
| TXT     | dkim._domainkey  | v=DKIM1;k=rsa;t=s;s=email;p=YOUR_DKIM_KEY                                                              |

#### >>> Replace `YOUR_DKIM_KEY` with the actual DKIM key generated by Mailcow, which can be found in the Mailcow admin interface under `Configuration -> ARC/DKIM keys`.

### 5. Login to Mailbox
#### >>> HTTP Mailbox: `http://crossroadscambodia.church:4442/SOGo`
#### >>> HTTPS Mailbox: `https://crossroadscambodia.church:4444/SOGo`
