ducking-octo-sansa
==================

###### useful links

* http://www.howtoforge.com/using-php5-fpm-with-apache2-on-ubuntu-12.04-lts

Lets start

```bash
	sudo su
```

```bash 
  	apt-get install apache2-mpm-worker
	apt-get install libapache2-mod-fastcgi php5-fpm php5
	a2enmod actions fastcgi alias
	service apache2 restart
```

FPM module configuration. Can be different for each user. Specify connection type(default to 'socket')

```bash 
	nano /etc/php5/fpm/pool.d/www.conf 
```

All-host FPM configuration. This file doesnt exist, create it.
```bash
	nano /etc/apache2/conf.d/php5-fpm.conf
```
insert that lines
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
