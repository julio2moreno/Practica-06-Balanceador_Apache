# Practica-06-Balanceador_Apache
## Creacion de Vagrantfile
Creamos una carpeta donde van a estar ubicadas las maquinas virtuales.

Dentro del directorio creamos el archivo vagrantfile ``vagrant init`` y lo abrimos ``code vagrantfile``.

En el archivo ``vagrantfile`` vamos a poner la instalacion de las maquinas virtuales:
````
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  # Apache HTTP Server Valanceador de Carga
  config.vm.define "balance" do |app|
    app.vm.hostname = "balance"
    app.vm.network "private_network", ip: "192.168.33.10"
    app.vm.provision "shell", path: "provision/provision-for-balance.sh"
  end
  
  # Apache HTTP Server
  config.vm.define "web1" do |app|
    app.vm.hostname = "web1"
    app.vm.network "private_network", ip: "192.168.33.11"
    app.vm.provision "shell", path: "provision/provision-for-apache.sh"
  end

  # Apache HTTP Server2
  config.vm.define "web2" do |app|
    app.vm.hostname = "web2"
    app.vm.network "private_network", ip: "192.168.33.12"
    app.vm.provision "shell", path: "provision/provision-for-apache.sh"
  end

  # MySQL Server
  config.vm.define "db" do |app|
    app.vm.hostname = "db"
    app.vm.network "private_network", ip: "192.168.33.13"
    app.vm.provision "shell", path: "provision/provision-for-mysql.sh"
  end

end
````
Ahora vamos a crear dos carpetas para organizar mejor los datos y tener la ubicacion de los archivos mas controlado.
creamos la carpeta ``config`` donde meteremos el archivo ``000-default.conf`` y otra carpeta ``provision`` para meter los archivos ``provision`` de las maquinas virtuales.

En el archivo ``000-default.conf`` vamos a poner la siguiente configuracion:
````
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html
     <Proxy balancer://mycluster>
        # Server 1
        BalancerMember http://192.168.33.11

        # Server 2
        BalancerMember http://192.168.33.12
    </Proxy>

    ProxyPass / balancer://mycluster/

</VirtualHost>
````
Dentro de la carpeta provision debemos de tener tres archivos.

1.-``provision-for-apache-sh``:
````
#!/bin/bash
apt-get update
apt-get install -y apache2
apt-get install -y php libapache2-mod-php php-mysql
sudo /etc/init.d/apache2 restart
cd /var/www/html
wget https://github.com/vrana/adminer/releases/download/v4.3.1/adminer-4.3.1-mysql.php
mv adminer-4.3.1-mysql.php adminer.php
apt-get install -y git
rm -f /var/www/html/index.html
cd /tmp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git
cp iaw-practica-lamp/src/. /var/www/html/ -R
cd /var/www/html
chown www-data:www-data * -R
apt-get install -y mysql-client-core-5.7
mysql -h 192.168.33.13 -u root -proot < /tmp/iaw-practica-lamp/db/database.sql
sed -i  's/localhost/192.168.33.13/' /var/www/html/config.php
sed -i  's/lamp_user/root/' /var/www/html/config.php
sed -i  's/lamp_user/root/' /var/www/html/config.php
````

2.-``provision-for-mysql.sh``:
````
#!/bin/bash
apt-get update
apt-get -y install debconf-utils

DB_ROOT_PASSWD=root
debconf-set-selections <<< "mysql-server mysql-server/root_password password $DB_ROOT_PASSWD"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DB_ROOT_PASSWD"

apt-get install -y mysql-server
sed -i -e 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
/etc/init.d/mysql restart

mysql -u root mysql -p$DB_ROOT_PASSWD <<< "GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '$DB_ROOT_PASSWD'; FLUSH PRIVILEGES;"
````
3.- ``provision-for-balance``:
````
#!/bin/bash
apt-get update
apt-get install -y apache2
apt-get install -y php libapache2-mod-php php-mysql
a2enmod proxy deflate
a2enmod proxy_http deflate
a2enmod proxy_ajp deflate
a2enmod rewrite deflate
a2enmod deflate deflate
a2enmod headers deflate
a2enmod proxy_balancer deflate
a2enmod proxy_connect deflate
a2enmod proxy_html deflate
a2enmod lbmethod_byrequests deflate
rm -f /etc/apache2/sites-enabled/000-default.conf
cd /etc/apache2/sites-enabled
sudo cp /vagrant/config/000-default.conf .
sudo /etc/init.d/apache2 restart
````

## Comandos utiles 
Para iniciar las maquinas ``vagrant up web`` y ``vagrant up db``.

Hacemos un ``vagrant provision`` para actualizar.

Para acceder ``vagrant ssh web`` o ``vagrant ssh db``

Para ver las maquinas que tenemos iniciadas ``vagrant status`` ó ``vagrant global-status``

