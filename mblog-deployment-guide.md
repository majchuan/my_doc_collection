
# 🚀 Deployment Guide for mBlog (.NET + PostgreSQL) on DigitalOcean with Docker & Nginx

This guide walks you through the complete deployment process for your `mBlog` application.

---

## ✅ 1. Dump Your Local PostgreSQL Database

```bash
pg_dump -U postgres -d mablog -Fc -f mablog.dump
```

---

## ✅ 2. Upload Dump File to Droplet

```bash
scp mablog.dump root@<your_droplet_ip>:/root/
```

---

## ✅ 3. Restore the Database on Droplet

```bash
ssh root@<your_droplet_ip>

# Drop and recreate the database
sudo -u postgres psql -c "DROP DATABASE IF EXISTS mablog;"
sudo -u postgres psql -c "CREATE DATABASE mablog;"

---

## ✅ 4. Create App User & Grant Permissions

```sql
-- In psql (sudo -u postgres psql)
CREATE USER mblog_app_user WITH PASSWORD 'your_secure_password';
GRANT CONNECT ON DATABASE mablog TO mblog_app_user;
\c mablog
GRANT USAGE ON SCHEMA public TO mblog_app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO mblog_app_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO mblog_app_user;
```

# Restore the dump
pg_restore -U postgres -d mablog ~/mablog.dump
```

---

## ✅ 5. Upload the Project to the Droplet

```bash
rsync -avz --progress --exclude='bin' --exclude='obj' /path/to/mBlog root@<your_droplet_ip>:/apps/
```

---

## ✅ 6. Nginx Configuration

Create `/etc/nginx/sites-available/q-runners.com`:

```nginx
server {
    server_name www.q-runners.com q-runners.com;

    location /blog/ {
        proxy_pass http://localhost:8081/blog/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/q-runners.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/q-runners.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    listen 80;
    server_name q-runners.com www.q-runners.com;
    return 301 https://$host$request_uri;
}
```

Enable and reload:
```bash
ln -s /etc/nginx/sites-available/q-runners.com /etc/nginx/sites-enabled/
nginx -t && sudo nginx -s reload
```

---

## ✅ 7. Add Swap Memory (2GB)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# Persist swap across reboots
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

## ✅ 8. Build Docker Image

```bash
cd /apps/mBlog/mBlog
docker build -t mblog .
```

---

## ✅ 9. Run Docker Container

```bash
docker run -d \
  -p 8081:80 \
  -e ASPNETCORE_ENVIRONMENT=Production \
  -e ASPNETCORE_URLS="http://+:80" \
  --name mblog \
  mblog
```

Make sure your app is configured with:

```csharp
app.UsePathBase("/blog");
```

---

## ✅ Notes

- Access your app at: [https://www.q-runners.com/blog](https://www.q-runners.com/blog)
- Ensure images/scripts use `~/` or `@Url.Content("~/...")` paths to resolve correctly under `/blog`
