#Document d'installation

========================================================================================================================
Installation, configuration et premiers pas avec nginx et php-fpm :
========================================================================================================================
Cette installation est appliquable sur Centos 6 en version minimale sans apache installé !
Si Apache est installé, arrêtez le service ou  adaptez ce tutorial à l'aide du site suivante :
http://www.if-not-true-then-false.com/2011/install-nginx-php-fpm-on-fedora-centos-red-hat-rhel/
------------------------------------------------------------------------------------------------------------------------
MARCHE A SUIVRE

1. Passer en root 

2. Installer les rep. nécessaires
	
  	 rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
  	 rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

3. Créer le fichier /etc/yum.repos.d/nginx.repo et ajouter le contenu suivant:

	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
	gpgcheck=0
	enabled=1

4. Installer nginx, php 5.5.3 et php-fpm

	yum --enablerepo=remi,remi-test install nginx php-fpm php-common

5. Installer les modules de PHP 5.5.0 que vous souhaitez (liste ci-dessous), à adapter :

	APC (php-pecl-apc) – APC caches and optimizes PHP intermediate code
	CLI (php-cli) – Command-line interface for PHP
	PEAR (php-pear) – PHP Extension and Application Repository framework
	PDO (php-pdo) – A database access abstraction module for PHP applications
	MySQL (php-mysqlnd) – A module for PHP applications that use MySQL databases
	PostgreSQL (php-pgsql) – A PostgreSQL database module for PHP
	MongoDB (php-pecl-mongo) – PHP MongoDB database driver
	SQLite (php-sqlite) – Extension for the SQLite V2 Embeddable SQL Database Engine
	Memcache (php-pecl-memcache) – Extension to work with the Memcached caching daemon
	Memcached (php-pecl-memcached) – Extension to work with the Memcached caching daemon
	GD (php-gd) – A module for PHP applications for using the gd graphics library
	XML (php-xml) – A module for PHP applications which use XML
	MBString (php-mbstring) – A module for PHP applications which need multi-byte string handling
	MCrypt (php-mcrypt) – Standard PHP module provides mcrypt library support

	Taper la commande suivante :

	yum --enablerepo=remi,remi-test install php-pecl-apc php-cli php-pear php-pdo php-mysqlnd php-pgsql php-pecl-mongo php-sqlite php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml


6. Démarrer Nginx

	service nginx start

	ou (deuxieme possibilite)

	/etc/init.d/nginx start 	

7. Démarrer PHP-FPM 

	service php-fpm start 

	ou (deuxieme possibilite)

	/etc/init.d/php-fpm start 

8. Mettre nginx au démarrage

	chkconfig --add nginx
	chkconfig --levels 235 nginx on


9. Mettre php-fpm au démarrage

	chkconfig --add php-fpm
	chkconfig --levels 235 php-fpm on

------------------------------------------------------------------------------------------------------------------------
Configuration de base et test de nginx et php-fpm : création d'un site de test (testsite.local)
------------------------------------------------------------------------------------------------------------------------

10. Créer un répertoire pour le site

	mkdir -p /srv/www/testsite.local/public_html
	mkdir -p /var/log/nginx/testsite.local
	chown -R apache:apache /srv/www/testsite.local
	chown -R nginx:nginx /var/log/nginx

11. Créer et configurer un hôte virtuel pour le site

	mkdir /etc/nginx/sites-available
	mkdir /etc/nginx/sites-enabled

12.  Ajouter au fichier /etc/nginx/nginx.conf après “include /etc/nginx/conf.d/*.conf” line (dans le block http)

	include /etc/nginx/sites-enabled/*;

13. Créer le fichier de l'hôte virtuel /etc/nginx/sites-available/testsite.local  (CONFIGURATION BASIQUE)
    et mettez-y le contenu suivant :
					
		server {
   		 	server_name testsite.local;
   		 	access_log /srv/www/testsite.local/logs/access.log;
    			 error_log /srv/www/testsite.local/logs/error.log;
  		 	root /srv/www/testsite.local/public_html;
 		
  			 location / {
      		 		index index.html index.htm index.php;
   			 }
 
    			 location ~ \.php$ {
       		   	     include /etc/nginx/fastcgi_params;
      				fastcgi_pass  127.0.0.1:9000;
       		 		fastcgi_index index.php;
      				fastcgi_param SCRIPT_FILENAME /srv/www/testsite.local/public_html$fastcgi_script_name;
    			}
		}
	

14. Relier votre hôte virtuel aux sites actifs et redémarrez nginx

	cd /etc/nginx/sites-enabled/
	ln -s /etc/nginx/sites-available/testsite.local
	service nginx restart


15. Ajouter le domaine testsite.local au fichier /etc/hosts , la ligne doit correspondre à ceci

	127.0.0.1               localhost.localdomain localhost testsite.local

16. Tester en créant un fichier index.php dans le dossier /srv/www/testsite.local/public_html/ et ajoutez le contenu:

	<?php 
   		 phpinfo();
	?>

17. Visualisez le résultat sur un navigateur (ex: lynx --> yum install lynx)

        lynx http://testsite.local

------------------------------------------------------------------------------------------------------------------------
Exemple d'isolation des utiliasteurs en différenciant trois user pour trois parties d'un même site
------------------------------------------------------------------------------------------------------------------------

0. Créer un utilisateur example et un groupe du meme nom

1. Faites une copie du fichier /etc/php-fpm.d/www.conf et renommez le pool1.conf

2. Editez le fichier et tout en haut: remplacez [www] par [mysite] par exemple.

3. Ajouter les lignes suivantes  avant la ligne listen = 

user = example
group = example

4. Changer la ligne listen = par la ligne suivante

listen = /var/run/php5-fpm-mysite.sock

5. Redémarrer php-fpm

service php-fpm restart

Vérifier les processus vos deviez voir les 2 pools avec des noms d'utilisateurs séparés 

6. Configurer le .sock au bon endroit dans /etc/nginx/default.conf:

location ~ \.php.*$ {
	fastcgi_split_path_info ^(.+\.php)(/.+)$;

	# NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

	# With php5-cgi alone:
	# fastcgi_pass 127.0.0.1:9000;
	# With php5-fpm:
	fastcgi_pass unix:/var/run/php5-fpm.sock;
	fastcgi_index index.php;
	include fastcgi_params;

	fastcgi_param PATH_INFO $uri;
}

7. Modifier la description du pass dans la ligne correspondante :

fastcgi_pass unix:/var/run/php5-fpm-mysite.sock;

8. Modifier le nombre de worker_process dans le fichier /etc/nginx/nginx.conf

Et voilà.