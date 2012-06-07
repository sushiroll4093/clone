Production Start
========

These instructions will walk you through how to get a production-ready instance of [Instructure](http://www.instructure.com/)'s open source Canvas LMS running and serving your users. If you just want to play around with Canvas and get the simplest server running possible, please see the [[Quick Start]] page.

If you need help installing Canvas or troubleshooting your installation, your best bet is to join the community mailing list or the IRC channel [(see the wiki)](https://github.com/instructure/canvas-lms/wiki) and ask specific questions there. It's likely that somebody else has already tackled the same problem.

Prerequisites
-------------

We assume you are minimally familiar with website configuration and administration - specifically, [Apache](http://httpd.apache.org/) and/or generic [Ruby on Rails](http://rubyonrails.org/) setups. It also doesn't hurt to have a small working knowledge of [Git](http://git-scm.com/), [MySQL](http://www.mysql.com/), and [Passenger](http://www.modrails.com/). We'll point out in this tutorial places where you may need to go learn about these components.

Secondly, this tutorial is targeting POSIX-based systems (like Mac OS X and Linux). This tutorial was written and tested using Ubuntu's latest LTS 10.04.1. If you have a different system, consider setting up a server or virtual machine running the latest [Ubuntu LTS](http://www.ubuntu.com/). We'll assume you've either done so or are familiar with these working parts enough to do translations yourself.

Finally, Canvas likes RAM. While it will run on smaller configurations, we recommend a server with at least 2GB RAM, especially if everything is being run on one server.

Choose your server configuration
=====

You can choose to run Canvas on one or many servers, hosted by a database. You can install the database on the same server as the one Canvas is hosted from, or you can install it separately. No matter what you choose, you will simply need to ensure that all Canvas instances can communicate with your database server, wherever it is.

Additionally, you will need one Canvas app server to run automated jobs. Again, this can be one of your web servers, or it can be a dedicated node. While there is no downside to running the automated job daemon alongside your webserver, if you plan on having much traffic it is recommended to keep the job traffic and user traffic partitioned onto different nodes for best performance.

For the purposes of this tutorial, we will be referring to the server (possibly one of many) that is running Canvas as *appserver*, whereas we'll be referring to the server running your database as *dbserver*. An *appserver* node can either be hosting the website, be processing automated jobs, or both, depending on whether or not you set up a webserver or the automated jobs daemon.

Database installation and configuration
======

Installing MySQL
---------

Rails, the library Canvas uses, supports many database adapters, but we primarily use two - MySQL and SQLite (for testing). Since this tutorial is for setting up a production environment, we recommend MySQL.

You can run MySQL on the same server you're going to run Canvas on, or not. It really doesn't matter. Just make sure the server you're running Canvas on can talk to the MySQL database.

If MySQL isn't already on the host you are planning on running your database on, if the host is Debian/Ubuntu, then this is as easy as 

```
sysadmin@dbserver:~$ sudo apt-get install mysql-server libmysqlclient-dev
```

N.B., if you're running MacOS X and using the excellent [Homebrew](https://github.com/mxcl/homebrew) tool, then you can just run `brew install mysql`. Note that you need [Xcode](http://developer.apple.com/tools/xcode/) though.

### Running MySQL on a different server

If you are running MySQL on a different server than the server that Canvas will be running on, you'll need to make sure MySQL is listening to connections from foreign clients. You can do this by editing `/etc/mysql/my.cnf` and commenting out the line that specifies a *bind-address* of *127.0.0.1*.

Configuring MySQL
-----------------

You'll want to set up a Canvas user inside of MySQL. To do this, you will need to execute some SQL commands to create the necessary user and databases. Note that in the below commands, you'll want to replace *localhost* with the hostname of the server Canvas is running on, if Canvas is running on a different server than MySQL.

You'll also want to pick a good password to replace *canvas_database_password*. Note that you can also change the name of the databases if you so choose. The configuration file where you tell Canvas about how to connect to this database will need to have the database name changed as well.

```
sysadmin@dbserver:~$ mysql -u root -p
> create database canvas_production;
> create database canvas_queue_production;
> create user 'canvas'@'localhost' identified by 'canvas_database_password';
> grant all privileges on canvas_production.* to 'canvas'@'localhost' with grant option;
> grant all privileges on canvas_queue_production.* to 'canvas'@'localhost' with grant option;
> exit
```

Getting the code
======

There's two primary ways to get a copy of Canvas

Using Git
-----------

You can install [Git](http://git-scm.com/) on Debian/Ubuntu by running

```
sysadmin@appserver:~$ sudo apt-get install git-core
```

Once you have a copy of Git installed on your system, getting the latest source for Canvas is as simple as checking out code from the repo, like so:

```
sysadmin@appserver:~$ git clone https://github.com/instructure/canvas-lms.git canvas
sysadmin@appserver:~$ cd canvas
sysadmin@appserver:~/canvas$ git checkout --track -b stable origin/stable
```

Using a Tarball or a Zip
-----------------

You can also download a tarball or zipfile.
  
   * [Canvas Tarball](http://www.instructure.com/code/canvas-stable.tar.gz)
   * [Canvas Zip](http://www.instructure.com/code/canvas-stable.zip)

Code installation
======

We need to put the Canvas code in the location where it will run from. On a Unix machine, choosing something like the following is a good choice:
```
/var/rails/canvas
```

Take your tarball or your checkout and make sure you move the contents to this directory you've chosen such that all of the directories inside the canvas directory (app, config, db, doc, public, etc). all exist inside this new directory you chose.

We'll be referring to */var/rails/canvas* (or whatever you chose) as your Rails application root.

As an example:

```
sysadmin@appserver:~$ sudo mkdir -p /var/rails/canvas
sysadmin@appserver:~$ sudo chown -R sysadmin /var/rails/canvas
sysadmin@appserver:~$ cd canvas
sysadmin@appserver:~/canvas$ ls
app     db   Gemfile  log     Rakefile  spec  tmp
config  doc  lib      public  script    test  vendor
sysadmin@appserver:~/canvas$ cp -av * /var/rails/canvas
sysadmin@appserver:~/canvas$ cd /var/rails/canvas
sysadmin@appserver:/var/rails/canvas$ ls
app     db   Gemfile  log     Rakefile  spec  tmp
config  doc  lib      public  script    test  vendor
sysadmin@appserver:/var/rails/canvas$
```

Dependency Installation
==========

External dependencies
-----------

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there's a few packages you're going to need to install. We recommend that you run:

```
sysadmin@appserver:~$ sudo apt-get install ruby ruby-dev zlib1g-dev rake rubygems libxml2-dev \
              libmysqlclient-dev libxslt1-dev libsqlite3-dev libhttpclient-ruby nano imagemagick \
              irb  libpq-dev nodejs
```

(note that for OS X, the vanilla Ruby that comes with your Mac should be fine, but you will need [Xcode](http://developer.apple.com/tools/xcode/))

Ruby Gems
------------

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that currently operates orthogonally to operating-system package management systems. While we hope to eventually make packages of Canvas for various packaging systems soon, until we do, the Ruby Gems system will be required.

### Upgrading Ruby Gems to 1.3.6 or later

If you ran the above dependency installation command, you just installed Ruby Gems. Unfortunately, Ubuntu 10.04 doesn't provide a recent enough Ruby Gems framework to support Canvas, as Canvas requires the use of Ruby Gems 1.3.6 or newer. To find out what version of Ruby Gems you have, you can run

```
sysadmin@appserver:~$ gem -v
1.3.7
sysadmin@appserver:~$
```

You can upgrade Ruby Gems to 1.3.6 or newer easily using an Ubuntu PPA (or something similar). For Ubuntu 10.04, we recommend trying [Mackenzie Morgan's Ruby Gem backport PPA](https://launchpad.net/~maco.m/+archive/ruby). If you have the *python-software-properties* package installed (most Ubuntu installations do), you can add this PPA and upgrade Ruby Gems as follows:

```
sysadmin@appserver:~$ sudo apt-add-repository ppa:maco.m/ruby
sysadmin@appserver:~$ sudo apt-get update
sysadmin@appserver:~$ sudo apt-get install rubygems
```

If you cannot use the above Ubuntu PPA, you can also do the following (assuming you have at least one version of Ruby Gems installed):

```
sysadmin@appserver:~$ sudo gem update --system
```

Some very old versions of Ruby Gems require even more steps:

```
sysadmin@appserver:~$ sudo gem install rubygems-update
sysadmin@appserver:~$ sudo /var/lib/gems/1.8/bin/update_rubygems
sysadmin@appserver:~$ sudo gem update --system
```

Installing Gems to a user folder
------

Once you have the latest Ruby Gems, we want to configure where Ruby Gems will install packages to. You can do this by setting the *GEM_HOME* environment variable prior to running either Ruby Gems, Bundler (described below), or Canvas.

```
sysadmin@appserver:~$ mkdir /home/sysadmin/gems
sysadmin@appserver:~$ export GEM_HOME=/home/sysadmin/gems
```

Of course, your *GEM_HOME* environment variable, used this way, will only last the duration of your shell session. We'll explain the necessary places to configure the *GEM_HOME* environment variable below, such as your Apache configuration, automated jobs support, and other things.

Bundler and Canvas dependencies
----------

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies. Bundler is great!

Once you have installed Ruby Gems and configured your *GEM_HOME*, **please navigate to the Canvas application root**, where you can install all of the Canvas dependencies using Bundler, like below.

```
sysadmin@appserver:/var/rails/canvas$ gem install bundler
sysadmin@appserver:/var/rails/canvas$ $GEM_HOME/bin/bundle install
```

JavaScript Runtime
------------------

You'll also need a JavaScript runtime to translate our CoffeeScript code to JavaScript and a few other things.  Mac OS X users already have _JavaScript Core_ and don't need to do anything.  Linux users will probably want NodeJS, which should have been installed in the `apt-get install` step above. Other options can be found at the [execjs homepage](https://github.com/sstephenson/execjs).

Canvas default configuration
------

Before we set up all the tables in your database, our Rails code depends on a small few configuration files, which ship with good example settings, so, we'll want to set those up quickly. We'll be examining them more shortly. From the root of your Canvas tree, you can pull in the default configuration values like so:

```
sysadmin@appserver:/var/rails/canvas$ for config in amazon_s3 database delayed_jobs domain file_store \
                 outgoing_mail security external_migration; do cp config/$config.yml.example config/$config.yml; done
```

Database configuration
-------------

Now we need to set up your database configuration to point to your MySQL server and your production databases. Open the file *config/database.yml*, and find the **production** environment section. You can open this file with an editor like this:

```
sysadmin@appserver:/var/rails/canvas$ nano config/database.yml
```

Update this section to reflect your MySQL server's location and authentication credentials. This is the place you will put the password and database name, along with anything else you set up, from the MySQL setup steps.

Outgoing mail configuration
---------------

For Canvas to work properly, you need an outgoing SMTP mail server. All you need to do is get valid outgoing SMTP settings. Open *config/outgoing_mail.yml*:

```
sysadmin@appserver:/var/rails/canvas$ nano config/outgoing_mail.yml
```

Find the **production** section and configure it to match your SMTP provider's settings. Note that the *domain* and *outgoing_address* fields are not for SMTP, but are for Canvas. *domain* is required, and is the domain name that outgoing emails are expected to come from. *outgoing_address* is optional, and if provided, will show up as the address in the *From* field of emails Canvas sends.

If you don't want to use authentication, simply comment out the lines for *user_name*, *password*, and *authentication*.

URL configuration
---------------------

In many notification emails, and other events that aren't triggered by a web request, Canvas needs to know the URL that it is visible from. For now, these are all constructed based off a domain name. Please edit the **production** section of *config/domain.yml* to be the appropriate domain name for your Canvas installation. For the *domain* field, this will be the part between `http://` and the next `/`. Instructure uses *canvas.instructure.com*.

```
sysadmin@appserver:/var/rails/canvas$ nano config/domain.yml
```

Note that the optional *files_domain* field is required if you plan to host user-uploaded files and wish to be secure. *files_domain* must be a different hostname from the browser's perspective, even though it can be the same Apache server, and even the same IP address.

Database population
-----------

Once your database is configured, we need to actually fill the database with tables and initial data. You can do this by running our *rake* migration and initialization tasks from your application's root:

```
sysadmin@appserver:/var/rails/canvas$ RAILS_ENV=production $GEM_HOME/bin/bundle exec rake db:initial_setup
```

File Generation
-----------

Canvas needs to build a number of assets before it will work correctly. You will need to run:

```
sysadmin@appserver:/var/rails/canvas$ $GEM_HOME/bin/bundle exec rake canvas:compile_assets
```

Canvas ownership
=========

### Making sure Canvas can't write to more things than it should.

Set up or choose a user you want the Canvas Rails application to run as. This can be the same user as your webserver (*www-data* on Debian/Ubuntu), your personal user account, or something else. Once you've chosen or created a new user, you need to change the ownership of key files in your application root to that user, like so

```
sysadmin@appserver:~$ cd /var/rails/canvas
sysadmin@appserver:/var/rails/canvas$ sudo adduser --disabled-password --gecos canvas canvasuser
sysadmin@appserver:/var/rails/canvas$ sudo mkdir -p log tmp/pids public/assets public/stylesheets/compiled
sysadmin@appserver:/var/rails/canvas$ sudo touch Gemfile.lock
sysadmin@appserver:/var/rails/canvas$ sudo chown -R canvasuser config/environment.rb log tmp public/assets \
                                                               public/stylesheets/compiled Gemfile.lock
```

Passenger will choose the user to run the application on based on the ownership settings of *config/environment.rb*. Note that it is probably wise to ensure that the ownership settings of all other files besides the ones with permissions set just above are restrictive, and only allow your *canvasuser* user account to read the rest of the files.

### Making sure other users can't read private Canvas files

There are a number of files in your configuration directory (`/var/rails/canvas/config`) that contain passwords, encryption keys, and other private data that would compromise the security of your Canvas installation if it became public. These are the *.yml* files inside the *config* directory, and we want to make them readable only by the *canvasuser* user.

```
sysadmin@appserver:/var/rails/canvas$ sudo chown canvasuser config/*.yml
sysadmin@appserver:/var/rails/canvas$ sudo chmod 400 config/*.yml
```

Note that once you change these settings, to modify the configuration files henceforth, you will have to use *sudo*.

Apache configuration
=========

Installation
----------

You're now going to need to set up the webserver. We're going to use [Apache](http://httpd.apache.org/) and [Passenger](http://www.modrails.com/) to serve the Canvas content. If you are on Debian/Ubuntu, you can quickly do this by typing

```
sysadmin@appserver:/var/rails/canvas$ sudo apt-get install apache2 libapache2-mod-passenger
```

We'll be using mod_rewrite, so you'll want to enable that.

```
sysadmin@appserver:/var/rails/canvas$ sudo a2enmod rewrite
```

Once you have Apache and Passenger installed, we're going to need to set up Apache, Passenger, and your Rails app to all know about each other. This will be a brief overview, and for more detail, you should check out the [Passenger documentation for setting up Apache](http://www.modrails.com/documentation/Users%20guide%20Apache.html).

Configure Passenger with Apache
-----------

First, make sure Passenger is enabled for your Apache configuration. In Debian/Ubuntu, the *libapache2-mod-passenger* package should have put symlinks inside of */etc/apache2/mods-enabled/* called *passenger.conf* and *passenger.load*. If it didn't or they are disabled somehow, you can enable passenger by running:

```
sysadmin@appserver:/var/rails/canvas$ sudo a2enmod passenger
```

In other setups, you just need to make sure you add the following lines to your Apache configuration, changing paths to appropriate values if necessary:

```
LoadModule passenger_module /usr/lib/apache2/modules/mod_passenger.so
PassengerRoot /usr
PassengerRuby /usr/bin/ruby
```

Configure SSL with Apache
-------------

Next, we need to make sure your Apache configuration supports SSL. Debian/Ubuntu doesn't ship Apache with the SSL module enabled by default, so you will need to create the appropriate symlinks to enable it.

```
sysadmin@appserver:/var/rails/canvas$ sudo a2enmod ssl
```

On other systems, you need to make sure something like below is in your config:

```
LoadModule ssl_module /usr/lib/apache2/modules/mod_ssl.so
SSLRandomSeed startup builtin
SSLRandomSeed startup file:/dev/urandom 512
SSLRandomSeed connect builtin
SSLRandomSeed connect file:/dev/urandom 512
SSLSessionCache        shmcb:/var/run/apache2/ssl_scache(512000)
SSLSessionCacheTimeout  300
SSLMutex  file:/var/run/apache2/ssl_mutex
SSLCipherSuite HIGH:MEDIUM:!ADH
SSLProtocol all -SSLv2
```

Configure Canvas with Apache
---------------

Now we need to tell Passenger about your particular Rails application. First, disable any Apache VirtualHosts you don't want running. On Debian/Ubuntu, you can simply unlink any of the symlinks in the */etc/apache2/sites-enabled* subdirectory you aren't interested in. In other set-ups, you can remove or comment out VirtualHosts you don't want. 

```
sysadmin@appserver:/var/rails/canvas$ cd /etc/apache2/sites-enabled
sysadmin@appserver:/etc/apache2/sites-enabled$ ls
000-default
sysadmin@appserver:/etc/apache2/sites-enabled$ sudo unlink 000-default
```

Now, we need to make a VirtualHost for your app. On Debian/Ubuntu, we are going to need to make a new file called */etc/apache2/sites-available/canvas*. On other setups, find where you put VirtualHosts definitions. You can open this file like so:

```
sysadmin@appserver:/etc/apache2/sites-enabled$ sudo nano /etc/apache2/sites-available/canvas
```

In the new file, or new spot, depending, you want to place the following snippet. **You will want to modify** the lines designated *ServerName*(2), *ServerAdmin*(2), *DocumentRoot*(2), *SetEnv*(2), *Directory*(2), and probably *SSLCertificateFile*(1) and *SSLCertificateKeyFile*(1), discussed below in the "Note about SSL Certificates".

```
<VirtualHost *:80>
  ServerName canvas.example.com
  ServerAlias files.canvas.example.com
  ServerAdmin youremail@example.com
  DocumentRoot /var/rails/canvas/public
  RewriteEngine On
  RewriteCond %{HTTP:X-Forwarded-Proto} !=https
  RewriteCond %{REQUEST_URI} !^/health_check
  RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [L]  
  ErrorLog /var/log/apache2/canvas_errors.log
  LogLevel warn
  CustomLog /var/log/apache2/canvas_access.log combined
  SetEnv GEM_HOME /home/sysadmin/gems
  SetEnv RAILS_ENV production
  <Directory /var/rails/canvas/public>
    Allow from all
    Options -MultiViews
  </Directory>
</VirtualHost>
<VirtualHost *:443>
  ServerName canvas.example.com
  ServerAlias files.canvas.example.com
  ServerAdmin youremail@example.com
  DocumentRoot /var/rails/canvas/public
  ErrorLog /var/log/apache2/canvas_errors.log
  LogLevel warn
  CustomLog /var/log/apache2/canvas_ssl_access.log combined
  SSLEngine on
  BrowserMatch "MSIE [2-6]" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0
  BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
  # the following ssl certificate files are generated for you from the ssl-cert package.
  SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
  SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
  SetEnv GEM_HOME /home/sysadmin/gems
  SetEnv RAILS_ENV production
  <Directory /var/rails/canvas/public>
    Allow from all
    Options -MultiViews
  </Directory>
</VirtualHost>
```

And finally, if you created this as its own file inside */etc/apache2/sites-available*, we'll need to make it an enabled site.

```
sysadmin@appserver:/etc/apache2/sites-enabled$ sudo a2ensite canvas
```

A Note about SSL Certificates
----------

You'll notice in the above Canvas configuration file that we provided directives to an *SSLCertificateFile* and an *SSLCertificateKeyFile*. The files specified are **self-signed** certificates that come with your operating system.

Browsers, by default, are configured not to accept self-signed certificates without complaining. The reason for this is because otherwise a server using a self-signed certificate can risk what's called a man-in-the-middle attack.

If you want to get a certificate for your Canvas installation that will be accepted automatically by your user's browsers, you will need to contact a *certificate authority* and generate one. For the sake of example, Verisign is a commonly used certificate authority.

For more information on setting up Apache with SSL, please see [O'Reilly OnLamp.com's instructions](http://onlamp.com/pub/a/onlamp/2008/03/04/step-by-step-configuring-ssl-under-apache.html), [Apache's official SSL documentation](http://httpd.apache.org/docs/2.0/ssl/), or any one of [many certificate authority's websites](http://www.dmoz.org/Computers/Security/Public_Key_Infrastructure/PKIX/Tools_and_Services/Third_Party_Certificate_Authorities/).

Cache configuration
========

Canvas supports two different methods of caching: Memcache and redis. Below are instructions for setting up redis.

Note: With the introduction of OAuth2 into Canvas, setting up redis is a requirement. Delegated authentication through OAuth2 will not function without it.

Redis
----

Required version: redis 2.2.x.

If you're using Homebrew on Mac OS X, you can install redis by running the command: `brew install redis`.

For Ubuntu, you can use the redis-server package.  However, on lucid, it's not new enough, so Instructure has provided at backport PPA to provide it: https://launchpad.net/~instructure/+archive/backports.

After installing redis, start the server. There are multiple options for doing this. You can set it up so it runs automatically when the server boots, or you can run it manually.

To run it manually from a Homebrew installation, run the command: `redis-server /usr/local/etc/redis.conf`.

Now we need to go back to your canvas-lms directory and edit a configuration file. Inside the config folder, we're going to copy [cache_store.yml.example](https://github.com/instructure/canvas-lms/blob/stable/config/cache_store.yml.example) and edit it:
```
sysadmin@appserver:/var/rails/canvas$ cp config/cache_store.yml.example config/cache_store.yml
sysadmin@appserver:/var/rails/canvas$ nano config/cache_store.yml
```

The file starts with all caching methods commented out. Uncomment the redis portion of the config file and update it to appropriately point to the server that is running redis. In our example, redis is running on the same server as Canvas.

```
# if this file doesn't exist, memcache will be used if there are any
# servers configured in config/memcache.yml
development:
  cache_store: mem_cache_store
  # if no servers are specified, we'll look in config/memcache.yml
  # servers:
  # - localhost
  #
  cache_store: redis_store
  # if no servers are specified, we'll look in config/redis.yml
  servers:
  - localhost
  database: 0
```

Save the file and restart Canvas.

Automated jobs
========

Canvas has some automated jobs that need to run at occasional intervals, such as email reports, statistics gathering, and a few other things. Your Canvas installation will not function properly without support for automated jobs, so we'll need to set that up as well.

Canvas comes with a daemon process that will monitor and manage any automated jobs that need to happen. If your application root is */var/rails/canvas*, this daemon process manager can be found at */var/rails/canvas/script/canvas_init*. 

**You'll need to run these job daemons on at least one server.** Canvas supports running the background jobs on multiple servers for capacity/redundancy, as well.

Because Canvas has so many jobs to run, it is advisable to dedicate one of your app servers to be just a job server. You can do this by simply skipping the Apache steps on one of your app servers, and then only on that server follow these automated jobs setup instructions.

Gem location
-----

To get automated jobs to work, you also need to tell the automated jobs start/stop daemon about your *GEM_HOME*. You can do this by creating a *GEM_HOME* file in your config directory like so (assuming you did `export GEM_HOME=...` earlier):

```
sysadmin@appserver:~$ cd /var/rails/canvas
sysadmin@appserver:/var/rails/canvas$ echo $GEM_HOME | sudo tee config/GEM_HOME
/home/sysadmin/gems
sysadmin@appserver:/var/rails/canvas$
```

Installation
----

If you're on Debian/Ubuntu, you can install this daemon process very easily, first by making a symlink from */var/rails/canvas/script/canvas_init* to */etc/init.d/canvas_init*, and then by configuring this script to run at valid runlevels (we'll be making an *upstart* script soon):

```
sysadmin@appserver:/var/rails/canvas$ sudo ln -s /var/rails/canvas/script/canvas_init /etc/init.d/canvas_init
sysadmin@appserver:/var/rails/canvas$ sudo update-rc.d canvas_init defaults
sysadmin@appserver:/var/rails/canvas$ sudo /etc/init.d/canvas_init start
```

Ready, set, go!
========

Restart Apache (`sudo /etc/init.d/apache2 restart`), and point your browser to your new Canvas installation! Log in with the administrator credentials you set up during database configuration, and you should be ready to use Canvas.

Troubleshooting
==========

We have a full page of frequently asked questions about troubleshooting your Canvas installation. See our [[Troubleshooting]] page.

Common configuration options
========

There are many other aspects of Canvas that you can now configure, having a working production environment. Please see [[Canvas Integration]] for more information.