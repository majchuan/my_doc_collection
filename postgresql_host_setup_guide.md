
# 🐘 PostgreSQL Host Server Configuration Guide

This guide walks you through setting up and configuring PostgreSQL on a **host machine** (not Docker), including remote access, user setup, and secure configuration for production use.

---

## 📦 1. Install PostgreSQL (Ubuntu)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

---

## 🧑‍🔧 2. Set Password for `postgres` User

```bash
sudo -u postgres psql
```

Inside the prompt:

```sql
ALTER USER postgres PASSWORD 'your_secure_password';
\q
```

---

## 📂 3. Allow Remote Connections

### a. Edit `postgresql.conf`

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
```

Find and **uncomment/change** the following line:

```conf
listen_addresses = '*'
```

### b. Edit `pg_hba.conf`

```bash
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

Add the following at the bottom **(adjust IP/CIDR as needed):**

```conf
host    all             all             0.0.0.0/0               md5
```

> 🛡️ To allow only specific IP (e.g. a server), use: `host all all 192.168.1.100/32 md5`

---

## 🔁 4. Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

---

## 👤 5. Create a New Database and User

```bash
sudo -u postgres psql
```

Inside PostgreSQL shell:

```sql
CREATE DATABASE myappdb;
CREATE USER app_user WITH PASSWORD 'strongpassword';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO app_user;
\q
```

---

## 🚀 6. Test Connection from Remote

On your app server or local machine:

```bash
psql -h <your_server_ip> -U app_user -d myappdb
```

Or via Node.js/ORM config:

```js
const pool = new Pool({
  host: 'your_server_ip',
  user: 'app_user',
  password: 'strongpassword',
  database: 'myappdb',
  port: 5432
});
```

---

## 🛡️ 7. (Optional) UFW Firewall Rule

If using UFW on Ubuntu:

```bash
sudo ufw allow 5432/tcp
```

Restrict access:

```bash
sudo ufw allow from <your_app_ip> to any port 5432
```

---

## 🧾 8. Useful PostgreSQL Commands

- List databases: `\l`
- List users: `\du`
- Connect to DB: `\c myappdb`
- Show tables: `\dt`
- Create dump:
  ```bash
  pg_dump -U app_user -d myappdb > myappdb.sql
  ```
- Restore dump:
  ```bash
  psql -U app_user -d myappdb < myappdb.sql
  ```

---

## 🔒 9. Security Tips

- Use strong passwords.
- Restrict IPs in `pg_hba.conf`.
- Run PostgreSQL on a non-default port (optional).
- Regularly update PostgreSQL:
  ```bash
  sudo apt update && sudo apt upgrade
  ```

---

📌 Now your PostgreSQL database is configured on your host server and ready for secure, remote access.
