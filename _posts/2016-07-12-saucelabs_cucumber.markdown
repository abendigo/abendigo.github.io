---
layout: post
title: Sauce Labs and Cucumber
---

We recently started a new project using [Web Components][web-components] and [Polymer][polymer].
We want to add some behaviour driven tests, using [Cucumber][cucumber] and [Selenium][selenium]
but do not want to continue using our in-house grid of Selenium nodes. It has proven too time
consuming and frustrating to set up and maintain them.

The goal is to have our CI system ([GoCD][gocd]) checkout and build pull requests (managed via
[Phabricator][phabricator]), deploy them to an S3 bucket on [AWS][aws], run the test suite, and
update the request with the results. I may get into this in later posts.

After a little searching, I found two main players offering hosted selenium servers.
[BrowserStack][browserstack] and [Sauce Labs][saucelabs]. Since [WCT][wct] has built in
support for Sauce Labs, I decided to give it a try first.

We are using [Gulp][gulp] as a task runner, so I looked for gulp plugins that could help.
I quickly found [gulp-cucumber][gulp-cucumber]. Unfortunately, our app needs to access some
internal API's, so I needed to make use of Sauce Labs sauce-connect local tunnel service.
I could not find a gulp plugin for this, but, I did find this [little gist][gist].
So, borrowing heavily, I hacked together this little gulp file:

    const gulp = require('gulp');
    const cucumber = require('gulp-cucumber');
    const sauceConnectLauncher = require('sauce-connect-launcher');

    gulp.task('default', () => {
      sauceConnectLauncher({
        username: process.env.SAUCE_USERNAME,
        accessKey: process.env.SAUCE_ACCESS_KEY
      }, (err, sauceConnectProcess) => {
        if (err) {
          throw err.message;
        }
        console.log("Sauce Connect ready");

        return gulp.src('features/marko.feature')
          .pipe(cucumber({}))
          .on('error', function(e) {
            sauceConnectProcess.close(function () {
              console.log("Closed Sauce Connect process");
            });
            throw e;
          })
          .on('end', function(e) {
            sauceConnectProcess.close(function () {
              console.log("Closed Sauce Connect process");
            });
          });
      });
    });



[polymer]: https://www.polymer-project.org/1.0/
[web-components]: http://webcomponents.org/
[cucumber]: https://cucumber.io/
[selenium]: http://www.seleniumhq.org/
[browserstack]: https://www.browserstack.com/
[saucelabs]: https://saucelabs.com/
[gocd]: https://www.go.cd/
[phabricator]: https://www.phacility.com/phabricator/
[aws]: https://aws.amazon.com/
[wct]: https://github.com/Polymer/web-component-tester
[gulp]: http://gulpjs.com/
[gulp-cucumber]: https://www.npmjs.com/package/gulp-cucumber
[gist]: https://gist.github.com/intergalactic-overlords/83114074087666967da3
