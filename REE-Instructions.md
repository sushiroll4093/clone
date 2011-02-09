Upgrading from standard Ruby to Ruby Enterprise Edition
================

These instructions assume you followed the [Production Start](https://github.com/instructure/canvas-lms/wiki/Production-Start) instructions for installing Canvas already.

Download Ruby Enterprise Edition
======

Download the latest version from [Phusion's website](http://www.rubyenterpriseedition.com/download.html). If you used Ubuntu 10.04 in the previous tutorial, you can take a shortcut and download Phusion's Ubuntu 10.04 deb packages from a tab on that page.

Install build dependencies
========

Because you will be rebuilding Passenger, you will need a few dependencies. Though the installer may tell you you need more, you should at least install the following packages:

```
sysadmin@appserver:~$ sudo apt-get install build-essential apache2-prefork-dev libapr1-dev libaprutil1-dev
```

Install Ruby Enterprise Edition and new Passenger
======

If you downloaded the deb package, installation takes two steps:

```
sysadmin@appserver:~$ sudo dpkg -i ruby-enterprise_1.8.7-....deb
sysadmin@appserver:~$ sudo /usr/local/bin/passenger-install-apache2-module
```

Otherwise, follow the installation instructions your Ruby Enterprise Edition download installer prompts you with. You will need to find the *passenger-install-apache2-module* script's location and run it as well. The install script should mention its path to you.

Either way, the Apache2 module installer will give you three lines to add to your Apache config. Keep track of these, as we will need them in a minute. They look like:

```
LoadModule passenger_module /...
PassengerRoot /...
PassengerRuby /...
```

Fix Apache
==========

Apache is already using a version of Passenger we installed through Ubuntu. You will need to disable that Apache module and enable the new one Phusion's installer just built for you.

First, disable the existing one:

```
sysadmin@appserver:~$ sudo a2dismod passenger
```

Now, we create a file that contains the three lines the installer gave you earlier.

```
sysadmin@appserver:~$ sudo nano /etc/apache2/mods-available/ree-passenger.load
```

Put the three lines in that configure your new Passenger instance. Then enable the new Passenger.

```
sysadmin@appserver:~$ sudo a2enmod ree-passenger
```

Install Gems for Ruby Enterprise Edition
=============

Ruby Enterprise Edition does not look in the standard Ruby places for Gems, but instead looks in its own directories. Because of this, we will need to reinstall the Gems Canvas requires for Ruby Enterprise Edition.

Note, you may have to replace `/usr/local/bin` below with `/opt/ruby-enterprise-1.8.7-*/bin` or wherever you installed REE to.

```
sysadmin@appserver:~$ export GEM_HOME=
sysadmin@appserver:~$ cd /var/rails/canvas
sysadmin@appserver:/var/rails/canvas$ sudo /usr/local/bin/gem install bundler
sysadmin@appserver:/var/rails/canvas$ sudo /usr/local/bin/bundle install
```

Now, we need to comment out the lines that set your old GEM_HOME in your Apache Canvas site config. Find both lines that start with `SetEnv GEM_HOME` and put a `#` before them.

```
sysadmin@appserver:~$ sudo nano /etc/apache2/sites-available/canvas
```

Fix your Automated Jobs daemon
=========

Make REE the default Ruby
--------------

If you installed Phusion Ruby Enterprise Edition via the Ubuntu deb package, then it should have just become the default Ruby interpreter (assuming your `PATH` puts `/usr/local/bin` before `/usr/bin`, as usual).

On the other hand, if you installed REE to a different location (such as `/opt`), you will need to make REE be the default Ruby version for your system. Adding a symlink in `/usr/local/bin` should be sufficient (again, assuming your PATH by default prefers `/usr/local/bin`):

```
sysadmin@appserver:~$ sudo ln -s /opt/ruby-enterprise-1.8.7-*/bin/ruby /usr/local/bin/ruby
```

After symlinking, you may need to log out and log back in before your shell notices the new Ruby.

You can check if REE is your default Ruby interpreter by running:

```
sysadmin@appserver:~$ ruby -v
ruby 1.8.7 (2010-04-19 patchlevel 253) [i686-linux], MBARI 0x8770, Ruby Enterprise Edition 2010.02
```

Remove old automated jobs GEM_HOME configuration
--------------

Since REE is now your default Ruby, your automated jobs daemon will now start running under Ruby Enterprise Edition as well.

To get your automated jobs daemon to use your new Ruby Enterprise Edition Gems, we need to disable any *GEM_HOME* configuration you did previously. 

Make sure you have removed the `/var/rails/canvas/config/GEM_HOME` file if you created one, and make sure GEM_HOME is no longer getting set in `/var/rails/canvas/script/canvas_init`.

```
sysadmin@appserver:~$ sudo rm -f /var/rails/canvas/config/GEM_HOME
sysadmin@appserver:~$ sudo nano /var/rails/canvas/script/canvas_init
```

Remove your old Gems (optional)
======

Though this step isn't necessary, you can ensure that your old Ruby configuration is no longer being used by removing the Gems at your old GEM_HOME location.

```
sysadmin@appserver:~$ sudo rm -rf /home/sysadmin/gems/*
```

Restart Apache
=========

Your reconfiguration should now be complete. Restart Apache and your new Ruby Enterprise Edition with corresponding Passenger build should be running.

```
sysadmin@appserver:~$ sudo /etc/init.d/apache2 restart
```

Restart Automated Jobs
==========

If this node is running automated jobs, you can restart them as well.

```
sysadmin@appserver:~$ sudo /etc/init.d/canvas_init stop
sysadmin@appserver:~$ sudo /etc/init.d/canvas_init start
```
