# drupal_installation

Installing Drupal on Ubuntu Linux: *A Comprehensive Guide*
This documentation provides a detailed, step-by-step guide to installing Drupal on an Ubuntu Linux system. The process will focus on setting up Drupal 10.1.0 (or the latest version compatible at the time of writing, March 14, 2025) using a LAMP stack (Linux, Apache, MySQL/MariaDB, PHP) and Composer, which is the recommended method for modern Drupal installations. This guide is tailored for Ubuntu 22.04 LTS, but it should work on other recent Ubuntu versions with minor adjustments.
Prerequisites
Before starting, ensure you have the following:
Ubuntu Server: This guide uses Ubuntu 22.04 LTS. Confirm your version with:
bash
lsb_release -a
Sudo Privileges: You need a user with sudo access. If not set up, create one:
bash
adduser username
usermod -aG sudo username
Internet Access: Required to download packages and Drupal files.
Minimum System Requirements:
PHP 8.1 or higher (Drupal 10 requires at least PHP 8.1).
Apache 2.4 or higher.
MySQL 5.7.8+ or MariaDB 10.3+.
At least 2GB of RAM (for performance).
Composer (for dependency management).
Step 1: Update Your System
Ensure your Ubuntu system is up to date to avoid package conflicts and ensure security patches are applied.
Update the package index and upgrade installed packages:
bash
sudo apt update && sudo apt upgrade -y
If a reboot is required (e.g., kernel update), check and reboot:
bash
[ -f /var/run/reboot-required ] && sudo reboot -f
After reboot, reconnect via SSH or your terminal.
Step 2: Install the LAMP Stack
Drupal requires a web server (Apache), a database (MariaDB/MySQL), and PHP. We’ll set up a LAMP stack.
2.1 Install Apache
Install Apache2:
bash
sudo apt install apache2 -y
Start and enable Apache to run on boot:
bash
sudo systemctl start apache2
sudo systemctl enable apache2
Verify Apache is running:
bash
sudo systemctl status apache2
Look for active (running) in the output.
Test Apache by visiting your server’s IP in a browser (e.g., http://your_server_ip):
You should see the default Apache page.
2.2 Install MariaDB
Install MariaDB (a MySQL fork):
bash
sudo apt install mariadb-server mariadb-client -y
Start and enable MariaDB:
bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
Secure the MariaDB installation:
bash
sudo mysql_secure_installation
Follow prompts:
Set a root password (or press Enter if none).
Remove anonymous users: Y.
Disallow root login remotely: Y.
Remove test database: Y.
Reload privilege tables: Y.
2.3 Install PHP and Required Extensions
Drupal 10 requires PHP 8.1+. Ubuntu 22.04 includes PHP 8.1 by default, but let’s ensure the correct version and extensions are installed.
Install PHP and necessary extensions:
bash
sudo apt install php php-cli php-mysql php-gd php-curl php-mbstring php-xml php-zip php-json php-intl php-bcmath -y
Verify PHP version:
bash
php -v
Ensure it’s 8.1 or higher.
Install additional recommended extensions:
bash
sudo apt install php-apcu php-uploadprogress -y
Step 3: Install Composer
Composer is the dependency manager for PHP and the recommended way to install Drupal.
Download and install Composer:
bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
Verify Composer installation:
bash
composer --version
Should output the Composer version (e.g., 2.x.x).
Step 4: Set Up the Database for Drupal
Drupal needs a database to store content and configuration.
Log in to MariaDB:
bash
sudo mysql -u root -p
Enter the root password set during mysql_secure_installation.
Create a database and user for Drupal:
sql
CREATE DATABASE drupal_10_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'drupal_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON drupal_10_db.* TO 'drupal_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
Replace secure_password with a strong password.
Step 5: Install Drupal Using Composer
Navigate to the web root:
bash
cd /var/www/html
Create a directory for Drupal and set permissions:
bash
sudo mkdir drupal
sudo chown -R www-data:www-data drupal
sudo chmod -R 775 drupal
Install Drupal with Composer:
bash
cd drupal
sudo -u www-data composer create-project drupal/recommended-project:10.1.0 .
This installs Drupal 10.1.0 and its dependencies.
Ignore any Deprecation Notice warnings about ${var}—they’re harmless.
Verify the installation:
bash
ls -la
You should see composer.json, vendor, web, etc.
Set final permissions:
bash
sudo chmod -R 755 /var/www/html/drupal
sudo chown -R www-data:www-data /var/www/html/drupal
Step 6: Configure Apache for Drupal
Create an Apache configuration file for Drupal:
bash
sudo nano /etc/apache2/sites-available/drupal.conf
Add the following configuration:
apache
<VirtualHost *:80>
    ServerName localhost
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/drupal/web

    <Directory /var/www/html/drupal/web>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
        RewriteEngine on
        RewriteBase /
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/drupal-error.log
    CustomLog ${APACHE_LOG_DIR}/drupal-access.log combined
</VirtualHost>
Save and exit (Ctrl+O, Enter, Ctrl+X).
Enable the site and required Apache modules:
bash
sudo a2ensite drupal.conf
sudo a2enmod rewrite
sudo a2dissite 000-default.conf  # Disable the default site
Test the Apache configuration:
bash
sudo apache2ctl configtest
Should output Syntax OK.
Restart Apache:
bash
sudo systemctl restart apache2
Step 7: Complete the Drupal Installation via Web Browser
Open your browser and navigate to:
http://localhost/drupal/web
Replace localhost with your server’s IP or domain if accessing remotely.
Follow the Drupal installation wizard:
Choose Language: Select your preferred language (e.g., English) and click Save and continue.
Choose Profile: Select Standard and click Save and continue.
Verify Requirements: If any issues are flagged (e.g., missing PHP extensions), install them with sudo apt install <package> and refresh.
Set Up Database:
Database name: drupal_10_db
Database username: drupal_user
Database password: secure_password
Click Save and continue.
Install Site: Wait for the installation to complete.
Configure Site:
Site name: Enter your site’s name.
Site email: Enter an admin email.
Username: Set an admin username.
Password: Set a strong admin password.
Click Save and continue.
After installation, you’ll see the Drupal welcome page. Log in with your admin credentials.
Step 8: Secure Your Drupal Installation (Optional but Recommended)
Set File Permissions:
After installation, revert settings.php to read-only:
bash
sudo chmod 644 /var/www/html/drupal/web/sites/default/settings.php
Install an SSL Certificate:
Secure your site with HTTPS using Let’s Encrypt:
bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
Follow prompts to set up SSL for your domain.
Firewall Configuration:
Allow HTTP and HTTPS traffic if using UFW:
bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
Step 9: Post-Installation Steps
Install Themes and Modules:
To install a module (e.g., Pathauto):
bash
cd /var/www/html/drupal
sudo -u www-data composer require drupal/pathauto
Enable it in the admin interface: /admin/modules.
Backup Your Site:
Back up files:
bash
cp -r /var/www/html/drupal ~/drupal_backup
Back up the database:
bash
mysqldump -u root -p drupal_10_db > ~/drupal_backup.sql
Explore Drupal:
Navigate to /admin to manage content, users, and settings.
Troubleshooting
Apache Fails to Start:
Check logs:
bash
cat /var/log/apache2/error.log
Ensure DocumentRoot (/var/www/html/drupal/web) exists.
Database Connection Error:
Verify credentials in /var/www/html/drupal/web/sites/default/settings.php.
Ensure MariaDB is running:
bash
sudo systemctl status mariadb
Theme/Module Installation Issues:
Ensure Composer has write permissions:
bash
sudo chown -R www-data:www-data /var/www/html/drupal
Cocoon Bobby Theme (or Similar) Fails:
If using the Cocoon Bobby theme (or any Drupal 8 theme), update its .info.yml to core_version_requirement: ^9 || ^10.
Install the classy base theme dependency:
bash
composer require drupal/classy
Additional Resources
Drupal Documentation: Visit Drupal.org for official guides on theming, module development, and more.
Community Support: Join the Drupal community on Drupal Slack or forums for help.
Conclusion
You’ve successfully installed Drupal 10.1.0 on Ubuntu 22.04 with a LAMP stack. You can now build and customize your website using Drupal’s powerful features. If you encounter issues, review the troubleshooting section or share your error logs for further assistance. What kind of site are you building? I can suggest modules or themes to enhance your setup!
