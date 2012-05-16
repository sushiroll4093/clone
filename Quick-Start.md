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

External dependencies
-----------

### Debian/Ubuntu 

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there are a few packages you're going to need to install. We recommend that you run:

```
$ sudo apt-get install ruby ruby-dev zlib1g-dev rake rubygems libxml2-dev libxslt1-dev \
                       libsqlite3-dev libhttpclient-ruby imagemagick irb nodejs
```

### Mac OS X

For OS X, you'll need to install [Xcode](http://developer.apple.com/tools/xcode/), and make sure you have Ruby 1.8.7 or newer. You can find out what version of Ruby your Mac came with by running:

```
$ ruby -v
```

Ruby Gems
------------

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that currently operates orthogonally to operating-system package management systems. While we hope to eventually make packages of Canvas for various packaging systems soon, until we do, the Ruby Gems system will be required.

### Upgrading Ruby Gems to 1.3.6 or later

If you ran the above Ubuntu/Debian dependency installation command or are running OS X, you either have or just installed Ruby Gems. Unfortunately, many operating systems don't provide a recent enough Ruby Gems framework to support Canvas, as Canvas requires the use of Ruby Gems 1.3.6 or newer. To find out what version of Ruby Gems you have, you can run

```
$ gem -v
```

If you run Ubuntu, you can upgrade Ruby Gems to 1.3.6 or later easily using an Ubuntu PPA (or something similar). For Ubuntu 10.04, we recommend trying [Mackenzie Morgan's Ruby Gem backport PPA](https://launchpad.net/~maco.m/+archive/ruby). If you have the *python-software-properties* package installed (most Ubuntu installations do), you can add this PPA and upgrade Ruby Gems as follows:

```
$ sudo apt-add-repository ppa:maco.m/ruby
```

If you have trouble adding the PPA or you are behind a Corporate firewall, please see [this trick](http://rockycode.com/blog/using-ubuntu-ppa-repositories-behind-firewall/) for enabling apt on port 80.  Once you have successfully added the PPA, update and install rubygems:

```
$ sudo apt-get update
$ sudo apt-get install rubygems
```

If you are running OS X or otherwise cannot use the above Ubuntu PPA, you can also try the following (assuming you at least have some version of Ruby Gems installed):

```
$ sudo gem update --system
```

It is possible that even the above command won't work. If it ran, and `gem -v` shows 1.3.6 or newer, you're fine. However, some very old versions of Ruby Gems will require even more work first, like so:

```
$ sudo gem install rubygems-update
$ sudo /var/lib/gems/1.8/bin/update_rubygems
$ sudo gem update --system
```

Installing Gems to a user folder
------

Once you have the latest Ruby Gems, we want to configure where Ruby Gems will install packages to. You can do this by setting the *GEM_HOME* environment variable prior to running either Ruby Gems, Bundler (described below), or Canvas.

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

Once you have installed Bundler, Ruby Gems, configured your *GEM_HOME*, **please navigate to the Canvas application root**, where you can install all of the Canvas dependencies using Bundler.  Since we are using SQLite for this Quick Start and don't want to require you to bother with installing and configuring MySQL or Postgres (for that, please see [[Production Start]]), we'll need to tell bundler to ignore them.

```
~$ cd canvas
~/canvas$ $GEM_HOME/bin/bundle install --without postgres mysql
```

JavaScript Runtime
------------------

You'll also need a JavaScript runtime to translate our CoffeeScript code to JavaScript and a few other things.  Mac OS X users already have _JavaScript Core_ and don't need to do anything.  Linux users will probably want NodeJS, which should have been installed in the `apt-get install` step above. Other options can be found at the [execjs homepage](https://github.com/sstephenson/execjs).

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

Now we need to set up your database configuration. We have provided a sample file for quickstarts, so you just need to copy it in.

```
~/canvas$ cp config/database.yml.sqlite-example config/database.yml
```

Database population
-----------

Once your database is configured, we need to actually fill the database with tables and initial data. You can do this by running our *rake* migration and initialization tasks from your application's root:

```
~/canvas$ $GEM_HOME/bin/bundle exec rake db:initial_setup
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
~/canvas$ script/server
```

Open up a browser on the same computer as the one running the server and navigate to [[http://localhost:3000/]] and log in with the user credentials you set up during database configuration. If you don't have a browser running on the same computer, just use the hostname of the computer, and go to http://&lt;hostname&gt;:3000/.

A note about long-running jobs
========

Canvas relies heavily on background job processors to perform tasks that take too long to do in-line during a web request. The [[Production Start]] instructions have details of how to set up dedicated job processors for production environments. For this Quick Start, just starting `script/server` as outlined above will by default run one job processor as well.

Troubleshooting
==========

We have a full page of frequently asked questions about troubleshooting your Canvas installation. See our [[Troubleshooting]] page.

Production ready configuration
=============

These instructions were meant to give you a taste of Canvas. If you want to actually set up a production ready Canvas instance, please read the [[Production Start]] page.