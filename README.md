# guacamole, 

# Guacamole / NPM / SAML SSO / FreeIPA

---

## register your saml application in Azure portal

Create .env file and add filling following values

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
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > ./runtime/guacdb/init/initdb.sql
```

- Launch
```bash
docker compose up -d
```

## Configure NPM
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

## To add Azure AD user as an admin
- login from local via http://your_domain_name:8080/guacamole/
- Use guacadmin/guacadmin to login.
- Add an local admin user. The user’s username must match the AzureAD user’s email, ie, jtong@creekside.network. Assign admin permissions etc. Do not fill in any password.

## Tips
- Remove local database login
    - Change docker compose file, and relauch containers
```markdown
   EXTENSION_PRIORITY: "* saml" => EXTENSION_PRIORITY: "saml"
```
