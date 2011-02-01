Quick Start
============

This page will explain how to get a working version of Canvas LMS running as quickly as possible. Note that you probably don't want to use these instructions
for getting a production-ready system running; for that, please see [[Production Start]].

Prerequisites
-------------

This tutorial is targeting POSIX-based systems (like Mac OS X and Linux). This tutorial was written and tested using Ubuntu's latest LTS 10.04.1, Mac OS X 10.6 (Snow Leopard), and Debian Squeeze 6.0. If you have a different system, consider setting up a server or virtual machine running the latest [Ubuntu LTS](http://www.ubuntu.com/). We'll assume you've either done so or are familiar with these working parts enough to do translations yourself.

A note about Debian - you will have much more luck if you use Debian Squeeze (6.0) or newer.

Getting the code
======

There are two primary ways to get a copy of Canvas

Using Git
-----------

You can install [Git](http://git-scm.com/) on Debian/Ubuntu by running

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

You can also download a tarball or zipfile, thanks to [GitHub](http://github.com/).
  
   * [Canvas Tarball](http://www.instructure.com/code/canvas-stable.tar.gz)
   * [Canvas Zip](http://www.instructure.com/code/canvas-stable.zip)

Application root
--------------

Wherever you check out the code to, we're going to call that your application root. The application root is the folder that has folders such as *app*, *config*, and *script*. For the purposes of this tutorial, we'll assume your application root is */home/user/canvas*.

Dependency Installation
==========

External dependencies
-----------

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there are a few packages you're going to need to install. We recommend that you run:

```
$ sudo apt-get install ruby ruby-dev zlib1g-dev rake rubygems libxml2-dev libmysqlclient-dev libxslt1-dev \
                       libsqlite3-dev libhttpclient-ruby imagemagick
```

For OS X, you'll need to install MySQL.  You have some options, including the old standbys MacPort and Fink, but we recommend [Homebrew](https://github.com/mxcl/homebrew).  For Homebrew, you can get MySQL running with:

```
$ brew install mysql

```

You'll also need to install Ruby 1.8.7+.  We recommend using [RVM](http://rvm.beginrescueend.com/).  Once installed, run:

```
$ rvm install ruby-1.8.7

```

This will install Ruby, RubyGems, and all necessary dependencies (except, of course the gems... which we'll cover next).

Ruby Gems
------------

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that currently operates orthogonally to operating-system package management systems. While we hope to eventually make packages of Canvas for various packaging systems soon, until we do, the Ruby Gems system will be required.

### Upgrading Ruby Gems to 1.3.6 or later

If you ran the above dependency installation command, you just installed Ruby Gems. For Mac users, feel free to skip ahead to "Bundler and Canvas dependencies" below. Unfortunately, Ubuntu 10.04 doesn't provide a recent enough Ruby Gems framework to support Canvas, as Canvas requires the use of Ruby Gems 1.3.6. To find out what version of Ruby Gems you have, you can run

```
$ gem -v
```

You can upgrade Ruby Gems to 1.3.6 or later easily using an Ubuntu PPA (or something similar). For Ubuntu 10.04, we recommend trying [Mackenzie Morgan's Ruby Gem backport PPA](https://launchpad.net/~maco.m/+archive/ruby). If you have the *python-software-properties* package installed (most Ubuntu installations do), you can add this PPA and upgrade Ruby Gems as follows:

```
$ sudo apt-add-repository ppa:maco.m/ruby
$ sudo apt-get update
$ sudo apt-get install rubygems
```

If you cannot use the above Ubuntu PPA, you can also do the following sequence of steps (assuming you have at least one version of Ruby Gems installed):

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

Of course, your *GEM_HOME* environment variable, used this way, will only last the duration of your shell session. If you'd like this environment variable to last between shell sessions, you can add `export GEM_HOME=~/gems` to the bottom of your `~/.bashrc` file.

Bundler and Canvas dependencies
----------

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies. Bundler is great!

Once you have installed Ruby Gems and configured your *GEM_HOME*, **please navigate to the Canvas application root**, where you can install all of the Canvas dependencies using Bundler, like below.

```
~$ cd canvas
~/canvas$ gem install bundler
~/canvas$ $GEM_HOME/bin/bundle install
```


Data setup
========

Canvas default configuration
------

Before we set up all the tables in your database, our Rails code depends on a small few configuration files, which ship with good example settings, so, we'll want to set those up quickly. We'll be examining them more shortly. From the root of your Canvas tree, you can pull in the default configuration values like so:

```
~/canvas$ for config in amazon_s3 delayed_jobs domain file_store outgoing_mail security scribd \
          web_conferences; do cp config/$config.yml.example config/$config.yml; done
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
~/canvas$ rake db:initial_setup
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

Ready, Set, Go!
=============

Now you just need to start the Canvas server! You will need to run the *script/server* daemon:

```
~/canvas$ script/server
```

Open up a browser on the same computer as the one running the server and navigate to [[http://localhost:3000/]] and log in with the user credentials you set up during database configuration. If you don't have a browser running on the same computer, just use the hostname of the computer, and go to http://&lt;hostname&gt;:3000/.

Troubleshooting
==========

We have a full page of frequently asked questions about troubleshooting your Canvas installation. See our [[Troubleshooting]] page.

Production ready configuration
=============

These instructions were meant to give you a taste of Canvas. If you want to actually set up a production ready Canvas instance, please read the [[Production Start]] page.
