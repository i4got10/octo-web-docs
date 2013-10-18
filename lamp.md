Установка LAMP на ubuntu 12.10
==================

Что в итоге получится:
> php-fpm + apache2(mod_fastcgi + mod_suexec) + mysql

> либо php-fpm + nginx proxy + apache2

> либо php-fpm + nginx

#### Полезные ссылки на эту тему
* http://www.howtoforge.com/using-php5-fpm-with-apache2-on-ubuntu-12.04-lts
* http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiExternalServer
* http://x10hosting.com/forums/vps-tutorials/148894-debian-apache-2-2-fastcgi-php-5-suexec-easy-way.html
* http://learnix.net/fastasscgi-part2/

Перед началом работы
```bash
sudo su
```

в случае каких-то изменений в конфигах
```bash
service php5-fpm restart && service apache2 restart
```

## Установка php и apache2

Репозиторий в котором есть последняя версия php5.4
```bash
add-apt-repository ppa:ondrej/php5-oldstable
apt-get update
apt-get upgrade
```

Установка пакетов
```bash 
apt-get install apache2-mpm-worker
apt-get install libapache2-mod-fastcgi php5-fpm php5 apache2-suexec-custom
a2enmod actions fastcgi alias suexec rewrite
```

Возможно придется раскоментировать строку в файле```/etc/apache2/mods-available/fastcgi.conf```

```
FastCgiWrapper /usr/lib/apache2/suexec
``` 

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
; php_flag[display_errors] = on
; php_flag[display_startup_errors] = on
; php_admin_value[memory_limit] = 256M
```

В папке ```/etc/php5``` лежат ini, отвечающие за настройки php в определенном режиме работы (*apache2/php.ini*, *cli/php.ini*, *fpm/php.ini*).
Чтобы применить настройки сразу везде, удобно положить настройки в файл ```/etc/php5/conf.d/30-alex.ini```. Пример такого файла

```ini
; configuration for php dbase module
extension=dbase.so

[PHP]
display_errors = On
display_startup_errors = On

memory_limit = 256M
upload_max_filesize = 32M
post_max_size = 32M

; xdebug
xdebug.max_nesting_level = 300
; http://www.xdebug.org/docs/profiler(enabled by ?XDEBUG_PROFILE)
xdebug.profiler_enable = 0
xdebug.profiler_enable_trigger = 1
xdebug.profiler_output_dir = /home/alex/xdebug_profiler
xdebug.profiler_output_name = cachegrind.%H

; увеличения времени жизни cookie
session.gc_maxlifetime = 2592000
```

Чтобы убрать назойливое предупреждение апача при запуске добавим файл ```/etc/apache2/conf.d/default.conf``` с таким содержимым
```
ServerName localhost
```

Пример конфигурации Apache 2.2 для сайта. Создается в папке */etc/apache2/sites-available*, далее делается symlink в папку */etc/apache2/sites-enabled*.

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

Необходимо не забыть добавить алиас в ```/etc/hosts```
```
echo "127.0.0.1 weburg-parser.local" >> /etc/hosts
```

### Установка PHP

Посмотреть список модулей с кратким описанием можно командой ```apt-cache search php5```

Большинство необходимых модулей устанавливается командой
```bash
apt-get install php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcached php5-xdebug php5-sqlite
service php5-fpm restart
```

#### Phpunit
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

```bash
apt-get install mysql-server mysql-client
```

проверка
```bash
mysql -uroot -p

mysql> SHOW DATABASES;
mysql> exit;
```

#### phpmyadmin
Будет предложена установка настроек по умолчанию для pma. 
Если вы хотите настройть pma вручную, смотрите доки ```/usr/share/doc/phpmyadmin```. 
Для корректной работы PMA трeбуется отдельный пользователь и дополнительная база данных(phpmyadmin по-умолчанию).
Для настройки можно использовать [скрипт настройки](http://localhost/phpmyadmin/doc/html/setup.html#setup_script).
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

Скачать послежнюю версию можно [тут](http://www.phpmyadmin.net/home_page/downloads.php). Распаковать в `/usr/share/phpmyadmin`

Если не работает вкладка настройки - https://bugs.launchpad.net/ubuntu/+source/phpmyadmin/+bug/1175142

#### Сниппеты

Для проверки работоспособости php, и, в особенности, SuExec можно сделать такой index.php
```php
<?php echo system('whoami'); ?>
<?php phpinfo(); ?>
```

#### Memcached
```bash
apt-get install memcached
```

## Установка nginx

```bash
apt-get install nginx
service nginx start
```

Повесим apache на 8080 порт
```bash
perl -e "s/:80/:8080/g;" -pi $(find /etc/apache2/sites-available -type f)
perl -e "s/80/8080/g;" -pi $(find /etc/apache2/ports.conf -type f)
```

Пример конфигурации для сайта

```nginxconf
server {
    listen 80;

    server_name megavika.local megavika.work;
    root /home/alex/PhpstormProjects/megavika/web;

    location / {
        # try to serve file directly, fallback to rewrite
        try_files $uri @rewriteapp;
    }

    location @rewriteapp {
        # rewrite all to app.php
        rewrite ^(.*)$ /app_dev.php/$1 last;
    }

    location ~ ^/(app|app_dev|config)\.php(/|$) {
        fastcgi_pass unix:/var/run/php5-fpm-alex.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
    }

    # Logging
    error_log /var/log/nginx/megavika.local-error.log;
    access_log /var/log/nginx/megavika.local-access.log;
}
```

Пример конфигурации для сайта с переадресацией на apache

```nginxconf
upstream apache {
    server localhost:8080;
}

server {
    listen 80;

    server_name qsb.local qsb.work;
    root /home/alex/PhpstormProjects/quicksilver-boats/web;

    # Перенаправление на back-end
    location / {
        proxy_pass http://apache;
        include /etc/nginx/proxy_params;
    }

    # Статическиое наполнение отдает сам nginx
    # back-end этим заниматься не должен
    location ~* \.(jpg|jpeg|gif|png|ico|css|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|js|mov)$ {
        root /home/alex/PhpstormProjects/quicksilver-boats/web/;
    }

    # Logging
    error_log /var/log/nginx/qsb.local-error.log;
    access_log /var/log/nginx/qsb.local-access.log;
}
```

Конфиг PMA для nginx можно добавить в ```gedit /etc/nginx/sites-available/default```

```nginxconf
location /phpmyadmin/ {
	root /usr/share/;
	index index.php index.html index.htm;
	location ~ ^/phpmyadmin/(.+\.php)$ {
		root /usr/share/;
		fastcgi_pass unix:/var/run/php5-fpm-alex.sock;
		fastcgi_split_path_info ^(.+\.php)(/.*)$;
		include fastcgi_params;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		fastcgi_param HTTPS off;
		fastcgi_index index.php;
	}
	location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
	        root /usr/share/;
	}
}
location /phpMyAdmin {
	rewrite ^/* /phpmyadmin last;
}
location /mysql {
	rewrite ^/* /phpmyadmin last;
}
```
