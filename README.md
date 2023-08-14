WordPress Setup Script for Amazon Linux EC2 
This Bash script automates the setup of WordPress on an Amazon Linux EC2 instance. 
When executed:
    It updates the installed packages using yum.
    Downloads the latest version of WordPress.
    Enables PHP 8.2 and installs necessary packages including Apache, PHP, MariaDB, and phpMyAdmin.
    Starts and sets Apache and MariaDB to run on boot.
    Creates a new database and user for WordPress.
    Configures WordPress using a sample configuration, including setting up unique security keys.
    Copies WordPress files to the Apache web root (/var/www/html).
    Sets the appropriate file permissions for the WordPress installation.
    Lastly, it backs up the Apache configuration file.

Note: Before running, make sure to replace placeholder values like 'YOUR_DB_PASSWORD' and 'YOUR_AUTH_KEY' with actual values.
