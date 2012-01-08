Canvas by Instructure!
===================

Canvas is a new, open-source [LMS](http://en.wikipedia.org/wiki/Learning_management_system) by [Instructure Inc](http://www.instructure.com/). It is released under the [AGPLv3](http://www.gnu.org/licenses/agpl.html) license for use by anyone interested in learning more about or using learning management systems.

Are you looking for one of our commercial licenses, professional services, support, or our hosted solution? [Please visit our main website](http://www.instructure.com/).

Getting Help
-----------

 * You can join [our announcements mailing list](http://groups.google.com/group/canvas-lms-announce).
 * You can post a message or participate in [our user group mailing list](http://groups.google.com/group/canvas-lms-users).
 * You can join #canvas-lms on [Freenode](http://freenode.net/using_the_network.shtml) and talk to the community.
 * You can read one of our tutorials (below).
 * You can read our [[FAQ]].
 * You can visit our [general Canvas usage support portal](http://support.instructure.com/).
 * You can visit our [YouTube Channel](http://www.youtube.com/CanvasLMS#g/p).
 * You can read our [API docs](http://canvas.instructure.com/doc/api/index.html).

Contributing
-----------

In order for us to continue to dual-license our Canvas product to best serve all of our customers, we need you to sign [[our contributor agreement|ica.pdf]] before we can accept a pull request from you. Please read our [[FAQ]] for more information.

To save yourself a considerable headache, please consider doing development against our master branch, instead of the default stable branch. Our stable branch is occasionally reforked from master from time to time, so your Git history may get very confused if you are attempting to contribute changesets against stable.

Installation Tutorials
--------

 * [[Quick Start]]
 * [[Production Start]]
 * [[Upgrading]]
 * [[Troubleshooting your Canvas installation|Troubleshooting]]
 * [[Integrating Canvas with other systems|Canvas Integration]]

Getting Code
-----------
There are two primary ways to get a copy of Canvas LMS

### Using Git

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

### Using a Tarball or a Zip

You can also download a tarball or zipfile.
  
   * [Canvas Tarball](http://www.instructure.com/code/canvas-stable.tar.gz)
   * [Canvas Zip](http://www.instructure.com/code/canvas-stable.zip)
