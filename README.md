This repository contains a Vagrant configuration to set up a local WordPress development environment using Ubuntu 20.04 (Focal Fossa). The setup includes Apache, MySQL, PHP, and WordPress installation and configuration.

Prerequisites:
Vagrant (version 2.2.10 or higher)
VirtualBox (version 6.1 or higher)

To get started
Clone this repository to your local machine:
    git clone https://github.com/suliatkazeem/automating-wordpress-setup.git
    cd automating-wordpress-setup

Start the Vagrant environment by running:
    vagrant up
    
Vagrant will download the Ubuntu 20.04 box, set up the virtual machine, and provision it with the necessary software to run WordPress.

Configuration Details
The Vagrantfile included in this repository configures the Vagrant environment as follows:
- Base Box: ubuntu/focal64
- Networking:
    Private network with IP: 192.168.56.31
    Public network (bridged network)
- Provider Configuration:
    Using VirtualBox with 1600MB of memory allocated
- Provisioning:
    Updates the package list and installs Apache, MySQL, PHP, and required PHP modules
    Downloads and sets up WordPress
    Configures Apache to serve WordPress
    Creates a MySQL database and user for WordPress
    Configures WordPress to connect to the MySQL database
  
The provisioning script included in the Vagrantfile performs the following steps:

1. Updates the package list and installs necessary packages:
    sudo apt update
    sudo apt install apache2 ghostscript libapache2-mod-php mysql-server php php-bcmath php-curl php-imagick php-intl php-json php-mbstring php-mysql php-xml php-zip -y
   
2. Sets up the directory for WordPress and downloads the latest WordPress version:
    sudo mkdir -p /srv/www
    sudo chown www-data: /srv/www
    curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
   
3. Configures Apache to serve WordPress:
    cat > /etc/apache2/sites-available/wordpress.conf <<EOF
    <VirtualHost *:80>
        DocumentRoot /srv/www/wordpress
        <Directory /srv/www/wordpress>
            Options FollowSymLinks
            AllowOverride Limit Options FileInfo
            DirectoryIndex index.php
            Require all granted
        </Directory>
        <Directory /srv/www/wordpress/wp-content>
            Options FollowSymLinks
            Require all granted
        </Directory>
    </VirtualHost>
    EOF

    sudo a2ensite wordpress
    sudo a2enmod rewrite
    sudo a2dissite 000-default
   
4. Sets up the MySQL database and user for WordPress:
    mysql -u root -e 'CREATE DATABASE wordpress;'
    mysql -u root -e 'CREATE USER wordpress@localhost IDENTIFIED BY "admin123";'
    mysql -u root -e 'GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost;'
    mysql -u root -e 'FLUSH PRIVILEGES;'

5. Configures WordPress to connect to the MySQL database:
    sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
    sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
    sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
    sudo -u www-data sed -i 's/password_here/admin123/' /srv/www/wordpress/wp-config.php
   
7. Restarts MySQL and Apache to apply changes:
    systemctl restart mysql
    systemctl restart apache2
   
Once the setup is complete, you can access the WordPress site by navigating to http://192.168.56.31 in your web browser.

To stop the Vagrant environment, run:
    vagrant halt
OR
    vagrant destroy

Contributions are welcome! Please submit a pull request or open an issue to discuss your ideas.
