## Development Branch

To save yourself a considerable headache, please consider doing development against our master branch, instead of the default stable branch. Our stable branch is occasionally reforked from master, so your Git history may get very confused if you are attempting to contribute changesets against stable.

## Overview

Getting code folded into Canvas core is not trivial. You should take a second right now and see if maybe your change would be better implemented as a separate service using the API or LTI. We have a high quality standard at Instructure and if you're not familiar with both Ruby and Rails then getting your code accepted may be difficult. We're happy to give suggestions on how to improve commits, but we're not going to teach you how to write Ruby code when you issue a pull request.

Every commit for Canvas is reviewed by at least one Instructure engineer (this is all true of Instructure-created code as well, btw). Every line is read and the reviewer is responsible for checking out the commit and testing it in their local environment. As such, commit messages need to provide enough information that the engineer knows what has changed and what should be tested.

The following checklist is worked through for every commit:

- Check out and try the changeset.
- Ensure that the commit message has a test plan.
- Ensure that the tests and test plan cover all necessary cases.
- Ensure that the code follows the language coding conventions.
- Ensure that the code is well designed and architected.
- Ensure that all user-facing strings/dates/times/numbers are [internationalized](Internationalization-I18n).

Other factors that should be considered:

- Must remain performant under heavy load.
- Must work in a multi-tenant environment. More on that in a minute, but basically enhancements should be built using the Plugin architecture of Canvas.
- Must be accessible to screen readers and other assistive technology devices.
- Should follow our [coding style](Style-Guides).

## Places to Start

If you're new to Canvas development, there are guides in this wiki for getting your dev environment set up (including getting specs running). Make sure you've given accepted our coding license agreement, then _start with something small_. Get to know the commit process with something small like a bug fix or a UI tweak. If you're not sure where to start post a message on the mailing list.

Once you've got your feet under you then you can start working on larger projects. For anything more than a bug fix, it probably makes sense to coordinate through the mailing list, since it's possible someone else is working on the same thing.

## Pull Requests

We like GitHub pull requests. If you report an issue, we’d love to see a pull request attached. Just keep in mind that because of the development standards mentioned above your commit is probably going to end up getting modified at least once before it’s accepted. Sometimes we’ll make the change ourselves, but often we’ll just let you know what needs to happen and help you fix it up yourself.

## Enhancements and Extensions

Because Canvas Cloud runs as a multi-tenant environment, any changes to the codebase will affect all institutions at once. If you're looking to add major pieces of functionality to Canvas, you'll need to keep this in mind, since most likely only _some_ institutions will want that functionality added.

To help with this we've built the notion of Plugins into Canvas. Plugins can be registered at runtime but only appear in the interface for enabled root accounts. There are some places in the code that have already been instrumented for plugins (such as web conferences and collaborations), but if you're looking to extend functionality somewhere else then the first step is going to be pluginifying that portion of the code, _then_ building a plugin for your specific implementation.

The easiest way to get to know Canvas Plugins is to view these files: [lib/canvas/plugin.rb](https://github.com/instructure/canvas-lms/blob/master/lib/canvas/plugin.rb) and [lib/canvas/plugins/default_plugins.rb](https://github.com/instructure/canvas-lms/blob/master/lib/canvas/plugins/default_plugins.rb)
