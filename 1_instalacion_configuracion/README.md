Instalación y Configuración
===========================

##Apache


```bash
# Instalacion
$ sudo apt-get install apache2

# Le indicamos al servidor de apache que permita el uso de los archivos .htaccess para cambiar la configuracion del servicio.

#/etc/apache2/apache2.conf
<Directory /var/www/>
->  Options Includes
    Options Indexes FollowSymLinks
->  AllowOverride All
    Require all granted
</Directory>

# Activar modulos de apache para las directivas .htaccess
$ sudo a2enmod rewrite
$ sudo a2enmod setenvif
$ sudo a2enmod headers
$ sudo a2enmod expires
$ sudo a2enmod deflate

# Evitar Informacion Sensible en las cabeceras HTTP que envia en Servidor
$ sudo echo "ServerSignature Off" >> /etc/apache2/apache2.conf
$ sudo echo "ServerTokens Prod" >> /etc/apache2/apache2.conf
```

##MySQL

```bash
# Instalacion de MySQL
$ sudo apt-get install mysql-server
$ sudo apt-get install mysql-client
```

**Configuración del usuario**
Le indicamos al servidor que escuche a todos los clientes que se quieran conectar, no solo a localhost.
```mysql
# /etc/mysql/my.cnf

#bind-address = 127.0.0.1
bind-address = 0.0.0.0

Añadimos un nuevo usuario con privilegios de administrador que se pueda conectar desde cualquier ip

mysql> CREATE USER ‘roman’@‘%’ IDENTIFIED BY ‘o5WKxJkRXw’;
mysql> GRANT ALL PRIVILEGES ON *.* TO ‘roman’@‘%’ WITH GRANT OPTION;
```

##PHP5

```bash
# Instalar php
$ sudo apt-get install php5

# Instalar sus modulos
$ sudo apt-get install php5-intl
$ sudo apt-get install php5-gd
$ sudo apt-get install php5-mysql
$ sudo apt-get install php5-json
```

**Configuración de apache2/php.ini**
```bash
#upload_max_filesize = 2M
upload_max_filesize = 8M

date.timezone = "Europe/Madrid"
short_open_tag = Off
magic_quotes_gpc = Off
register_globals = Off
session.auto_start = Off
```

Copiar estas directivas al archivo **cli/php.ini**
Es necesario reiniciar el servicio de apache para que se efectúen los cambios.

```bash
$ sudo service apache2 restart
```

##Symfony

Instalación de Composer como gestor de dependencias PHP.

```bash
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

```bash
# Descargar symfony
$ sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
$ sudo chmod a+x /usr/local/bin/symfony
```

Para comprobar los requerimientos de symfony dentro del proyecto ejecutamos

```bash
$ php bin/symfony_requeriments
```

Crear e iniciar la aplicación
```bash
$ symfony new lfv
```

Arrancar el servidor
```bash
$ php bin/console server:run
```

Comprobar la configuracion del proyecto accediendo a **http://localhost:8000/config.php**

Actualizando dependencias con Composer

```bash
$ cd lfv
$ composer update
```

Comprobar si las dependencias contienen vulnerabilidades
```bash
$ php bin/console security:check
```

***Autores: Román Arranz Guerrero, Fco Javier Bolívar Lupiáñez*** *(Extraído del libro de Symfony que se puede encontrar en la web oficial del framework)*
