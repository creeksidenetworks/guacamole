# Guacamole / NPM / SAML SSO

---

## Prepare database initialization

```bash
mkdir ./guacdb/init
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > ./guacdb/init/initdb.sql
```
 
