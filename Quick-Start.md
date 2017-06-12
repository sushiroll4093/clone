Quick Start
============

READ ME FIRST
----------------

This guide is intended for those who want to get a version of Canvas LMS running as quickly as possible (e.g. for a development environment). The environment produced by this guide is lacking in several features (e.g. [emails are not sent out](#a-note-about-emails), [delayed jobs are not daemonized](#a-note-about-long-running-jobs), [there is no proper application server provided](#ready-set-go)) **and is not suitable for production use**. 

**See the [[Production Start]] guide for instructions on standing up a production-ready system.**

If you need help installing Canvas or troubleshooting your installation, your best bet is to join the community mailing list or the IRC channel [(see the wiki)](https://github.com/instructure/canvas-lms/wiki) and ask specific questions there. It's likely that somebody else has already tackled the same problem. Note that a common category of questions are those that stem from following this guide instead of the [[Production Start]] guide. If you are **sure** that you want to continue with the Quick Start guide, read on; otherwise, you may want to consider jumping into the [[Production Start]] guide instead.

Automated Script
----------------

If you are running Mac OS X, Arch Linux, Fedora, or Ubuntu Linux, there is [an automated script that will set up your environment](https://github.com/FreedomBen/canvas-development-tools/blob/master/CODES.sh).  You may wish to go through the manual setup way at least once though to understand what is happening, but if you need a quick and (mostly) painless setup the script is there for you.

1.  Download the script:  `curl -O https://raw.githubusercontent.com/FreedomBen/canvas-development-tools/master/CODES.sh`
2.  Make the script executable:  `chmod +x CODES.sh`
3.  Run the script (no args will print a help message):  `./CODES.sh --full`

**Note**: when using Ubuntu 14 LTS server, the `CODES.sh` script doesn't seem to do the same thing that the instructions below indicate for installing nodejs.  In addition, installing ruby2.4.1 through the script fails, whereas it succeeds with the steps below.  Taking care of installing these dependencies first, then running the script, seems to work.  So the complete process for the quick start on Ubuntu 14 LTS server (tested in May 2016 on a VirtualBox instance as well as Amazon EC2) is as follows:

    # do this FIRST, before running CODES.sh; the script does something else entirely
    curl -sL https://deb.nodesource.com/setup_6.x | sudo bash -
    sudo apt-get install nodejs
    
    # next, add the ppa:brightbox/ruby-ng and install Ruby 2.4.1
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.4.1 ruby2.4-dev
    
    # Canvas using Yarn, install it with :
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    sudo apt-get update && sudo apt-get install yarn
    
    # NOW, run CODES.sh
    ./CODES.sh --full
    
    # after the setup completes, cd to the canvas root directory, and point your browser to (server):3000
    cd ~/canvas-lms
    bundle exec rails server
    

Prerequisites
-------------

This tutorial is targeting POSIX-based systems (like Mac OS X and Linux). This tutorial was written and tested using Ubuntu's latest LTS 14.04.2, Mac OS X 10.10 (Yosemite), and Debian Jessie 8.1. If you have a different system, consider setting up a server or virtual machine running the latest [Ubuntu LTS](http://www.ubuntu.com/). We'll assume you've either done so or are familiar with these working parts enough to do translations yourself.

Getting the code
======

There are two primary ways to get a copy of Canvas

Using Git
-----------

If you don't already have [Git](http://git-scm.com/), you can install it on Debian/Ubuntu by running

```
$ sudo apt-get install git
```

Once you have a copy of Git installed on your system, getting the latest source for Canvas is as simple as checking out code from the repo, like so:

```
~$ git clone https://github.com/instructure/canvas-lms.git canvas
~$ cd canvas
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

Canvas requires Ruby 2.1. A minimum version of 2.1.6 is recommended.

External dependencies
-----------

### Debian/Ubuntu 

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there are a few packages you're going to need to install. If you're running Ubuntu 14.04 Trusty, you'll need to add a PPA in order to get Ruby 2.1:

```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:brightbox/ruby-ng
$ sudo apt-get update
```

```
$ sudo apt-get install ruby2.1 ruby2.1-dev zlib1g-dev libxml2-dev \
                       libsqlite3-dev postgresql-9.3 libpq-dev \
                       libxmlsec1-dev curl make g++
```
> Note: In Ubuntu, in case you encounter any error such as E: Package 'postgresql-9.3' has no installation candidate, it may be because postgresql-9.3 is not available in that Ubuntu version. The following can be helpful : http://technobytz.com/install-postgresql-9-3-ubuntu.html , https://www.postgresql.org/download/linux/ubuntu/

Node.js installation:
```
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs build-essential
```

After installing Postgres, you will need to set your system username as a postgres superuser.  You can do so by running the following commands:

```
sudo -u postgres createuser $USER
sudo -u postgres psql -c "alter user $USER with superuser" postgres
```

### Mac OS X

For OS X, you'll need to install the [Command Line Tools for Xcode](http://developer.apple.com/downloads), and make sure you have Ruby 2.4.1. You can find out what version of Ruby your Mac came with by running:

```
$ ruby -v
```

You also need Postgres and the [xmlsec library](http://www.aleksey.com/xmlsec/) installed. The easiest way to get these is via [homebrew](http://brew.sh/). Once you have homebrew installed, just run:

```
$ brew install postgresql-9.5 nodejs xmlsec1
```

Ruby Gems
------------

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that operates orthogonally to operating-system package management systems.

Installing Gems to a user folder
------

We want to configure where Ruby Gems will install packages to. You can do this by setting the *GEM_HOME* environment variable prior to running either Ruby Gems, Bundler (described below), or Canvas.

```
$ mkdir ~/.gems
$ export GEM_HOME=~/.gems
```

Of course, your *GEM_HOME* environment variable, used this way, will only last the duration of your shell session. If you'd like this environment variable to last between shell sessions, you can add `export GEM_HOME=~/.gems` to the bottom of your `~/.bashrc` file on Debian/Ubuntu, or your `~/.bash_login` file on Mac OS X.

Bundler
----------

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies. Bundler is great!

Assuming your *GEM_HOME* is configured, you can install Bundler using Ruby Gems:

```
$ gem install bundler
```

On Debian Jessie, you'll need to substitute `gem` with `gem2.1`.

Canvas Dependencies
---------

Once you have installed Bundler, Ruby Gems, configured your *GEM_HOME*, **please navigate to the Canvas application root**, where you can install all of the Canvas dependencies using Bundler. 

```
~$ cd canvas
~/canvas$ bundle install
~/canvas$ npm install
```

If you're on OS X Mavericks or Yosemite and hit an error with the thrift gem, you might have to set the following bundler flag and then run bundle install again (see https://issues.apache.org/jira/browse/THRIFT-2219):

```
~/canvas$ $GEM_HOME/bin/bundle config build.thrift --with-cppflags='-D_FORTIFY_SOURCE=0'
```

If you are on El Capitan and encounter an error with the thrift gem that has something like the following in the text.
```
compact_protocol.c:431:41: error: shifting a negative signed value is undefined [-Werror,-Wshift-negative-value]
```

Use the following command to install the gem and then continue with another bundle install.

```
gem install thrift -v 0.8.0 -- --with-cppflags=\"-D_FORTIFY_SOURCE=0 -Wno-shift-negative-value\"
```

The problem is the clang got updated in El Capitan and some of the flags cause problems now on install for the old version of thrift.

If you hit an error with the eventmachine gem, you might have to set the following bundler flag and then run bundle install again:

```
~/canvas$ $GEM_HOME/bin/bundle config build.eventmachine --with-cppflags=-I/usr/local/opt/openssl/include
```

JavaScript Runtime
------------------

You'll also need a JavaScript runtime to translate our CoffeeScript code to JavaScript and a few other things.  We use Node.js for this. Mac OS X users can download the installer from [node.js](http://nodejs.org). Linux users should already have it from the `apt-get install` step above.

CoffeeScript can be installed the same way as on other platforms, through `npm` (which is included with the nodeJS installation):
```
$ sudo npm install -g coffee-script@1.6.2
```

Data setup
========

Canvas default configuration
------

Before we set up all the tables in your database, our Rails code depends on a small few configuration files, which ship with good example settings, so, we'll want to set those up quickly. We'll be examining them more shortly. From the root of your Canvas tree, you can pull in the default configuration values like so:

```
~/canvas$ for config in amazon_s3 delayed_jobs domain file_store outgoing_mail security external_migration; \
          do cp -v config/$config.yml.example config/$config.yml; done
```

Database configuration
---------

Now we need to set up your database configuration. We have provided a sample file for quickstarts, so you just need to copy it in. You'll also want to create two databases. Depending on your OS (i.e. on Linux), you may need to use a postgres user to create the database, and configure database.yml to use a specific username to connect. See the [[Production Start]] tutorial for details on doing that. On OS X, your local user will have permissions to create databases already, so no special configuration is necessary.

```
~/canvas$ cp config/database.yml.example config/database.yml
~/canvas$ createdb canvas_development
```

Note: When installing postgres with brew, you may have trouble connecting to the database and you may get an error like:

```
~/canvas$ createdb canvas_development
createdb: could not connect to database postgres: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/pgsql_socket/.s.PGSQL.5432"?
```

If you get a connection error when creating your databases, run the following and add it to your .bash_profile:

```
export PGHOST=localhost
```

If, after that, you get another error like:

```
~/canvas$ createdb canvas_development
createdb: could not connect to database template1: could not connect to server: Connection refused
    Is the server running on host "localhost" (::1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (127.0.0.1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (fe80::1) and accepting
    TCP/IP connections on port 5432?
```

then postgres may not be running.  To start it:

```
$ initdb /usr/local/var/postgres -E utf8
$ pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
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
psql -c 'CREATE USER canvas' -d postgres
psql -c 'ALTER USER canvas CREATEDB' -d postgres
createdb -U canvas canvas_test
psql -c 'GRANT ALL PRIVILEGES ON DATABASE canvas_test TO canvas' -d canvas_test
psql -c 'GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO canvas' -d canvas_test
psql -c 'GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO canvas' -d canvas_test
RAILS_ENV=test $GEM_HOME/bin/bundle exec rake db:test:reset
```

Make sure you can run a spec file (the full suite takes too long to run locally):

```
bundle exec rspec spec/models/assignment_spec.rb
```

If you get a warning saying that you have the wrong version of Bundle installed, try this:
```
$GEM_HOME/bin/bundle exec rspec spec/models/assignment_spec.rb
```

File Generation
-----------

Canvas needs to build a number of assets before it will work correctly. You will need to run:

```
~/canvas$ $GEM_HOME/bin/bundle exec rake canvas:compile_assets
```

Note that we've seen trouble with npm trying to hold too many files open at once.  If you see an error with `libuv` while running npm, try increasing your `ulimit`.  To do this in OS X add `ulimit -n 4096` to your `~/.bash_profile` or `~/.zsh_profile`.

Performance Tweaks
======

Installing redis will significantly improve your Canvas performance. For detailed instructions, see [[Production Start#redis]]. On OS X, use the following:

```
brew install redis
redis-server /usr/local/etc/redis.conf
echo -e "development:\n  cache_store: redis_store" > config/cache_store.yml
echo -e "development:\n  servers:\n  - redis://localhost" > config/redis.yml
```

If you're just looking to try out Canvas, there's some minor configuration tweaks you can make to give yourself a real performance boost. All you need to do is add three lines to a file in your configuration directory. (If you plan on doing Canvas development, you may want to skip this step or only enable class caching, as these settings will require you to restart your server each time you change Ruby or ERB files.)

```
~/canvas$ echo -n 'config.cache_classes = true
config.action_controller.perform_caching = true
config.action_view.cache_template_loading = true
' > config/environments/development-local.rb
```

Please be aware that the instructions described in the [[Production Start]] tutorial will give you a *much* faster Canvas installation.

A note about emails
=======

Canvas will often attempt to send email. With the Quick Start instructions, email will go straight to the console that `rails server` is running on. If you want to set up email that actually goes to email addresses, please follow the [[Production Start]] instructions.

Ready, Set, Go!
=============

Now you just need to start the Canvas server! You will need to run the *rails server* daemon:

```
~/canvas$ $GEM_HOME/bin/bundle exec rails server
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