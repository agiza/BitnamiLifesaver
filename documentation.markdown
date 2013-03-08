## Typical git pull experience

	cd /opt/bitnami/apps/wordpress/htdocs
	git pull

Now if you get tired of that, you could always make an alias out of that, like so:

	vim ~/.bashrc
	alias public='cd /opt/bitnami/apps/wordpress/htdocs'

Be sure to log out of SSH and log back in for the changes to be reflected.

## Enabling short tags in PHP

This lets you do `<?` instead of `<?php. 

	vim /opt/bitnami/php/etc/php.ini
	:/short_open_tag 
	# turn it from "Off" to "On", then restart Apache

## Creating a remote MySQL login

Hopefully your GitHub isn't going to have configuration stored in it (that's a great time to introduce a .gitignore exception) but in case you don't have this luxury, it's nice to have a remote MySQL login so that the sandbox and the production site will both work with the same login.

	vim /opt/bitnami/mysql/my.cnf
	bind-address = 10.10.10.10 # Used to say "127.0.0.1", changing to my Bitnami IP
	:wq

	mysql -u root -p
	CREATE USER 'remote'@'%' IDENTIFIED BY 'b76hEfUC';
	GRANT ALL PRIVILEGES ON myDatabaseName.* TO remote;
	
This may not be incredibly secure but I love it an use it all the time.

(Note: I was able to get this far but never actually got it working. Why? Something to do with the fact that they're only allowing port 3306. So, this idea is merely a work in progress).

## How to restart Apache2
	sudo /opt/bitnami/ctlscript.sh restart apache

## Edit the HTTP.conf file: 
	/opt/bitnami/apache2/conf/httpd.conf

It turns out that this is nothing more than a declaration of where the actual virtual host configuration files are. The real one for WordPress is here:

	/opt/bitnami/apps/wordpress/conf/wordpress.conf

## Default public directory for primary domain/IP:

	~/htdocs

## Default public directory for WordPress:
	/opt/bitnami/apps/wordpress/htdocs

## Helpful forums
[http://answers.bitnami.org/](http://answers.bitnami.org/)

## Default MySQL information (subject to change with installation, so always check the wp-config.php)
- database: bitnami_wordpress
- username: bn_wordpress
- password: 221e55cff4
- server IP: localhost:3306

## MySQL for root user

Sometimes it's helpful to have root MySQL access. It's not necessary for the WordPress site but for serious work in the phpMyAdmin (such as creating new databases) it's a much. Hilariously, here are the very unstable defaults:

- username: root
- password: bitnami

## Default WordPress login credentials (subject to change, but this time I think not so much lol)
- username: user
- password: bitnami

## Enabling PHPMyAdmin

As per [these instructions](http://bitnami.org/faq/virtual_machines):

	vim /opt/bitnami/apps/phpmyadmin/conf/phpmyadmin.conf
	
Find "allow from 127.0.0.1" and change this to: "Allow from "all". Actually, this didn't work for me, so I replaced the whole phpmyadmin.conf with this:

	Alias /phpmyadmin "/opt/bitnami/apps/phpmyadmin/htdocs"
	 
	<Directory "/opt/bitnami/apps/phpmyadmin/htdocs">
	AuthType Basic
	AuthName phpMyAdmin
	AuthUserFile "/opt/bitnami/apache2/users"
	Require valid-user
	Order allow,deny
	Allow from all
	Satisfy all
	ErrorDocument 403 "For security reasons, this URL is only accesible using localhost (127.0.0.1) as the hostname"
	</Directory>

Then go to /phpmyadmin/ in the browser and you'll be prompted for a username and password. These are as follows:

- username: administrator
- password: bitnami

Then you'll be prompted for your actual database credentials, which are shown above.

## Git

I had serious problems getting git installed. This didn't work:

	sudo apt-get install git

It was returning this error:
	
	E: Unable to locate package git-core

It's too late for me to even care about why this next thing worked, but I pasted that error directly into Google and got [this link](http://ubuntuforums.org/showthread.php?t=1778843) which told us to run this:

	sudo apt-get update && sudo apt-get install git-core

This worked like a charm:
	
	git init
	Initialized empty Git repository in /opt/bitnami/apache2/htdocs/.git/

## Configuring WordPress to be the primary directory

Scrolling halfway down, this is the bomb. Follow it extremely closely. 

http://wiki.bitnami.org/Applications/BitNami_Wordpress_Stack#How_to_change_the_WordPress_domain_name.3f
