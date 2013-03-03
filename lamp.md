Установка LAMP на ubuntu 12.10
==================

###### Полезные ссылки
* http://www.howtoforge.com/using-php5-fpm-with-apache2-on-ubuntu-12.04-lts

перед началом
```bash
	sudo su
```

#### Установка Apache2

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
