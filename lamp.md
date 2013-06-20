Установка LAMP на ubuntu 12.10
==================

php-fpm + apache2(mod_fastcgi + mod_suexec) + mysql

###### Полезные ссылки на эту тему
* http://www.howtoforge.com/using-php5-fpm-with-apache2-on-ubuntu-12.04-lts
* http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiExternalServer
* http://x10hosting.com/forums/vps-tutorials/148894-debian-apache-2-2-fastcgi-php-5-suexec-easy-way.html
* http://learnix.net/fastasscgi-part2/

перед началом
```bash
sudo su
```

в случае каких-то изменений в конфигах
```bash
service php5-fpm restart && service apache2 restart
```

### Установка Apache2

Установка php и apache2
```bash 
apt-get install apache2-mpm-worker
apt-get install libapache2-mod-fastcgi php5-fpm php5 apache2-suexec-custom
a2enmod actions fastcgi alias suexec rewrite
```

Возможно придется раскоментировать строку ```FastCgiWrapper /usr/lib/apache2/suexec``` ```nano /etc/apache2/mods-available/fastcgi.conf```

Конфигурация модуля FPM. Делается для каждого пользователя, можно определять дополнительные настройки php.ini
```bash 
nano /etc/php5/fpm/pool.d/alex.conf 
```

пример файла
```ini
; pool name
[alex]

; system user
user = alex
group = alex

; listen socket
listen = /var/run/php5-fpm-alex.sock

;process manager
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

; chroot 
chdir = /

; PHP ini settings
php_flag[display_errors] = on
php_flag[display_startup_errors] = on
php_admin_value[memory_limit] = 256M
php_admin_value[upload_max_filesize] = 32M
php_admin_value[post_max_size] = 32M
php_admin_value[xdebug.max_nesting_level] = 300
```

Чтобы убрать назойливое предупреждение апача добавим файл
```bash
nano /etc/apache2/conf.d/default.conf
```

с таким содержимым
```
ServerName localhost
```

Пример конфигурации для сайта
```apacheconf
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	ServerName weburg-parser.local
	SuexecUserGroup alex alex
	DocumentRoot /home/alex/public_html/weburg-parser.local/www
	DirectoryIndex index.php index.html
	
	<Directory />
		Options FollowSymLinks
		AllowOverride None
	</Directory>
	
	<Directory /home/alex/public_html/weburg-parser.local/www/>
		Options ExecCGI FollowSymLinks
		AllowOverride all
		Order allow,deny
		Allow from all
	</Directory>
	
	<IfModule mod_fastcgi.c>  
		AddHandler php5-fcgi .php  
		
		Action php5-fcgi /php5-fcgi
		Alias /php5-fcgi /usr/lib/cgi-bin/php5-fcgi
		
		FastCGIExternalServer /usr/lib/cgi-bin/php5-fcgi -socket /var/run/php5-fpm-alex.sock -user alex -group alex
		
		<Directory /usr/lib/cgi-bin/php5-fcgi>  
			Options ExecCGI FollowSymLinks  
			SetHandler fastcgi-script  
			Order allow,deny  
			Allow from all  
		</Directory>  
	</IfModule>
	
	ErrorLog ${APACHE_LOG_DIR}/error.log
	
	# Possible values include: debug, info, notice, warn, error, crit, alert, emerg.
	LogLevel warn
	
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
```
echo "127.0.0.1 weburg-parser.local" >> /etc/hosts
```

### Установка PHP

Посмотреть список модулей с кратким описанием можно командой ```apt-cache search php5```

Большинство необходимых модулей устанавливается командой
```bash
apt-get install php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-xdebug php5-sqlite
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

###### phpmyadmin
Будет предложена установка настроек по умолчанию для pma. Если вы хотите настройть pma вручную, смотрите доки ```/usr/share/doc/phpmyadmin```. Для корректной работы PMA трeбуется отдельный пользователь и своя база(phpmyadmin по-умолчанию)
```bash
add-apt-repository ppa:nijel/phpmyadmin
apt-get update
apt-get install phpmyadmin
```
PMA требует доступ к файлам в папке для подключения своих конфигов ```ls -l /etc/phpmyadmin/```(такое поведение определено в файле ```/etc/phpmyadmin/config.inc.php```), поэтому
```bash
usermod -a -G www-data alex
```
Не все сборки подключают конфиг к апачу сами, поэтому
```bash
cd /etc/apache2/conf.d/
ln -s /etc/phpmyadmin/apache.conf phpmyadmin.conf
```

Возможно вы захотите увеличить время жизни cookie в ```/etc/phpmyadmin/config.inc.php```
```php
$cfg['Servers'][$i]['LoginCookieValidity'] = 2592000; # 1 month
```

Для проверки можно пройти по адресу [http://localhost/phpmyadmin](http://localhost/phpmyadmin)

Для проверки работоспособости php, и, в особенности, SuExec можно сделать такой index.php
```php
<?php echo system('whoami'); ?>
<?php phpinfo(); ?>
```
