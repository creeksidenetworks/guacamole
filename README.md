# Guacamole / NPM / SAML SSO / FreeIPA

---

## Register your saml application in Azure portal

- Registration values

| Identifier | https://URL |
|------------|------------|
| Reply URL | https://URL/api/ext/saml/callback |


- Create .env file and add filling following values

```markdown
## SAML Configuration
SAML_IDP_METADATA_URL=<meta URL>
## Alternative: Single Sign On URL from Azure registration, end with "saml2"
SAML_IDP_URL=<login URL>
## These must match what was entered into Azure AD
SAML_ENTITY_ID=<guacamole URL>
SAML_CALLBACK_URL=<guacamole URL>
```

## Launch containers
- Prepare database initialization

```bash
mkdir -p ./runtime/guacdb/init
```
```bash
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > ./runtime/guacdb/init/initdb.sql
```

- Launch
```bash
docker compose up -d
```

## Proxy with NPM
- Use admin@example.com/changeme to login NPM @ http://127.0.0.1:81/

- Add proxy host
    - Domain Names: [your domain name]
    - Scheme: http
    - Forward Hostname: guacamole
    - Forward Port: 8080
    - Turn on websocket support

- Enable SSL
    - Turn Force SSL on

- In advance configue, paste following, then you should be able to access via https://your_domain_name

```markdown
location / {
    proxy_pass http://guacamole:8080/guacamole/;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_cookie_path /guacamole/ /;
    access_log off;
    # allow large uploads (default=1m)
    # 4096m = 4GByte
    client_max_body_size 4096m;
}
```

# Proxy non-standard port with NGINX 

- Obtain Let's encryption certificates

```bash
CLOUD_API_TOKEN=xxx
MY_EMAIL=xxx
DOMAIN_NAME=xxx

sudo yum install certbot  python3-certbot-dns-cloudflare -y
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

sudo mkdir -p /etc/cloudflare
sudo echo "dns_cloudflare_api_token = $CLOUD_API_TOKEN"  | sudo tee tee /etc/cloudflare/certbot.cloudflare.api.ini
sudo chmod 600  /etc/cloudflare/certbot.cloudflare.api.ini

sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials \
    /etc/cloudflare/certbot.cloudflare.api.ini \
    -m $MY_EMAIL -d $DOMAIN_NAME
```
- Renewal server certificate automatically
```markdown
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin

0 0 * * * /usr/bin/certbot renew --nginx --renew-hook "/bin/systemctl reload nginx.service"
```
- Create your site configuration file under /etc/nginx/conf.d/yourserver.conf
```markdown
server {
    listen     27443 ssl;
    server_name your_server_fqdn;

    ssl_certificate /etc/letsencrypt/live/your_server_fqdn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_server_fqdn/privkey.pem;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    location / {
        proxy_pass http://127.0.0.1:8080/guacamole/;

        # remove the /guacamole/ prefix from the URL
        proxy_cookie_path /guacamole/ /;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Disable buffering for WebSocket
        proxy_buffering off;

        # Rewrite backend redirects to include the custom port
        proxy_redirect https://your_server_fqdn/ https://your_server_fqdn:27443/;

        # Allow large uploads (default=1m)
        # 4096m = 4GByte
        client_max_body_size 4096m;

        # Disable access logging for WebSocket (optional)
        access_log off;
    }
}
```
- Start nginx service
```bash
sudo systemctl start nginx 
sudo systemctl enable nginx 
```
## To add Azure AD user as an admin
- login from local via http://your_domain_name:8080/guacamole/
- Use guacadmin/guacadmin to login.
- Add an local admin user. The user’s username must match the AzureAD user’s email, ie, jtong@creekside.network. Assign admin permissions etc. Do not fill in any password.

## Tips
- Remove local database login by modify docker-compose.yml

| EXTENSION_PRIORITY (w/ login) | "*, saml" |
|------------|------------|
| EXTENSION_PRIORITY (w/o login) | "saml" |
