Vagrant.configure("2") do |config|

 config.vm.define "web" do |web_srv|
  web_srv.vm.box = "ubuntu/xenial64"
  web_srv.vm.hostname = "WebServer"
  web_srv.vm.network "forwarded_port", guest: 80, host: 8880
  web_srv.vm.network "private_network", ip: "192.168.19.81"
  web_srv.vm.provider "virtualbox" do |web|
   web.name = "WebServer"
  end

  web_srv.vm.provision "shell",
   inline: <<-script
   sudo apt-get update
   sudo apt-get install -y apache2 php libapache2-mod-php php-curl php-gd php-mbstring php-mcrypt php-mysql php-xml php-xmlrpc

   sudo sh -c "echo '<Directory /var/www/html/>\n    AllowOverride All\n</Directory>' >> /etc/apache2/apache2.conf"
   sudo a2enmod rewrite

   cd /tmp
   sudo curl -O https://wordpress.org/latest.tar.gz
   sudo tar xzvf latest.tar.gz
   sudo touch /tmp/wordpress/.htaccess
   sudo chmod 660 /tmp/wordpress/.htaccess
   sudo mv /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   sudo mkdir /tmp/wordpress/wp-content/upgrade
   sudo cp -a /tmp/wordpress/. /var/www/html
   sudo rm latest.tar.gz
   sudo rm /var/www/html/index.html
   sudo chmod -R 755 /var/www/html
   sudo chown -R ubuntu.www-data /var/www/html

   sudo sed -i 's/database_name_here/wp_db/' /var/www/html/wp-config.php
   sudo sed -i 's/username_here/wp_db_u/' /var/www/html/wp-config.php
   sudo sed -i 's/password_here/password/' /var/www/html/wp-config.php
   sudo sed -i 's/localhost/192.168.19.82/' /var/www/html/wp-config.php

   sudo service apache2 restart
  script
 end

 config.vm.define "db" do |db|
  db.vm.box = "ubuntu/xenial64"
  db.vm.hostname = "DataBase"
  db.vm.network "private_network", ip: "192.168.19.82"
  db.vm.provider "virtualbox" do |db_srv|
   db_srv.name = "DataBase"
  end

  db.vm.provision "shell",
  inline: <<-script
  sudo apt-get update

  sudo debconf-set-selections <<< 'mysql-server-5.1 mysql-server/root_password password password'
  sudo debconf-set-selections <<< 'mysql-server-5.1 mysql-server/root_password_again password password'
  sudo apt-get -y install mysql-server

  sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf


  sudo mysql -u root -ppassword -e "CREATE DATABASE wp_db DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;"
  sudo mysql -u root -ppassword -e "GRANT ALL ON wp_db.* TO 'wp_db_u'@'192.168.19.81' IDENTIFIED BY 'password';"

  sudo service mysql restart

  script

 end
end
