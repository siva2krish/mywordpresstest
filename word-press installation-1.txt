###binbash##

###WordPress Deployment Automation script ##



sudo apt-get -y  update
sudo apt-get -y  install apache2
sudo apache2ctl configtest
apache2ctl configtest
sudo systemctl restart apache2
sudo ufw app list
sudo ufw app info "Apache Full"
sudo apt-get -y  install curl
curl http://icanhazip.com
dbpass=snp001

export DEBIAN_FRONTEND=noninteractive
echo mysql-server mysql-server/root_password password $dbpass | debconf-set-selections
echo mysql-server mysql-server/root_password_again password $dbpass | debconf-set-selections
sudo apt-get -y install mysql-server*
sudo apt-get -y  install php libapache2-mod-php php-mcrypt php-mysql
sudo systemctl restart apache2
sudo systemctl status apache2
#apt-cache search php- | less
#apt-cache show package_name
apt-cache show php-cli
sudo apt-get -y  install php-cli
sudo apt-get -y  install  php5-mysqlnd-ms
sudo apt-get -y  install php-mysql
#sudo apt-get -y  install package1 package2 ...
dbusername="root"
dbpass="snp001"
echo "create DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;" |  mysql -u $dbusername -p$dbpass
#echo "GRANT ALL ON wordpress.* TO 'root'@'localhost' IDENTIFIED BY 'snp0001';" | mysql -u $dbusername -p$dbpass
echo "FLUSH PRIVILEGES;" |  mysql -u $dbusername -p$dbpass
echo "EXIT;" | mysql -u $dbusername -p$dbpass
sudo apt-get -y  update
sudo apt-get -y  install php-curl* php-gd* php-mbstring* php-mcrypt* php-xml* php-xmlrpc*
sudo systemctl restart apache2
sudo apache2ctl configtest
sudo systemctl restart apache2
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
touch /tmp/wordpress/.htaccess
chmod 660 /tmp/wordpress/.htaccess
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
mkdir /tmp/wordpress/wp-content/upgrade
sudo cp -a /tmp/wordpress/. /var/www/html
sudo chown -R root:root /var/www/html
sudo find /var/www/html -type d -exec chmod g+s {} \;
sudo chmod g+w /var/www/html/wp-content
sudo chmod -R g+w /var/www/html/wp-content/themes
sudo chmod -R g+w /var/www/html/wp-content/plugins
curl -s https://api.wordpress.org/secret-key/1.1/salt/
cd /var/www/html
mv wp-config.php /tmp/
wget http://220.226.22.115/configfiles/wp-config.php1
mv  wp-config.php1 /home/
cd /home/
mv wp-config.php1 wp-config.php
mv  wp-config.php /var/www/html
cd /var/www/html
chmod 777 wp-config.php
chown root:root *
cd /home/
wget http://220.226.22.115/configfiles/dir.conf
cp -r dir.conf /etc/apache2/mods-enabled/
/etc/init.d/apache2 restart
/etc/init.d/apache2 status
/etc/init.d/mysql status
echo "wordpress installationtion sucessfully completed"