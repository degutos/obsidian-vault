

Reference doc:
https://github.com/kodekloudhub/learning-app-ecommerce


## DB Server

```sh
 sudo yum install mariadb-server
 sudo systemctl start mariadb
 sudo systemctl enable mariadb
 sudo mysql

# configure database MariaDB
CREATE DATABASE ecomdb;
CREATE USER 'ecomuser'@'%' IDENTIFIED BY 'ecompassword';
GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'%';
FLUSH PRIVILEGES;
quit
```


Create the db-load-script.sql in /opt. Run as sudo -i

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

This should be your content

```sh
# cat db-load-script.sql 
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");
```


Then run

```sh
mysql  
```


## Web Server

```sh
sudo yum install httpd php php-mysqlnd -y
```

Make the php page the default page for httpd.

```sh
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

Enable and start httpd

```sh
sudo systemctl enable httpd --now
```

Download code from the git command. 
GIT repo URL: `https://github.com/kodekloudhub/learning-app-ecommerce.git`  
target directory: `/var/www/html/`

```sh
sudo git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

Let us now point the web server to the database server. Update `index.php` to replace the default IP address of the database server to point to `db`

```sh
sudo sed -i 's/172.20.1.101/db/g' /var/www/html/index.php
```


Apache httpd should be responding now on port 80

