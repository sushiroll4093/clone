Quick Start
============

This page will explain how to get a working version of Canvas LMS running as quickly as possible. Note that you probably don't want to use these instructions
for getting a production-ready system running; for that, please see [[Production Start]].

If you need help installing Canvas or troubleshooting your installation, your best bet is to join the community mailing list or the IRC channel [(see the wiki)](https://github.com/instructure/canvas-lms/wiki) and ask specific questions there. It's likely that somebody else has already tackled the same problem.

Prerequisites
-------------

This tutorial is targeting POSIX-based systems (like Mac OS X and Linux). This tutorial was written and tested using Ubuntu's latest LTS 10.04.1, Mac OS X 10.6 (Snow Leopard), and Debian Squeeze 6.0. If you have a different system, consider setting up a server or virtual machine running the latest [Ubuntu LTS](http://www.ubuntu.com/). We'll assume you've either done so or are familiar with these working parts enough to do translations yourself.

A note about Debian - you will have much more luck if you use Debian Squeeze (6.0) or newer.

Getting the code
======

There are two primary ways to get a copy of Canvas

Using Git
-----------

If you don't already have [Git](http://git-scm.com/), you can install it on Debian/Ubuntu by running

```
$ sudo apt-get install git-core
```

Once you have a copy of Git installed on your system, getting the latest source for Canvas is as simple as checking out code from the repo, like so:

```
~$ git clone https://github.com/instructure/canvas-lms.git canvas
~$ cd canvas
~/canvas$ git checkout --track -b stable origin/stable
```

Using a Tarball or a Zip
-----------------

You can also download a tarball or zipfile.
  
   * [Canvas Tarball](http://github.com/instructure/canvas-lms/tarball/stable)
   * [Canvas Zip](http://github.com/instructure/canvas-lms/zipball/stable)

Application root
--------------

Wherever you check out the code to, we're going to call that your application root. The application root is the folder that has folders such as *app*, *config*, and *script*. For the purposes of this tutorial, we'll assume your application root is */home/user/canvas*.

Dependency Installation
==========

Canvas requires Ruby 1.9.3. A minimum version of 1.9.3p286 is recommended. If you are using Debian/Ubuntu, note that apt refers to ruby 1.9.3 as ruby1.9.1 for compatibility reasons.

External dependencies
-----------

### Debian/Ubuntu 

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there are a few packages you're going to need to install. We recommend that you run:

```
$ sudo apt-get install ruby1.9.1 ruby1.9.1-dev zlib1g-dev rubygems1.9.1 libxml2-dev libxslt1-dev \
                       libsqlite3-dev libhttpclient-ruby imagemagick irb1.9.1 \
                       libxmlsec1-dev postgresql python-software-properties
```

Node.js installation:

```
$ sudo add-apt-repository ppa:chris-lea/node.js
$ sudo apt-get update
$ sudo apt-get install nodejs
```

CoffeeScript installation:

```
$ sudo npm install -g coffee-script
```

### Mac OS X

For OS X, you'll need to install the [Command Line Tools for Xcode](http://developer.apple.com/downloads), and make sure you have Ruby 1.9.3. You can find out what version of Ruby your Mac came with by running:

```
$ ruby -v
```

You also need Postgres and the [xmlsec library](http://www.aleksey.com/xmlsec/) installed. The easiest way to get these is via [homebrew](http://mxcl.github.com/homebrew/). Once you have homebrew installed, just run:

```
$ brew install xmlsec1 postgresql
```

Ruby Gems
------------

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that operates orthogonally to operating-system package management systems.

Installing Gems to a user folder
------

We want to configure where Ruby Gems will install packages to. You can do this by setting the *GEM_HOME* environment variable prior to running either Ruby Gems, Bundler (described below), or Canvas.

```
$ mkdir ~/gems
$ export GEM_HOME=~/gems
```

Of course, your *GEM_HOME* environment variable, used this way, will only last the duration of your shell session. If you'd like this environment variable to last between shell sessions, you can add `export GEM_HOME=~/gems` to the bottom of your `~/.bashrc` file on Debian/Ubuntu, or your `~/.bash_login` file on Mac OS X.

Bundler
----------

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies. Bundler is great!

Assuming your *GEM_HOME* is configured, you can install Bundler using Ruby Gems:

```
$ gem install bundler
```

Canvas Dependencies
---------

Once you have installed Bundler, Ruby Gems, configured your *GEM_HOME*, **please navigate to the Canvas application root**, where you can install all of the Canvas dependencies using Bundler.  Since we are using PostgreSQL for this Quick Start and don't want to require you to bother with installing and configuring MySQL, we'll need to tell bundler to ignore it.

```
~$ cd canvas
~/canvas$ $GEM_HOME/bin/bundle install --without mysql
```

If there is a warning about libcurl being missing (Seen on Ubuntu 11.04) run the following, then the above command again.

```
apt-get install libcurl4-gnutls-dev
```

If you are running Ubuntu LTS 12+ you will need to run the following commands:
```
sudo apt-get install make
sudo apt-get install postgresql-server-dev-9.1
```

JavaScript Runtime
------------------

You'll also need a JavaScript runtime to translate our CoffeeScript code to JavaScript and a few other things.   We use Node.js for this. Mac OS X users can download the installer from [node.js](http://nodejs.org). Linux users should already have it from the `apt-get install` step above.

Data setup
========

Canvas default configuration
------

Before we set up all the tables in your database, our Rails code depends on a small few configuration files, which ship with good example settings, so, we'll want to set those up quickly. We'll be examining them more shortly. From the root of your Canvas tree, you can pull in the default configuration values like so:

```
~/canvas$ for config in amazon_s3 delayed_jobs domain file_store outgoing_mail security scribd external_migration; \
          do cp config/$config.yml.example config/$config.yml; done
```

Database configuration
---------

Now we need to set up your database configuration. We have provided a sample file for quickstarts, so you just need to copy it in. You'll also want to create two databases. Depending on your OS (i.e. on Linux), you may need to use a postgres user to create the database, and configure database.yml to use a specific username to connect. See the [[Production Start]] tutorial for details on doing that. On OS X, your local user will have permissions create databases already, so no special configuration is necessary.

```
~/canvas$ cp config/database.yml.example config/database.yml
~/canvas$ createdb canvas_development
~/canvas$ createdb canvas_queue_development
```

Note: When installing postgres with brew, you may have trouble connecting to the database and you may get an error like:

```
bandersons-MacBook-Pro-2:canvas banderson$ createdb canvas_development
createdb: could not connect to database postgres: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/pgsql_socket/.s.PGSQL.5432"?
```

If you get a connection error when creating your databases, run the following and add it to your .bash_profile:

```
export PGHOST=localhost
```

Database population
-----------

Once your database is configured, we need to actually fill the database with tables and initial data. You can do this by running our *rake* migration and initialization tasks from your application's root:

```
~/canvas$ $GEM_HOME/bin/bundle exec rake db:initial_setup
```

Test database configuration
-----------

If you want to test your installation, you'll also need to create a test database:

```
createdb canvas_test
psql -c 'CREATE USER canvas' -d canvas_test
psql -c 'GRANT ALL PRIVILEGES ON DATABASE canvas_test TO canvas' -d canvas_test
psql -c 'GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO canvas' -d canvas_test
psql -c 'GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO canvas' -d canvas_test
RAILS_ENV=test bundle exec rake db:test:reset
```

You can then run the tests:

```
spec spec/
```

File Generation
-----------

Canvas needs to build a number of assets before it will work correctly. You will need to run:

```
~/canvas$ $GEM_HOME/bin/bundle exec rake canvas:compile_assets
```

Performance Tweaks
======

If you plan on doing Canvas development, you'll probably want to skip this step, but if you're just looking to try out Canvas, there's some minor configuration tweaks you can make to give yourself a real performance boost. All you need to do is add three lines to a file in your configuration directory.

```
~/canvas$ echo -n 'config.cache_classes = true
config.action_controller.perform_caching = true
config.action_view.cache_template_loading = true
' > config/environments/development-local.rb
```

Please be aware that the instructions described in the [[Production Start]] tutorial will give you a *much* faster Canvas installation.

A note about emails
=======

Canvas will often attempt to send email. With the Quick Start instructions, email will go straight to the console that `script/server` is running on. If you want to set up email that actually goes to email addresses, please follow the [[Production Start]] instructions.

Ready, Set, Go!
=============

Now you just need to start the Canvas server! You will need to run the *script/server* daemon:

```
~/canvas$ $GEM_HOME/bin/bundle exec script/server
```

Open up a browser on the same computer as the one running the server and navigate to [[http://localhost:3000/]] and log in with the user credentials you set up during database configuration. If you don't have a browser running on the same computer, just use the hostname of the computer, and go to http://&lt;hostname&gt;:3000/.

Logging in For the First Time
===========

Your username and password will be whatever you set it up to be during the `rake db:initial_setup` step above. (You should have seen a prompt on the command line that asked for your email and password.)

A note about long-running jobs
========

Canvas relies heavily on background job processors to perform tasks that take too long to do in-line during a web request. The [[Production Start]] instructions have details of how to set up dedicated job processors for production environments. To start a background job processor, run the following command:

```
~/canvas$ $GEM_HOME/bin/bundle exec script/delayed_job run
```

Troubleshooting
==========

We have a full page of frequently asked questions about troubleshooting your Canvas installation. See our [[Troubleshooting]] page.

Production ready configuration
=============

These instructions were meant to give you a taste of Canvas. If you want to actually set up a production ready Canvas instance, please read the [[Production Start]] page.