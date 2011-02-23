Troubleshooting
==========

Is your question not on here? Please contact us over [our user mailing list](http://groups.google.com/group/canvas-lms-users).

### How do I get Canvas to see my changes to the configuration files?

If you are running a [[production|Production-Start]] setup, then you need to restart both the webserver serving Canvas, and the automated jobs daemon. On Debian/Ubuntu, you can do this like:

```
sysadmin@appserver:~$ sudo /etc/init.d/apache2 restart
sysadmin@appserver:~$ sudo /etc/init.d/canvas_init restart
```

The [[quick-start|Quick-Start]] setup may only require stopping and starting the `script/server` process, if it requires any restart at all.

### Why isn't outgoing mail working?

There are a number of possible reasons why your outgoing mail configuration may not be working correctly. One of the easiest ways to figure out where your Canvas mail is going is to check the outgoing mail table's error column. You can see the last error that Canvas had (if any) after some scheduled job (such as email) by running:

```
sysadmin@appserver:~$ echo "select last_error from delayed_jobs order by updated_at desc limit 1;" | \
                      mysql -u root -p -D canvas_queue_production
```

### I'm not even sure automated jobs like email are getting picked up.

Do you have the automated jobs daemon running? Try

```
sysadmin@appserver:~$ sudo /etc/init.d/canvas_init status
```

If you want more information about the specific kinds of jobs that may or may not be running in your system, you can visit

```
https://<your-canvas-hostname>/delayed_jobs
```

### Something else is broken.

Try checking the *error_reports* table in the database for any messages possibly pertaining to the problem:

```
sysadmin@appserver:~$ echo "select message, backtrace from error_reports order by id desc limit 1;" | \
                      mysql -u root -p -D canvas_production
```

### How do I access the Rails console?

If you followed the [[Production Start]] instructions, you can get a Rails console open by running (with appropriate adjustments):

```
sysadmin@appserver:~$ cd /var/rails/canvas
sysadmin@appserver:/var/rails/canvas$ sudo su canvasuser -c "env GEM_HOME=/home/sysadmin/gems \
                                      RAILS_ENV=production script/console"
```

Accessing a Rails console gives you unprecedented control over Canvas' inner workings. Be sure you know what you're doing.
