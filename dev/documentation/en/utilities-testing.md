---
title: Testing - Technical Documentation - OJS/OMP
---

# Testing Overview

{:toc}

PKP software uses a variety of tools to support testing:

* [Cypress](https://cypress.io) for integration testing and test data creation
* [PHPUnit](https://phpunit.de/) for unit testing

This documentation will not cover these tools in depth; please refer to the excellent documentation available for each.

# Unit Testing

PKP software uses [PHPUnit](https://phpunit.de/) for unit testing. Unit tests are implemented in `lib/pkp/tests` (that's tests for the `pkp-lib` submodule) and `tests` (that's tests for the application itself, i.e. OJS or OMP).

To run the tests, execute:
```
sh lib/pkp/tools/runAllTests.sh
```

# Test Data

OJS, OMP, and the Preprint Sever all include a set of [Cypress](https://cypress.io) tests that generate a test data set. These tests will take an empty environment and install the software, create a context (journal/press/server), and populate it with content, including users, submissions, peer reviews, etc.

In order to generate the test data, you will need an empty database and file storage area. Navigating to your installation of OJS, OMP, or PPS should present the installation page.

## Configuring your environment

You will need to tell Cypress about your environment. You can do this in several ways, but we recommend [creating a `cypress.env.json` file]([https://docs.cypress.io/guides/guides/environment-variables.html#Option-2-cypress-env-json) in your application's installation directory as follows:
```
{
	baseUrl: "http://localhost/path/to/application",
	DBTYPE: "MySQLi",
	DBUSERNAME: "database_username_here",
	DBPASSWORD: "database_password_here",
	DBNAME: "database_name_here",
	DBHOST: "127.0.0.1",
	FILESDIR: "/path/to/files_dir"
}
```
Alternately, you can use environment variables, but be sure to prefix each with `CYPRESS_` (see [Cypress documentation](https://docs.cypress.io/guides/guides/environment-variables.html#Option-3-CYPRESS)).

## Building the test data

Once your environment is configured, you can build the test data by simply running...
```
node_modules/cypress/bin/cypress run --config integrationFolder=lib/pkp/cypress/tests/data
```

This may take 15-30 minutes to build.

# Integration Tests

Integration tests build upon the test data set to check for regressions in various parts of the software. These are implemented in...

* `lib/pkp/cypress/tests/integration` for aspects that are included in the `pkp-lib` submodule (`lib/pkp` in your installation),
* `cypress/tests/integration` for aspects that are specific to the application you are working with, and
* `cypress/tests` within plugin repositories for plugins that support integration testing.

To run each of these, enter the application's installation directory and specify the directory that contains the tests you wish to run, e.g.:
```
cd /path/to/ojs
node_modules/cypress/bin/cypress run --config integrationFolder=lib/pkp/cypress/tests/integration
```

# Continuous Integration

PHP software uses [Travis CI](https://travis-ci.org) for Continuous Integration (CI) testing. This includes all of the types of tests listed above -- unit tests, data build tests, and integration tests. Thanks to Travis's CI service and the mechanisms described above, PKP software is continuously tested for regressions as it is developed.

## Application Testing

Whenever a new commit is pushed to the application repository (e.g. OJS or OMP), the Travis service will run the test data, integration, and unit tests. These are scripted in the repository's `.travis.yml` file.

## Plugin CI Testing

Some plugins support testing of their own, such as the [Static Pages](https://github.com/pkp/staticPages/) and [Quick Submit](https://github.com/pkp/quickSubmit) plugins. These plugins make use of certain parts of the application tests to prepare a test environment so that the plugin can be tested succinctly.

See the `.travis.yml` file included in these plugins for an example. The plugin tests are implemented in `cypress/tests`.

## Debugging

CI testing can be difficult to debug. It is best to work on your local machine and only run CI testing when your local tests are reliable. However, occasionally CI tests will fail on the Travis environment even when they run without problems locally. Here are some suggestions for debugging:

### Travis Debug Mode

When [Travis debug mode](https://docs.travis-ci.com/user/running-build-in-debug-mode/) is enabled, it's possible to log into the Travis environment via SSH and explore the tests manually.

### Dumping Screenshots

Use the uuencode tool on the Travis VM to permit dumping of screenshot files via the Travis console log. To do this, add the following to `.travis.yml`...

```
after_failure:
  - sudo apt-get install sharutils
  - tar cz cypress/screenshots | uuencode /dev/stdout
```

With this in place, when a test fails, the log will contain a uuencoded dump of screenshots of the failure. To extract the screenshots from the Travis log, save the raw log to your local machine, then run:
```
uudecode /path/to/log.txt | tar xvz
```

