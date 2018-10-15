Canvas by Instructure!
===================

Canvas is a well-established (circa 2010, used in many colleges, universities, and K-12 schools), open-source [LMS](http://en.wikipedia.org/wiki/Learning_management_system) by [Instructure Inc](http://www.canvaslms.com/). It is released under the [AGPLv3](http://www.gnu.org/licenses/agpl.html) license for use by anyone interested in learning more about or using learning management systems.

Our [Canvas Dev & Friends](https://instructure.github.io/) site is a great resource for getting started in developing on Canvas itself, using the Canvas APIs, or creating well-integrated 3rd party tools.

Are you looking for one of our commercial subscriptions, professional services, support, or our hosted solution? Check out [canvaslms.com](https://www.canvaslms.com/).

Trying Out Canvas
-----------------

If you'd like to use Canvas in your own course, or you just want try Canvas out, there is no need to do all the work of installing it yourself. Canvas Cloud is free for teachers to use, [sign up](https://canvas.instructure.com/register) to get started.

If at a later time you do want to migrate from your own install to Canvas Cloud, or vice versa, you can use the course export and import features to migrate all your content.

Getting Help
-----------

 * [[FAQ]]
 * [Installation Tutorials](#installation-tutorials)
 * [API docs](http://api.instructure.com)
 * [Canvas Guides](https://guides.instructure.com)
 * [User Group Mailing List](http://groups.google.com/group/canvas-lms-users)
 * [#canvas-lms on Freenode](http://webchat.freenode.net/?channels=canvas-lms&uio=d4)
 * [Canvas Vimeo Channel](https://vimeo.com/canvaslms)

Contributing
-----------

In order for us to continue to dual-license our Canvas product to best serve all of our customers, we need you to sign before we can accept a pull request from you. After submitting a pull request, you'll see a status check that indicates if a signature is required or not. If the CLA check fails, click on Details and then complete the web form. Once finished, the CLA check on the pull request will pass successfully. Please read our [[FAQ]] for more information, and be sure to review the [[Coding Guidelines]].

To save yourself a considerable headache, please consider doing development against our master branch, instead of the default stable branch. Our stable branch is occasionally reforked from master, so your Git history may get very confused if you are attempting to contribute changesets against stable.

If you would like to contribute a translation change, please submit it through our translation service [[Transifex|https://www.transifex.com/instructure/canvas-lms/]].

See [CONTRIBUTING.md](https://github.com/instructure/canvas-lms/blob/stable/CONTRIBUTING.md) for additional details on contributing.

Security
-----------

We take security very seriously. **Please do not submit issues or pull requests for security issues**. Instead, you can email security@instructure.com and we will respond as soon as possible. You can watch for security advisories to update your Canvas installation on our [Security Notices forum][security-notices].

Installation Tutorials
--------

 * [[Quick Start]]
 * [[Production Start]]
 * [[Upgrading]]
 * [[Troubleshooting your Canvas installation|Troubleshooting]]
 * [[Integrating Canvas with other systems|Canvas Integration]]

Getting Code
-----------

There are two primary ways to get a copy of Canvas LMS: git and zip/tarball download.

## Using Git

You can install [Git](http://git-scm.com/) on Debian/Ubuntu by running

```
$ sudo apt-get install git-core
```

Next, clone this repo: 

```
~$ git clone https://github.com/instructure/canvas-lms.git canvas
~$ cd canvas
```

To use develop against production code, checkout the `origin/stable` branch:

```
~/canvas$ git checkout --track -b stable origin/stable
```

### Using a Tarball or a Zip

You can also download a tarball or zip file.
  
   * [Canvas Tarball](http://github.com/instructure/canvas-lms/tarball/stable)
   * [Canvas Zip](http://github.com/instructure/canvas-lms/zipball/stable)

[security-notices]: https://community.canvaslms.com/community/answers/security