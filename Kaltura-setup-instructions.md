**Users are encouraged to try the official [Kaltura Integration for Canvas](http://corp.kaltura.com/Products/Video-Applications/Canvas-Video-App)**

_These instructions provided graciously by John Desha, with wiki formatting edits._

For installing Kaltura CE on Ubuntu LTS

Install ubuntu LTS core only.
I set up a server with 2 GB ram, 2 cpu. You'll probably want a large-ish amount of hard drive space. This server will be hosting all your videos. Further, with debug logging on, Kaltura can generate ~ 800 MB a day of log data.

## Prereq install

Install the following packages:

```
sudo apt-get install apache2 php5 php5-cli mysql-server mysql-client \
  curl memcached php5-curl php5-gd php5-memcache php5-mysql php-apc \
  php5-xsl php5-imap libssh2-php
```

Record your mysql root password

Install these 32-bit packages that Kaltura requires (when we're on 64 bit):

```
sudo apt-get install ia32-libs lib32asound2 lib32gcc1 lib32ncurses5 \
  lib32stdc++6 lib32z1 libc6-i386
```

Install the JRE 6 for the video analytics module

```
sudo apt-get install default-jre
```

Enable some apache modules:

```
sudo a2enmod rewrite headers expires filter deflate file_cache env proxy
```

Modify `/etc/php/apache2/php.ini` and `/etc/php/cli/php.ini`and set `request_order = "CGP"`

Fix `/etc/php5/conf.d/imap.ini` (change `#` to `;` to comment out first line, or you get error later)

Restart apache

```
sudo /etc/init.d/apache2 restart
```

Modify `/etc/mysql/my.cnf`

```
thread_stack = 262144  (update this)
lower_case_table_names = 1  (add this)
```

Restart mysql

```
sudo /etc/init.d/mysql restart
```

Install Pentaho for analytics module

```
sudo mkdir /usr/local/pentaho
cd /tmp
wget http://sourceforge.net/projects/pentaho/files/Data%20Integration/3.2.0-stable/pdi-ce-3.2.0-stable.tar.gz
sudo mv pdi-ce-3.2.0-stable.tar.gz /usr/local/pentaho
cd /usr/local/pentaho
sudo tar xvzf pdi-ce-3.2.0-stable.tar.gz
sudo mv data-integration pdi
```

Install xymon for monitoring

```
sudo apt-get install xymon
```

and set it up in Apache if interested.
[[http://www.kaltura.org/kaltura-ce-v30-installing-xymon-monitoring-package-ubuntu-1004-0]]

## Kaltura install

Download [KalturaCE](http://www.kaltura.org/project/community_edition_video_platform) (kalturaCE_v3.0.0.tgz). You need an account to get it. Instructure runs a slightly tweaked version available [here](https://s3.amazonaws.com/instructure-kaltura/kalturaCE_v3.0.0-instructure.tar.gz), but we recommend you install the official version.

```
cd ~
tar xvzf kalturaCE_v3.0.0.tgz
cd kalturaCE_v3.0.0
```

As per [[http://www.kaltura.org/error-when-trying-get-account-usage-data-internal-server-error]], you'll want to edit `package/app/dwh/ddl/dwh_ddl_install.sh` and fix the typo on line 9. Change UESR to USER.

```
sudo php install.php
```

answers (blank = default):

```
> n
>
>
>
> kalturadev.example.com
> you@example.com
> *******
>
>
> root
> *******
> http://kalturadev.example.com/hobbit/
> Y
```

Add this to the bottom of `/etc/apache2/apache2.conf`

```
# Include kaltura config
Include /opt/kaltura/app/configurations/apache/my_kaltura.conf
```

Note that it will create a new virtual host and log to `/opt/kaltura/`, not `/var/log/apache2/`

Change `/etc/hosts` first line to:

```
127.0.0.1       kalturadev.example.com      localhost
```

Restart Apache

```
sudo /etc/init.d/apache2 restart
```

You need to create start scripts for the search daemon and batch manager, so

```
cd /etc/init.d/
ln -s /opt/kaltura/app/scripts/searchd.sh kaltura-searchd
ln -s /opt/kaltura/app/scripts/serviceBatchMgr.sh kaltura-serviceBatchMgr
update-rc.d kaltura-searchd defaults
update-rc.d kaltura-serviceBatchMgr defaults
```

## Configure Canvas

In a web browser, go to `http://kalturadev.example.com/start/`

- Click on Admin Console (right image) and log in.
- Click Add New Publisher sub-tab under Publishers tab and add the new user.
- On Publisher Management page, select Manage in the Action dropdown menu for the new Publisher. This will log you in to the management console as that publisher.
- Click the Settings tab at the top.
- Click the Integration Settings sub-tab.

Copy the two IDs and Secret keys to the `canvas/config/kaltura.yml` file settings as follows:

```
Partner ID => partner_id
Sub Partner ID => subpartner_id
Administrator Secret => secret_key
User Secret => user_secret_key
```

- Click on Studio tab.

Select one of these players or add a new one, and copy the ID number for it to the `canvas/config/kaltura.yml` file for the `player_ui_conf` and `upload_ui_conf` values (I was told `upload_ui_conf` doesn’t matter yet, so not sure what should go here).

To get the `kcw_ui_conf` value, you need to log into the mysql db on the kaltura server:

```
> mysql -u root -p
mysql> use kaltura;
mysql> select * from ui_conf where obj_type=2;
```

Pick one of these (such as “default KCW for KMC”) and copy the “id” number to `canvas/config/kaltura.yml` for the `kcw_ui_conf` value.

Set the remaining `canvas/config/kaltura.yml` settings while here:

```
domain: kaltura server url (no http://)
resource_domain: same as above
```

Restart Canvas Apache and Canvas_init.
