# Setting up a Development Environment for EMS
## Prerequisites 
* [Git](https://git-scm.com/downloads)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [Ruby](https://www.ruby-lang.org/en/documentation/installation/)
* [Vagrant](https://www.vagrantup.com/downloads.html)

## Cloning the Github repo
* Make sure you have an SSH key linked to your Github account.
	* If you’re unsure of how to do this, check the [Github Instructions](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

* Create somewhere to store the Repo
	* `mkdir -p ~/code/innoved`
* Clone the Repo
	* `git clone --progress -o origin git@github.com:innoved/VLE2.git -b develop ~/code/innoved/vle2`

## Start and SSH into the VM
* `cd ~/code/innoved/vle2/vagrant`
* `vagrant up`
* `vagrant ssh`

## Setting up the VM
* *Update and upgrade packages*
	* `sudo apt-get update`
	* `sudo apt-get -y upgrade`
		* When asked to configure GRUB select `VBOX_HEADDISK`
	* `sudo apt-get install -y python-software-properties git yasm build-essential pkg-config ntp curl vim software-properties-common`
	
* *Add some PPA’s for PHP Apache and MySQL*
	* `sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php`
	* `sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/apache2`
	* `sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/mysql-5.6`
	* `sudo add-apt-repository ppa:chris-lea/redis-server`
	* `sudo apt-get update`

* 	 *Install PHP*
	* `sudo apt-get install -y php5.6 php5.6-cli dh-make-php php5.6-dev`
	* `sudo apt-get install -y php5.6-mbstring php5.6-mcrypt php5.6-mysql php5.6-xml php5.6-curl php5.6-gd php5.6-gmp php5.6-imap php5.6-tidy php5.6-intl`
	* Verify with `php -v`
```
PHP 5.6.27-1+deb.sury.org~precise+1 (cli)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

* *Configure PHP*
	* Modify `/etc/php/5.6/cli/php.ini`  for
		* date.timezone = "Europe/London"
		* date.default_latitude = 53.5448
		* date.default_longitude = -2.6318
		* memory_limit = 1024M
	
	* Modify `/etc/php/5.6/apache2/php.ini`  for
    		* date.timezone = “Europe/London”
		* date.default_latitude = 53.5448
		* date.default_longitude = -2.6318
		* memory_limit = 1024M
		* post_max_size = 512M
		* upload_max_filesize = 512M
		* expose_php = Off
	* Modify `/etc/php5/cli/conf.d/20-intl.ini` add the following
```
[intl]
intl.default_locale = en_utf8	
```
	* Test with `php -v`
```
PHP 5.6.27-1+deb.sury.org~precise+1 (cli)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```


* *Install and Configure Apache*
	* `sudo apt-get install -y apache2`
	* `sudo apt-get install -y libapache2-mod-php5.6`
	* `sudo a2enmod rewrite`
	* `sudo a2enmod ssl`
	* `sudo a2enmod headers`
	* Test with `apache2 -v`
```
Server version: Apache/2.4.23 (Ubuntu)
Server built:   2016-10-06T00:00:00
```

* *Install and Configure Redis*
	* `sudo apt-get install -y redis-server`
	* `sudo pecl install igbinary`
	* Modify both `php.ini` files 
		1. `/etc/php/5.6/cli/php.ini`
		2. `/etc/php/5.6/apache2/php.ini`
			* Under `Dynamic Extensions` add the following
			* `extension=igbinary.so`
	* `git clone https://github.com/nicolasff/phpredis.git`
	* `cd phpredis && phpize`
	* `./configure --enable-redis-igbinary`
	* `sudo make && sudo make install`
	* `cd ..`
	* Modify both `php.ini` files 
		1. `/etc/php/5.6/cli/php.ini`
		2. `/etc/php/5.6/apache2/php.ini`
			* Under `Dynamic Extensions` add the following
			* `extension=redis.so`
    * Modify `/etc/php/5.6/apache2/php.ini` for
        * `session.save_handler = redis`
        * `session.save_path = "tcp://localhost:6379?weight=1&timeout=5”`

	* Test with `redis-server --version`
```
Redis server v=3.0.7 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 build=49ffd7ce21c94625
```

* *Install ImageMagick*
	* `sudo apt-get install -y imagemagick`
	* Test with `convert --version`
```
Version: ImageMagick 6.6.9-7 2016-06-01 Q16 http://www.imagemagick.org
Copyright: Copyright (C) 1999-2011 ImageMagick Studio LLC
Features: OpenMP
```

* *Install OAuth for PHP*
	* `sudo pecl install OAuth-1.2.3`
	* Modify both `php.ini` files 
		1. `/etc/php/5.6/cli/php.ini`
		2. `/etc/php/5.6/apache2/php.ini`
			* Under `Dynamic Extensions` add the following
			* `extension=oauth.so`
	* `sudo service apache2 restart` 
	* Test with `php -m` and verify that `OAuth` is listed in the loaded extensions

* *Install MySQL*
	*  `dpkg -s mysql-client >/dev/null 2>&1 && sudo apt-get -y remove mysql-client` 
	* `dpkg -s mysql-server >/dev/null 2>&1 && sudo apt-get -y remove mysql-server`
	* `command -v debconf-set-selections || sudo apt-get -y install debconf-utils`
	* `sudo debconf-set-selections <<< "mysql-server-5.6 mysql-server/root_password password reverse"`
	* `sudo debconf-set-selections <<< "mysql-server-5.6 mysql-server/root_password_again password reverse"`
	* `export DEBIAN_FRONTEND=noninteractive`
	* `sudo apt-get -y install mysql-client-5.6 mysql-server-5.6`
	* Test with `mysql --version`
```
mysql  Ver 14.14 Distrib 5.6.32, for debian-linux-gnu (x86_64) using  EditLine wrapper	
```

* *Install Composer*
	* `curl -sS https://getcomposer.org/installer | sudo php — —install-dir=/usr/local/bin —filename=composer`
	* `sudo chown -R $USER $HOME/.composer`
	
* *Set some Environment variables*
	* `ln -s /srv/reversal/ad-hoc ~/ad-hoc`
	* `ln -s /srv/reversal/api/laravel ~/reversal`
	* Edit `~/.bashrc`  and add the following to the bottom of the file
```
export INNOVED_PUBLIC_PATH=/srv/reversal/public
export INNOVED_CONFIG_PATH=/srv/reversal/config
export INNOVED_DATA_PATH=/srv/reversal/data
export INNOVED_CODE_PATH=/srv/reversal/src
export LARAVEL_ENVIRONMENT=development
export DB_USR=root
export DB_PASS=reverse
```

* *Create an environment*
	* Our current Environment are:
		1. EMSLFE
		2. EMSPL
		3. EMSRLT
		4. IRFU
		5. PROCONW
		* /LFE will be used as an example/
	* Repeat the following steps for each Environment you wish to set up. Changing `emslfe` to the Environment you are setting up.
		* Connect to MySQL with `mysql -u root -p` with `reverse` as the password.
		* `CREATE DATABASE innovedvle_emslfe DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;`
		* `exit` out of MySQL terminal.
		* Get a DB dump for `emslfe` from Simon or Mark.
		* Rename the `mysql_dump.sql` file to `emslfe.sql`
		* Place the file in the `ad-hoc` folder.
		* Get a `setpwd.sql` file from someone and place that in the `ad-hoc` folder too.
		* Import the DB dump from live. - `reverse` is the password.
		* `mysql -u root -p innovedvle_emslfe < ~/ad-hoc/emslfe.sql`
		* Reset all user passwords for the database. - `reverse` is the password.
		* `mysql -u root -p innovedvle_emslfe < ~/ad-hoc/setpwd.sql`

* *Install and Configure Laravel*
	* `cd ~/reversal`	
	* `composer install`
	* Getting the following error on completion is fine and can be ignored:
```
> php artisan clear-compiled
Could not open input file: artisan
Script php artisan clear-compiled handling the post-install-cmd event returned with error code 1 
```
	* `cd /srv/reversal/script/bin`
	* `bash chgenv setup`
	* `cd ~/`
	* The `chgenv` command can now be ran from anywhere.
	* Change to an environment with `chgenv <env name>`
	* Create `~/reversal/app/config/development/session.php` with the following content:
```
<?php

return array(

'path' => '/',

        /*
        |--------------------------------------------------------------------------
        | Session Cookie Domain
        |--------------------------------------------------------------------------
        |
        | Here you may change the domain of the cookie used to identify a session
        | in your application. This will determine which domains the cookie is
        | available to in your application. A sensible default has been set.
        |
        */

        'domain' => '.reversal.vagrant.vm',

        /*
        |--------------------------------------------------------------------------
        | HTTPS Only Cookies
        |--------------------------------------------------------------------------
        |
        | By setting this option to true, session cookies will only be sent back
        | to the server if the browser has a HTTPS connection. This will keep
        | the cookie from being sent to you if it can not be done securely.
        |
        */

        'secure' => false,

); 
```

* *Set hosts file values*
	* Edit the VM’s `/etc/hosts` file so that the IPv4 block looks like this:
```
127.0.0.1       localhost
127.0.0.1       reversal.vagrant.vm    laravel.innovedv2api.vm
127.0.1.1       www precise64
```
	* Now `exit` from the vagrant’s ssh session and edit your PC’s `hosts` file.
		* On Linux and OSX `/etc/hosts`
		* On Windows `C:\Windows\system32\drivers\etc\hosts`
		* Add the following line:
			* `192.168.33.40 	reversal.vagrant.vm	larval.innovedv2api.com`

* *Change to an Environment*
	* `vagrant ssh`
	* `chgenv emslfe`
	* `exit`

## Installation finished.
* You should now be able to access the VLE application in your browser by going to `http://reversal.vagrant.vm`
* Username `innovedadmin`
* Password `innoved123`
