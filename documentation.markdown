## Typical git pull experience

	cd /opt/bitnami/apps/wordpress/htdocs
	git pull

Now if you get tired of that, you could always make an alias out of that, like so:

	vim ~/.bashrc
	alias public='cd /opt/bitnami/apps/wordpress/htdocs'

Be sure to log out of SSH and log back in for the changes to be reflected.

## Enabling short tags in PHP

This lets you do `<?` instead of `<?php`. 

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

It turns out that this is nothing more than a declaration of where the actual virtual host configuration files are. Scroll to the bottom of this file and you'll find where the real stuff is.

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

Actually, what I ended up doing was just move all of the WP files up to their parent directory. This was the quick and dirty way to get the first domain up and running on this server, but now it's coming back to haunt my now that I'm trying to create a secondary virtual host.

## Creating a Virtual Host

Confusingly, [this article](http://wiki.bitnami.org/Components/Apache#How_to_create_a_Virtual_Host.3f) makes it look like you should be able to uncomment the last line in the main httpd.conf file, have it "require" the `/opt/bitnami/apache2/conf/extra/httpd-vhosts.conf` file, and create a Virtual Host pointing to this directory:

	/opt/bitnami/apps/yourapp/htdocs

Unfortunately, doing this and restarting apache results in a 403 error (despite changing owners and permissions to being as wide as possible). Why? No clue. 

Solution? Follow [this tutorial](http://answers.bitnami.org/questions/580/multiple-domains-on-bitnami-wordpress-for-amazon-ec2/2295) and create a virtual host inside  `/opt/bitnami/apache2/conf/extra/httpd-vhosts.conf` that looks something like this:


	<VirtualHost *:80>
		ServerName yourdomain.com
		ServerAlias yourdomain.com
		DocumentRoot "/opt/bitnami/apache2/htdocs/yourdomain.com"
		ErrorLog "logs/error_log"
		CustomLog "logs/access_log" common
	</VirtualHost>

	<Directory "/opt/bitnami/apache2/htdocs/yourdomain.com">
		Allow from all
		Order Deny, Allow
	</Directory>

Restart apache and your new virtual host *should* be up and running. 
