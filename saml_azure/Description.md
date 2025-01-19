# Guacamole / NPM / SAML SSO

---

## Prepare database initialization

```bash
mkdir -p ./runtime/guacdb/init
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > ./runtime/guacdb/init/initdb.sql
```

## launch containers
```bash
docker compose up -d
```

## Configure NPM
Use admin@example.com/changeme to login NPM @ http://127.0.0.1:81/

### Add proxy host
Domain Names: [your domain name] 

Scheme: http 

Forward Hostname: guacamole 

Forward Port: 8080 

### Enable docker compose up -d

Turn Force SSL on

## login from local via http://your_domain_name:8080/guacamole/

Use guacadmin/guacadmin to login.

Add an local admin user. The user’s username must match the AzureAD user’s email, ie, jtong@creekside.network. Assign admin permissions etc. Do not fill in any password.

## Configure NPM 2

Copy contents from npm_adv.conf and paste into proxy host's Custom Nginx Configuration under Advanced tab.

Then you should be able to access via https://your_domain_name

## Remove local database login

Change docker compose file, and relauch containers

   EXTENSION_PRIORITY: "* saml" => EXTENSION_PRIORITY: "saml"

