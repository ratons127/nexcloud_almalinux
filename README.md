# nexcloud_almalinux
nextcloud install in proxmox vm almalinux

To install Nextcloud on AlmaLinux 9, you’ll need to set up a LAMP stack (Linux, Apache, MySQL/MariaDB, PHP) and configure the necessary Nextcloud dependencies. Below are the steps you can follow to install Nextcloud on your AlmaLinux 9 server.

### Step 1: Update Your System
Before starting, it's a good practice to update your system to ensure that all existing packages are up-to-date.
```bash
sudo dnf update -y
```

### Step 2: Install Apache, MariaDB, PHP, and Required Extensions
Next, install the necessary software packages (Apache, MariaDB, PHP, and required PHP extensions).

#### Install Apache
```bash
sudo dnf install httpd -y
```

#### Install MariaDB (MySQL replacement)
```bash
sudo dnf install mariadb-server mariadb -y
```

#### Install PHP and Required Extensions
```bash
sudo dnf install php php-mysqlnd php-xml php-gd php-json php-mbstring php-curl php-zip php-bz2 -y
```

### Step 3: Start and Enable Apache and MariaDB Services
Enable and start the Apache web server and MariaDB service so that they automatically start on boot.
```bash
sudo systemctl enable --now httpd
sudo systemctl enable --now mariadb
```

### Step 4: Secure MariaDB
Run the following command to secure your MariaDB installation (set the root password, remove insecure defaults, etc.):
```bash
sudo mysql_secure_installation
```
Follow the prompts to set the root password and configure other security settings.

### Step 5: Create a Database for Nextcloud
Log into MariaDB as the root user:
```bash
sudo mysql -u root -p
```

Then create the database and user for Nextcloud:
```sql
CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Step 6: Download and Install Nextcloud
Now, you’ll download and install Nextcloud.

#### Change to the Apache Web Root Directory:
```bash
cd /var/www/html
```

#### Download the latest Nextcloud version:
```bash
sudo wget https://download.nextcloud.com/server/releases/nextcloud-26.0.1.zip
```

#### Unzip the downloaded file:
```bash
sudo unzip nextcloud-26.0.1.zip
```

#### Set the correct permissions for the Nextcloud directory:
```bash
sudo chown -R apache:apache /var/www/html/nextcloud
```

### Step 7: Configure Apache for Nextcloud
Create a new configuration file for Nextcloud:
```bash
sudo nano /etc/httpd/conf.d/nextcloud.conf
```

Add the following configuration:
```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/nextcloud
    ServerName yourdomain.com

    <Directory /var/www/html/nextcloud>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
Replace `yourdomain.com` with your actual domain or IP address.

#### Enable the `mod_rewrite` module and restart Apache:
```bash
sudo systemctl restart httpd
```

### Step 8: Adjust the Firewall
Allow HTTP and HTTPS traffic through the firewall:
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Step 9: Finish the Installation through the Web Interface
Now, open a browser and navigate to `http://yourdomain.com` or `http://your-server-ip/nextcloud`. The Nextcloud setup page will appear.

1. **Data Folder**: Choose or confirm the data folder where Nextcloud will store files.
2. **Database Setup**: 
   - Database user: `nextclouduser`
   - Database password: The password you set earlier.
   - Database name: `nextcloud`
   - Database host: `localhost`

3. **Admin Account**: Set up the admin username and password.

4. **Complete the Installation**: Click "Finish setup" to complete the installation.

### Step 10: (Optional) Enable HTTPS with Let's Encrypt
For secure communication, you may want to enable HTTPS using Let's Encrypt.

#### Install Certbot:
```bash
sudo dnf install certbot python3-certbot-apache -y
```

#### Obtain and Install the SSL Certificate:
```bash
sudo certbot --apache -d yourdomain.com
```

This will automatically configure Apache for HTTPS.

### Step 11: Enable Cron Jobs for Nextcloud (Optional but recommended)
To ensure Nextcloud runs background tasks, set up a cron job:

```bash
sudo crontab -u apache -e
```

Add the following line to the crontab:
```bash
*/15  *  *  *  * php -f /var/www/html/nextcloud/cron.php
```

Save and exit.

### Step 12: Access Nextcloud
Now, you should be able to access Nextcloud through your domain or IP address (e.g., `http://yourdomain.com` or `https://yourdomain.com` if using SSL).

---

This should complete your Nextcloud installation on AlmaLinux 9. If you run into any issues or need further customization, feel free to ask!


error
```
sudo dnf install certbot python3-certbot-apache -y
Last metadata expiration check: 0:11:20 ago on Sun 06 Apr 2025 02:53:25 AM UTC.
No match for argument: certbot
No match for argument: python3-certbot-apache
Error: Unable to find a match: certbot python3-certbot-apache
```

It looks like `certbot` is not available in the default AlmaLinux repositories. However, you can install it by enabling the EPEL (Extra Packages for Enterprise Linux) repository or using a different method, such as installing it from the official Certbot repository. Let’s go through a solution:

### Step 1: Enable the EPEL Repository
First, ensure that the EPEL repository is enabled, as Certbot is available there.

```bash
sudo dnf install epel-release -y
```

After installing the EPEL repository, update your system:

```bash
sudo dnf update -y
```

### Step 2: Install Certbot
Now that the EPEL repository is enabled, try installing Certbot again:

```bash
sudo dnf install certbot -y
```

### Step 3: Install the Apache Plugin
If you need the Apache plugin for Certbot (which automatically configures Apache to use SSL), you can install it separately:

```bash
sudo dnf install python3-certbot-apache -y
```

### Step 4: Obtain an SSL Certificate
Once Certbot and the Apache plugin are installed, you can use Certbot to obtain an SSL certificate for your domain:

```bash
sudo certbot --apache -d yourdomain.com
```

Replace `yourdomain.com` with your actual domain.

### Alternative Method: Installing Certbot Using `snapd`
If for some reason the EPEL method doesn’t work, you can install Certbot using `snapd`. This approach works across most Linux distributions, including AlmaLinux.

1. **Install Snapd**:
   ```bash
   sudo dnf install snapd -y
   sudo systemctl enable --now snapd.socket
   ```

2. **Install Certbot via Snap**:
   ```bash
   sudo snap install --classic certbot
   ```

3. **Run Certbot**:
   After installation, you can run Certbot:
   ```bash
   sudo certbot --apache -d yourdomain.com
   ```

This will automatically configure Apache to use SSL for your domain.

### Step 5: Verify SSL Configuration
After you’ve successfully obtained the SSL certificate, verify that Apache is serving the website over HTTPS by visiting `https://yourdomain.com` in your browser.

---

If you encounter any further issues, feel free to let me know!
