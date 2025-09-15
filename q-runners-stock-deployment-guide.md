
# Q-Runners Stock App Deployment Guide

This guide covers deploying the `q-runners/stock` full-stack application (React frontend + Node.js backend) to a DigitalOcean droplet using Docker.

---

## 1. Project Structure

```
q-runners/
├── client/stock/        # React frontend
├── server/              # Node.js backend
├── docker-compose.yml   # Docker Compose file
├── nginx/               # Nginx config (if applicable)
```

---

## 2. Push Project to Droplet

Use `rsync` to copy the project from local to the droplet, excluding `node_modules` and `build` directories.
```bash
rsync -avz --progress --exclude='node_modules' --exclude='build' /path/to/q-runners/ username@your_droplet_ip:/apps-dev/q-runners
```

## 2.1 Copy q-runners from apps-dev to apps-prod
```Copy folder, To copy a directory, you need the -r (recursive) flag.
cp -r /source/folder /destination/folder

```copy files
cp -r /source/folder/* /destination/folder/
```bash
rsync -avz --progress --exclude='node_modules' --exclude='build' /path/to/q-runners/ username@your_droplet_ip:/apps-prod/q-runners
```

---

## 3. Set Up Swap Memory (for Docker builds on low memory machines)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## 4. Docker Compose File (`docker-compose.yml`)

Ensure your `docker-compose.yml` includes services for both frontend and backend.

Example:
```yaml
#version: '3.8'
services:
  backend:
    build:
      context: .
      dockerfile: server/Dockerfile
    ports:
      - "3001:3001"
    volumes:
      - ./shared:/usr/src/app/shared:ro
    environment:
      - NODE_ENV=production
      - USER_ID=securityUserName
      - API_KEY='securrityKey'
      - DATABASE_USER=securityDabaseUser
      - HOST=xxx.xxx.xxx.xx.
      - DATABASE=securityDatabaseName
      - PORT=5432
      - PASSWORD=securityPassword
    restart: always

  frontend:
    build:
      context: .
      dockerfile: client/stock/Dockerfile
    ports:
      - "8080:80"
    depends_on:
      - backend
    restart: always
```

---

## 5. Environment Variables

Make sure your backend and frontend services reference `.env` files or Docker `environment:` fields to load API keys and secrets.

---

## 6. Database Initialization (Optional)

If the DB is not seeded via container init, run the following SQL:

```sql
CREATE USER securityUser WITH PASSWORD 'secure_password';
CREATE DATABASE securityDatabaseName OWNER securityUser;
GRANT ALL PRIVILEGES ON DATABASE mstockprofile TO securityUser;
```

Use `psql` inside the container or from host if PostgreSQL is hosted externally.

---

## 7. Build and Run the Application
## 7.1 Build frontend indvidual for small vps. 
REACT_APP_API_URL="" npm run build

From the root project directory:

```bash
docker compose build
docker compose up -d
```

---

## 8. Nginx Configuration (Optional)

If using Nginx as a reverse proxy:

Example for `/stock` subpath:
Edit the file
sudo nano /etc/nginx/sites-available/q-runners.com

```nginx
location /stock/ {
    proxy_pass http://localhost:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

Reload Nginx:

```bash
sudo nginx -s reload
```

---

## 9. Test with cURL

```bash
curl -I http://your_domain_or_ip/stock/
```

---

## 10. Troubleshooting

- Use `docker logs <container_name>` to check logs.
- Check file permissions and ownership if files are not served properly.
- Use `docker compose down` to tear down services if needed.

## 11. Set Up HTTPS with Let's Encrypt Certificate

step 1: install Cerbot and Nginx plugin
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

step 2: Run Cerbot to Obtain and COnfigure SSL
sudo certbot --nginx -d www.q-runners.com -d q-runners.com

Certbot will:

Get a certificate from Let's Encrypt

Automatically configure your Nginx to redirect HTTP → HTTPS

Ask for your email and consent for Let's Encrypt terms

step3: Reolad Nginx Manually
sudo nginx -t
sudo systemctl reload nginx


## 11. Configure customer domain on SquareCommerce

step 1 : login into the SquareCOmmerce dashboard, navigate to Domains

step 2 : select the domain you want to point (www.q-runners.com)
         click Manage DNS settings
step 3 : To point your domain to your custom server(DigitalOcean droplet)

RecordType	Name / Host	  Value (Target)	            TTL
A	              @	        <Your Droplet Public IP>	  Auto
A	              www	      <Your Droplet Public IP>	  Auto

Replace <Your Droplet Public IP> with the public IP of your VPS

step 4 : Wait for DNS propagation

step 5 : COnfirm SSL is working
