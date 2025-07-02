# linux-wsl2-php-dev-env-config
WSL2/Linux PHP dev environment configuration

**Keep in mind, that this configuration is for development purposes only. It's meant to be run on local environments, not production ones.**

# 1.1 If you are using WSL2, enable and install it:

```
wsl --install
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --set-default-version 2
```

# 1.2 Install Ubuntu

# 2. Install dependencies

This will install PHP 8.4, mysql and apache2. You can install multiple versions of PHP and switch between them.

```
sudo add-apt-repository ppa:ondrej/php 
sudo apt install -y php8.4-fpm php8.4-mbstring php8.4-curl php8.4-bz2 php8.4-zip php8.4-xml php8.4-gd php8.4-mysql php8.4-intl php8.4-sqlite3 php8.4-soap php8.4-bcmath php8.4-memcached php8.4-redis php8.4-xmlrpc php8.4-imagick apt-transport-https apache2 mysql-client mysql-server libapache2-mod-php8.4 nano
```

# 3.1 MySQL configuration

`sudo usermod -d /var/lib/mysql/ mysql`

Then, we edit config file:

`sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

```
port = <port-number>
bind-address = 0.0.0.0
```

Port number is optional. Set if needed other than default 3306.

Bind address will allow you to connect from any IP.

At the end, restart MySQL:

`sudo systemctl restart mysql`

# 3.2 Create MySQL user

Switch user to mysql:

`sudo mysql`

Create user:

`CREATE USER 'user'@'%' IDENTIFIED BY 'password';`

Grant privileges:

`GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';`

`FLUSH PRIVILEGES;`

# 3.3 Configure MySQL timezones:

`mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql -p`

# 4.1 Configure Apache2:

This step is optional, but it will allow you to easily put and access project files from home directory.

Enable userdir mod:

`sudo a2enmod userdir`

Add execute permissions to user home directory:

`sudo chmod +x /home/user`

Edit configuration:

`sudo nano /etc/apache2/mods-available/userdir.conf`

```
UserDir projects
UserDir disabled root

<Directory /home/*/projects>
        AllowOverride All
        Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
        Require all granted
</Directory>
```

Create directory:

`mkdir /home/user/projects && chmod -R 755 /home/user/projects`

Restart Apache:

`sudo systemctl restart apache2`

Now, you can access files using `http://<server-address>/~user`

Remember, to comment disabling of PHP execution in user directory (depending on the config):

`sudo nano /etc/apache2/mods-available/php8.4.conf`

```
#<IfModule mod_userdir.c>
#    <Directory /home/*/public_html>
#        php_admin_flag engine Off
#    </Directory>
#</IfModule>
```

# 4.2 Configure sites-enabled:

This will allow to access project files using domain name.

Create new file:

`sudo nano /etc/apache2/sites-available/my-site.conf`

```
<VirtualHost *:80>
    ServerName my-site.test
    DocumentRoot /home/user/projects/my-site

    <Directory /home/user/projects/my-site>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Remember to configure hosts file, so that domain name will be resolved to localhost.

Enable site:

`sudo a2ensite my-site`

Enable rewrite mod:

`sudo a2enmod rewrite`

Restart Apache:

`sudo systemctl restart apache2`

# 5. Configure sendmail:

To easily capture and read emails, we can do a little trick:

```sudo nano \usr\local\bin\sendmail```

```php
#!/usr/bin/php
<?php
$input = file_get_contents('php://stdin');
preg_match('|^To: (.*)|', $input, $matches);
$filename = '/var/log/mail/' . time() . '.eml';
file_put_contents($filename, $input);
```

After doing that, email will be accessible in `/var/log/mail` directory.

# 6. Switching between PHP versions:

Disable PHP version:

`sudo a2dismod php7.4`

Enable PHP version:

`sudo a2enmod php8.4`

Change PHP used in system:

`sudo update-alternatives --config php`
