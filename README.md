# Install Mail Server Using Docker
## I. Configure DNS for Mail Server
### 1. Add DNS Record

| Type       | Host             | Value                               |
| ---------- | ---------------- | ----------------------------------- |
| A          | mail             | 38.47.39.132                        |
| CNAME      | autodiscover     | mail.crossroadscambodia.org      |
| CNAME      | autoconfig       | mail.crossroadscambodia.org      |

### 2. Add MX Record

| Type    | Host     | Value                               | Priority   |
| ------- | -------- | ----------------------------------- | ---------- |
| MX      | @        | mail.crossroadscambodia.org      | 10         |

#### >>> Reference: https://docs.mailcow.email/getstarted/prerequisite-dns/

## II. Install mailcow in Docker
### 1. Clone mailcow-dockerized from GitHub
#### >>> Clone mailcow-dockerized into the `/opt` folder.
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
mail.crossroadscambodia.org
```

### 4. Select Timezone 
#### >>> Just press `Enter` to select your default timezone `[Asia/Phnom_Penh]`

### 5. Select Branch
#### >>> Choose number `1` to select master branch 

### 6. Modify `mailcow.conf`
```
cd /opt/mailcow-dockerized/
vi mailcow.conf
```
#### >>> And change `HTTP_PORT` and `HTTPS_PORT` to your desired port
```
HTTP_PORT=3333
HTTP_BIND=

HTTPS_PORT=4444
HTTPS_BIND=
```
#### >>> If you start "mailcow" it will automatically generate and request a letsencrypt certificate for your domains. If you don't want that, but instead use your own certificate you need to modify the `mailcow.conf` and change the line to:
```
SKIP_LETS_ENCRYPT=y
```

### 7. Configure Nginx HTTPS Proxy for Mail Service
#### >>> Copy `fullchain.pem` and `privkey.pem` of your domain `crossroadscambodia.org` that we've generated using certbot
```
cp /opt/https-httpd/fullchain.pem /opt/mailcow-dockerized/data/assets/ssl/crossroadscambodia_church_cert.pem
cp /opt/https-httpd/privkey.pem /opt/mailcow-dockerized/data/assets/ssl/crossroadscambodia_church_privkey.pem
```

#### >>> Modify Nginx SSL Configuration of mailcow
```
cd /opt/mailcow-dockerized/data/conf/nginx/templates/
vi sites-default.conf.j2
```

#### >>> Add 2 lines below
```
ssl_certificate /etc/ssl/mail/crossroadscambodia_church_cert.pem;  # Fullchain file path
ssl_certificate_key /etc/ssl/mail/crossroadscambodia_church_privkey.pem;  # Key file path
```

### 8. Start mailcow services
```
cd /opt/mailcow-dockerized/
docker compose up -d
```

### ***Notice:
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

## III. Setup Mail Domain and Mailbox
### 1. Login to Admin Dashboard
#### >>> HTTP Mail Admin Dashboard: http://mail.crossroadscambodia.org:3333/admin
#### >>> HTTPS Mail Admin Dashboard: https://mail.crossroadscambodia.org:4444/admin
#### >>> The default username is `admin`, and the password is `moohoo`

### 2. Setup Mail Domain
#### >>> You need to setup your domain first at `E-Mail` -> `Configuration` -> `Domains` -> `Add domain`.
#### >>> Here's a sample of domain configuration
##### - `Domain`
```
crossroadscambodia.org
```
##### - `Description`
```
Crossroads Mail Domain
```
##### - `Template`
```
Default
```
##### - `Max. possible aliases`
```
400
```
##### - `Max. possible mailboxes`
```
50
```
##### - `Default mailbox quota (MiB)`
```
3072
```
##### - `Max. quota per mailbox (MiB)`
```
5120
```
##### - `Total domain quota (MiB)`
```
163840
```
##### - `Rate limit (msgs/day)`
```
1000000
```
#### >>> Click `Add domain and restart SOGo` to finish.

### 3. Setup Mailbox
#### >>> To add your mailboxes, you can add them at `E-Mail` -> `Configuration` -> `Mailboxes` -> `Mailboxes` -> `Add mailbox`.
#### >>> Here's a sample of mailbox configuration
##### - `Username`
```
rongroeung
```
##### - `Domain`
```
crossroadscambodia.org
```
##### - `Full name`
```
Theng Rathrongroeung
```
##### - `Template`
```
Default
```
##### - `Quota (MiB)`
```
3072
```
##### - `Rate limit (msgs/day)`
```
10000
```
#### >>> Click `Add` to finish.

### 4. Adding DKIM, DMARC, and SPF Records in DNS Record (Optional)
#### >>> To enhance the security and deliverability of your emails, it’s important to add DKIM, DMARC, and SPF records to your DNS configuration.

| Type    | Host             | Value                                                         |
| ------- | ---------------- | ------------------------------------------------------------- |
| TXT     | @                | v=spf1 mx a -all                                              |
| TXT     | _dmarc           | v=DMARC1;p=none;rua=mailto:dmarc@crossroadscambodia.org;   |
| TXT     | dkim._domainkey  | v=DKIM1;k=rsa;t=s;s=email;p=YOUR_DKIM_KEY                     |

#### >>> Replace `YOUR_DKIM_KEY` with the actual DKIM key generated by Mailcow, which can be found in the Mailcow admin interface under `E-Mail` -> `Configuration` -> `Domains` -> `Edit` -> `Domain: crossroadscambodia.org (dkim._domainkey)`.

### 5. Login to Mailbox
#### >>> HTTP Mailbox: http://mail.crossroadscambodia.org:3333
#### >>> HTTPS Mailbox: https://mail.crossroadscambodia.org:4444

### 6. Add SMTP Relay
#### >>> Note: Do this step only in case our mail server got unauthorized when send email to other mail domains especially Gmail Domain.
#### >>> To prevent spam, Google has restricted sending email directly from Our Mail Server to Gmail Server. So we need a SMTP Relay Service Provider which is a third-party service that specializes in delivering emails. They have strong sender reputations and are whitelisted by major email providers. By using a relay, our server will send the email to the relay service first, and the relay service will then deliver it to Gmail. We can find a free SMTP Relay Service in Google Search Engine, then register and take the access info of the SMTP Relay to configure in our Mail Admin Dashboard.
#### >>> To add SMTP Relay in our Mail System, login to Mail Admin Dashboard https://mail.crossroadscambodia.org:4444/admin
#### >>> Then click `System` -> `Configuration` -> `Routing`.
#### >>> Scoll down to `Transport Maps` section, then follow `Add transport` configuration below.
##### - `Destination`
```
gmail.com
```
##### - `Next hop`
```
smtp-relay.brevo.com:587
```
##### - `Username`
```
username-provided-by-smtp-relay-provider
```
##### - `Password`
```
password-provided-by-smtp-relay-provider
```
##### - `Destination is a regular expression to match against MX name (.*\.google\.com to route all mail targeted to a MX ending in google.com over this hop)`
```
Untick
```
##### - `Active`
```
Tick
```
#### >>> Click `Add` to finish.

## IV. Reference
#### >>> https://docs.mailcow.email/
#### >>> https://technicalsahil.com/how-to-install-mailcow-on-ubuntu-22-04/
