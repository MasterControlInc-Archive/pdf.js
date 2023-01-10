# MasterControl Fork info
We forked PDF.js so that we could apply a few customizations to their viewer. We're working from a branch called mc-master. We've also added some gulp tasks to build and deploy to our artifactory instance. Follow the instructions from the original README below to install dependencies after you've checked out our branch.

## Maintaining our fork
The `master` branch in our fork should be identical to `master` on the upstream mozilla repo. Because of this it should always be safe to sync `master` with the upstream and push that to our repo without doing any special testing.

### Synchronizing with upstream mozilla:pdf.js
Generally speaking keeping up to date with our dependencies is a great thing. If you need to update our fork with the latest from mozilla's repo here is a quick runthrough:
- add upstream remote to your working copy
    ```
    $ git remote add upstream git@github.com:mozilla/pdf.js.git
    $ git fetch upstream
    ```
- merge `upstream/master` into our fork's master (`master` not `mc-master`)
    ```
    $ git switch master
    $ git merge upstream/master
    ```
- push `master` up to our fork's repo
    ```
    $ git push master
    ```

### Updating `mc-master` with changes from the upstream master
This should be done after syncing master as described above. The idea here is to separate this type of sync from your story/defect work for better history tracking. You can have multiple PRs for your story work, this would be one of them.

- Pick a release that you want to merge. Note: choose a released tag rather than master, since there could be unreleased/untested functionality in the master branch.
- Merge the tagged commit into our `mc-master`
    ```
    $ git switch -c merge-v3.1.81-into-mc-master
    $ git merge v3.1.81
    ```
- make sure things still work, deploy snapshot, test, etc
- Create a PR for `merge-v3.1.81-into-mc-master` that will go in before your story branch PR goes in. Make sure to use a squash-merge and note which version from Mozilla has been merged in. **Note: when creating a PR make sure to select our repository and _not_ Mozilla's.**
- Either base your story branch off this branch or merge it into your branch as you see fit.

### Building during development
To build for use in MasterControl during developmental testing you can either run

    $ gulp generic

Which will quickly build PDF.js for development purposes (not minified, packaged, etc)

OR

    $ gulp mc-build
    
This will build PDF.js in a minified form and zip it up in a file named `mcPDFjs-<version>.zip`. This is the artifact that we deploy to artifactory.

You can copy the result (or unzip the mc artifact) into `<site>/services/StaticContent/js/PDFjs/` so that folder contains the `build` and `web` folders.

### Deploying an artifact to Artifactory
Please make sure you don't have any uncommitted changes as they'd be zipped and deployed in the artifact too. Create the following environment variables with your artifactory credentials. The password could use your artifactory API key.  `artifactory_username` and `artifactory_password`. The following commands will first call `mc-build`.

#### Deploying a snapshot 
You should be able to deploy a snapshot to artifactory by running the following command.

    $ gulp mc-deploy-snapshot

#### Deploying a release 
**Only release builds from mc-master!** This will deploy straight to our libs-release-local repo, **you have been warned**.

First make sure the version is correct in pdfjs.config. Then run:

    $ gulp mc-deploy-release
    
Please don't forget to tag the commit you deployed and push the tags to the repository with commands similar to the following:

    $ git tag -a v3.1.37-mc -m "Release v3.1.37-mc"
    $ git push --tags origin mc-master
    
We're using the tagname pattern of `v[versionNumber]-mc` like `v3.1.37-mc` to help differentiate our tags since we're building from a different branch and the commit counting (used in the automatic version number creation) will be different and likely _behind_ the Mozilla PDFjs numbers.

# ORIGINAL README BELOW
# PDF.js [![Build Status](https://travis-ci.org/mozilla/pdf.js.svg?branch=master)](https://travis-ci.org/mozilla/pdf.js)

PDF.js is a Portable Document Format (PDF) viewer that is built with HTML5.

PDF.js is community-driven and supported by Mozilla Labs. Our goal is to
create a general-purpose, web standards-based platform for parsing and
rendering PDFs.

## Contributing

PDF.js is an open source project and always looking for more contributors. To
get involved, visit:

