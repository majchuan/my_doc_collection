
# Setting Up Nextcloud with NTFS Drive Access for User and Web Server on Linux Mint

This guide explains how to set up Nextcloud on a Linux Mint server where both:
- The Nextcloud server (running under `www-data`) has access to a data folder on an NTFS drive.
- Your user (`your_user_name`) can also access the entire NTFS drive to work with development files (e.g., compiling .NET code).

---

## 1. Mount the NTFS Drive Properly

Edit the `/etc/fstab` file:

```bash
sudo nano /etc/fstab
```

Add or update the following line:

```
/dev/sdb2  /mnt/nextcloud_data  ntfs-3g  uid=0,gid=0,umask=002,permissions,allow_other  0  0
```

Explanation:
- `uid=0,gid=0`: Gives root ownership. We will manage group/user access manually.
- `umask=002`: Grants write permissions to group members.
- `permissions`: Enables support for standard UNIX file permissions.
- `allow_other`: Allows users other than the mounter to access the mount.

Then mount it:

```bash
sudo mount -a
```

---

## 2. Set Correct Folder Structure

Make sure the Nextcloud data directory exists:

```bash
sudo mkdir -p /mnt/nextcloud_data/nextcloud
```

Then give access to both `www-data` and `your_user_name`:

```bash
sudo chown -R root:users /mnt/nextcloud_data
sudo usermod -aG users your_user_name
sudo usermod -aG users www-data
sudo chmod -R 775 /mnt/nextcloud_data
```

---

## 3. Install Nextcloud

Install dependencies:

```bash
sudo apt update
sudo apt install apache2 mariadb-server libapache2-mod-php php php-mysql php-gd php-json php-xml php-mbstring php-curl php-zip php-intl php-bcmath unzip
```

Download and configure Nextcloud:

```bash
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
sudo mv nextcloud /var/www/
sudo chown -R www-data:www-data /var/www/nextcloud
sudo chmod -R 755 /var/www/nextcloud
```

Create a config file for Apache:

```bash
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

Paste:

```
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/nextcloud/
    ServerName yourdomain.com

    <Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```

Enable Nextcloud site:

```bash
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite headers env dir mime
sudo systemctl reload apache2
```

---

## 4. Point Nextcloud to the External Data Folder

Before running the web installer, change `data` directory in Nextcloud config to use:

```
/mnt/nextcloud_data/nextcloud
```

Ensure the directory is owned and accessible by `www-data`:

```bash
sudo chown -R www-data:www-data /mnt/nextcloud_data/nextcloud
sudo chmod -R 770 /mnt/nextcloud_data/nextcloud
```

---

## 5. Complete Setup via Browser

Visit:

```
http://yourdomain.com
```

During setup:
- Set admin username/password
- Set data directory to `/mnt/nextcloud_data/nextcloud`
- Enter database settings

---

## 6. Ensure dotnet Compiles Work for `your_user_name`

Ensure that `your_user_name` has write access to `/mnt/nextcloud_data`:

```bash
sudo chown -R root:users /mnt/nextcloud_data
sudo chmod -R 775 /mnt/nextcloud_data
sudo usermod -aG users your_user_name
```

Then re-login or reboot to apply group changes.

---

## Notes

- NTFS lacks full UNIX permission support. The `permissions` mount option helps simulate it.
- If possible, consider converting the drive to ext4 for full compatibility.

---

## Optional: Set Up HTTPS

Install certbot:

```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache
```

---

## Done!
