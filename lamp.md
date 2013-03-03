Установка LAMP на ubuntu 12.10
==================

php-fpm + apache2 + mysql

###### Полезные ссылки
* http://www.howtoforge.com/using-php5-fpm-with-apache2-on-ubuntu-12.04-lts
* http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiExternalServer

перед началом
```bash
	sudo su
```

### Установка Apache2

Установка php и apache2
```bash 
  	apt-get install apache2-mpm-worker
	apt-get install libapache2-mod-fastcgi php5-fpm php5
	a2enmod actions fastcgi alias
	service apache2 restart
```

Конфигурация модуля FPM. Можно делать для каждого хоста свою. Следует обратить внимание на тип подключения (```listen = /var/run/php5-fpm.sock```)
```bash 
	nano /etc/php5/fpm/pool.d/www.conf 
```

Стандартная конфигурация модуля FPM для каждого хоста. Этого файла не существует, необходимо его создать
```bash
	nano /etc/apache2/conf.d/php5-fpm.conf
```

Вставьте следующие строки
```htaccess
	<IfModule mod_fastcgi.c>  
		AddHandler php5-fcgi .php  
		
		Action php5-fcgi /php5-fcgi
		Alias /php5-fcgi /usr/lib/cgi-bin/php5-fcgi
		
		FastCGIExternalServer /usr/lib/cgi-bin/php5-fcgi -socket /var/run/php5-fpm.sock -pass-header Authorization
		
		<Directory /usr/lib/cgi-bin/php5-fcgi>  
		  Options ExecCGI FollowSymLinks  
		  SetHandler fastcgi-script  
		  Order allow,deny  
		  Allow from all  
		</Directory>  
	</IfModule>
```

Чтобы убрать назойливое предупреждение апача добавим файл
```bash
	nano /etc/apache2/conf.d/default.conf
```

с таким содержимым
```
	ServerName localhost
```

### Установка PHP

Посмотреть список модулей с кратким описанием можно командой ```bash apt-cache search php5```

Большинство необходимых модулей устанавливается командой
```bash
	apt-get install php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-xdebug
	service php5-fpm restart
```

###### Установка phpunit
```bash
    apt-get install phpunit
```

Для проверки ```phpunit --version```.
В случае ошибки 
```bash
    pear upgrade pear
    pear channel-discover pear.phpunit.de
    pear install --alldeps phpunit/PHPUnit
```

### Установка MySQL

mysql
```bash
	apt-get install mysql-server mysql-client
```

проверка
```bash
	mysql -uroot -p

	mysql> SHOW DATABASES;
	mysql> exit;
```

###### phpmyadmin.
Будет предложена установка настроек по умолчанию для pma. Если вы хотите настройть pma вручную, смотрите доки ```/usr/share/doc/phpmyadmin```. Для корректной работы PMA трeбуется отдельный пользователь и своя база(phpmyadmin по-умолчанию)
```bash
	apt-get install phpmyadmin
```
Не все сборки подключают конфиг к апачу сами, поэтому
```bash
	cd /etc/apache2/conf.d/
	ln -s /etc/phpmyadmin/apache.conf phpmyadmin.conf
```

Для проверки можно пройти по адресу http://localhost/phpmyadmin