+ [Issue Reporting Guide](https://github.com/mozilla/pdf.js/blob/master/.github/CONTRIBUTING.md)
+ [Code Contribution Guide](https://github.com/mozilla/pdf.js/wiki/Contributing)
+ [Frequently Asked Questions](https://github.com/mozilla/pdf.js/wiki/Frequently-Asked-Questions)
+ [Good Beginner Bugs](https://github.com/mozilla/pdf.js/issues?direction=desc&labels=5-good-beginner-bug&page=1&sort=created&state=open)
+ [Projects](https://github.com/mozilla/pdf.js/projects)

Feel free to stop by #pdfjs on irc.mozilla.org for questions or guidance.

## Getting Started

### Online demo

+ https://mozilla.github.io/pdf.js/web/viewer.html

### Browser Extensions

#### Firefox

PDF.js is built into version 19+ of Firefox.

#### Chrome

+ The official extension for Chrome can be installed from the [Chrome Web Store](https://chrome.google.com/webstore/detail/pdf-viewer/oemmndcbldboiebfnladdacbdfmadadm).
*This extension is maintained by [@Rob--W](https://github.com/Rob--W).*
+ Build Your Own - Get the code as explained below and issue `gulp chromium`. Then open
Chrome, go to `Tools > Extension` and load the (unpackaged) extension from the
directory `build/chromium`.

## Getting the Code

To get a local copy of the current code, clone it using git:

    $ git clone https://github.com/mozilla/pdf.js.git
    $ cd pdf.js

Next, install Node.js via the [official package](https://nodejs.org) or via
[nvm](https://github.com/creationix/nvm). You need to install the gulp package
globally (see also [gulp's getting started](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md#getting-started)):

    $ npm install -g gulp-cli

If everything worked out, install all dependencies for PDF.js:

    $ npm install

Finally, you need to start a local web server as some browsers do not allow opening
PDF files using a `file://` URL. Run:

    $ gulp server

and then you can open:

+ http://localhost:8888/web/viewer.html

Please keep in mind that this requires an ES6 compatible browser; refer to [Building PDF.js](https://github.com/mozilla/pdf.js/blob/master/README.md#building-pdfjs) for usage with older browsers.

It is also possible to view all test PDF files on the right side by opening:

+ http://localhost:8888/test/pdfs/?frame

## Building PDF.js

In order to bundle all `src/` files into two production scripts and build the generic
viewer, run:

    $ gulp generic

This will generate `pdf.js` and `pdf.worker.js` in the `build/generic/build/` directory.
Both scripts are needed but only `pdf.js` needs to be included since `pdf.worker.js` will
be loaded by `pdf.js`. The PDF.js files are large and should be minified for production.

## Using PDF.js in a web application

To use PDF.js in a web application you can choose to use a pre-built version of the library
or to build it from source. We supply pre-built versions for usage with NPM and Bower under
the `pdfjs-dist` name. For more information and examples please refer to the
[wiki page](https://github.com/mozilla/pdf.js/wiki/Setup-pdf.js-in-a-website) on this subject.

## Including via a CDN

PDF.js is hosted on several free CDNs:
 - https://www.jsdelivr.com/package/npm/pdfjs-dist
 - https://cdnjs.com/libraries/pdf.js
 - https://unpkg.com/pdfjs-dist/

## Learning

You can play with the PDF.js API directly from your browser using the live demos below:

+ [Interactive examples](https://mozilla.github.io/pdf.js/examples/index.html#interactive-examples)

More examples can be found in the [examples folder](https://github.com/mozilla/pdf.js/tree/master/examples/). Some of them are using the pdfjs-dist package, which can be built and installed in this repo directory via `gulp dist-install` command.

For an introduction to the PDF.js code, check out the presentation by our
contributor Julian Viereck:

+ https://www.youtube.com/watch?v=Iv15UY-4Fg8

More learning resources can be found at:

+ https://github.com/mozilla/pdf.js/wiki/Additional-Learning-Resources

## Questions

Check out our FAQs and get answers to common questions:

+ https://github.com/mozilla/pdf.js/wiki/Frequently-Asked-Questions

Talk to us on IRC (Internet Relay Chat):

+ #pdfjs on irc.mozilla.org

File an issue:

+ https://github.com/mozilla/pdf.js/issues/new

Follow us on twitter: @pdfjs

+ https://twitter.com/pdfjs
