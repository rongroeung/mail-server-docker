# Install A Mail Server Using Mailcow With Docker
## 1. Install mailcow-dockerized
#### >>> Clone mailcow into the `/opt` folder.

```
cd /opt/
git clone https://github.com/mailcow/mailcow-dockerized
```

## 2. Generate your configuration file and follow the steps in the script.

```
./generate_config.sh
```

## 3. Enter your mailserver FQDN (this is your mailserver hostname, not your domain name)

```
mail.crossroadscambodia.church
```

## 4. Select your timezone

## 5. Modify `mailserver.conf`

#### >>> If you start "mailcow" it will automatically generate and request a letsencrypt certificate for your domains. If you don't want that, but instead use your own certificate you need to modify the `mailserver.conf` and change the line to:

```
SKIP_LETS_ENCRYPT=y
```

#### >>> And change `HTTP_PORT` and `HTTPS_PORT` to your desired port

```
HTTP_PORT=4442
HTTP_BIND=

HTTPS_PORT=4443
HTTPS_BIND=
```

## 6. Import SSL Certificates Into Working Directory of mailcow
#### >>> Import `fullchain.pem` and `privkey.pem` that we've generated using certbot

```
cp /opt/https-httpd/fullchain.pem /opt/mailcow-dockerized/data/assets/ssl/cert.pem
cp /opt/https-httpd/privkey.pem /opt/mailcow-dockerized/data/assets/ssl/privkey.pem
```

## 7. Modify the Nginx SSL Configuration of mailcow

```
cd /opt/mailcow-dockerized/data/conf/nginx/templates/
vi listen_ssl.template
```

#### >>> Add 2 lines below

```
ssl_certificate /etc/ssl/mail/cert.pem;  # Fullchain file path
ssl_certificate_key /etc/ssl/mail/privkey.pem;  # Key file path
```


## 8. Start mailcow services

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

## 9. Login to mailcow

#### >>> When all services are started successfully, you can now login to the admin dashboard and configure your domain, mailboxes, aliases, etc.

#### >>> HTTP Mail Admin Dashboard: `http://mail.crossroadscambodia.church:4442/`

#### >>> HTTPS Mail Admin Dashboard: `https://mail.crossroadscambodia.church:4443/`

#### >>> The default username is `admin`, and the password is `moohoo`

## 10. Set up your domain(s)

#### >>> You need to set up your domain first at `Configuration -> Mail Setup -> Domains`.

## 11. Set up your mailbox(es)

#### >>> If you want to configure your mailboxes, you can add them at `Configuration -> Mail Setup -> Mailboxes`.

