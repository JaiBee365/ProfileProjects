#!/bin/bash

# Update packages
yum update -y

# Navigate to /home/ec2-user
cd /home/ec2-user

# Download WordPress
wget https://wordpress.org/latest.tar.gz

# Extract WordPress
tar -xzf latest.tar.gz

# Enable PHP 8.2
amazon-linux-extras enable php8.2
yum clean metadata

# Install necessary packages
yum install httpd -y
yum install php -y 
yum install php-xml -y
yum install mariadb-server -y
yum install php-mysql -y 
yum install php-fpm -y
yum install epel-release -y 
yum install phpmyadmin -y
yum install rpm-build -y
yum install redhat-rpm-config -y
yum install php-mysqlnd -y

# Start and enable httpd
systemctl start httpd
systemctl enable httpd

# Start and enable MariaDB
systemctl start mariadb
systemctl enable mariadb

# Create database and user
mysql -u root <<-EOF
CREATE USER 'wordpress-user'@'localhost' IDENTIFIED BY 'YOUR_DB_PASSWORD';
CREATE DATABASE `wordpress-db`;
GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user"@"localhost";
FLUSH PRIVILEGES;
EOF

# Restart httpd
systemctl restart httpd

# Configure WordPress
cp /home/ec2-user/wordpress/wp-config-sample.php /home/ec2-user/wordpress/wp-config.php
sed -i "s/database_name_here/YOUR_DB_NAME/g" /home/ec2-user/wordpress/wp-config.php
sed -i "s/username_here/YOUR_DB_USERNAME/g" /home/ec2-user/wordpress/wp-config.php
sed -i "s/password_here/YOUR_DB_PASSWORD/g" /home/ec2-user/wordpress/wp-config.php

# Replace security keys in the wp-config.php file
# Note: Use a service to generate unique security keys for your WordPress installation.
declare -A keys=(
    ["AUTH_KEY"]='YOUR_AUTH_KEY' \
    ["SECURE_AUTH_KEY"]='YOUR_SECURE_AUTH_KEY' \
    ["LOGGED_IN_KEY"]='YOUR_LOGGED_IN_KEY' \
    ["NONCE_KEY"]='YOUR_NONCE_KEY' \
    ["AUTH_SALT"]='YOUR_AUTH_SALT' \
    ["SECURE_AUTH_SALT"]='YOUR_SECURE_AUTH_SALT' \
    ["LOGGED_IN_SALT"]='YOUR_LOGGED_IN_SALT' \
    ["NONCE_SALT"]='YOUR_NONCE_SALT'
)

for key in "${!keys[@]}"; do
    sed -i "s/define('$key', 'put your unique phrase here');/define('$key', '${keys[$key]}');/g" /home/ec2-user/wordpress/wp-config.php
done

# Add the FS_METHOD constant to wp-config.php
sed -i "/define( 'DB_COLLATE', '' );/a define('FS_METHOD', 'direct');" /home/ec2-user/wordpress/wp-config.php

# Copy WordPress files to the web root
sudo cp -r /home/ec2-user/wordpress/* /var/www/html/
sleep 5

# Set file permissions
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/

# Backup httpd.conf
sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
