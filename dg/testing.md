<frontmatter>
  header: header.md
  title: "DG: Testing"
  pageNav: 2
  siteNav: dg-nav.md
  footer: footer.md
</frontmatter>

# Testing

This page contains information useful for testing of CATcher.

-------------------------------------------------------------------------------------

## Loading custom sessions

This is a hidden CATcher feature useful for manual testing.

If your sessions are not present in the default dropdown list on CATcher's startup page, you can load custom sessions by clicking on the **file icon** beside the session dropdown.

Following which, submit a file with the `.json` file extension, where the format is specified below.

```json
{
"profiles": [
    {
      "profileName": "CATcher",
      "encodedText": "CATcher-org/public_data"
    }
  ]
}
```

The json supplied should only consist of **one key-pair value**, where the key is `"profiles"` and the value is an array of `Profiles`, where each `Profile` is an object containing the `profileName` and `encodedText` fields.

`profileName` refers to the profile name displayed in the session select page. `encodedText` refers to the repository which stores the required settings for your custom session. The `encodedText` will be in the format of `organisation/repository`.

> **Note**: You **must** have both of these fields in each `Profile` and the values for these fields **should not be empty**! Else, the `.json` file that you have supplied will not be parsed successfully.

-------------------------------------------------------------------------------------

## Testing with Jasmine

Jasmine is a behavior-driven development framework specific for testing JavaScript code. We follow the Jasmine Style Guide loosely for our tests (Link under [Resources for Jasmine](#resources-for-jasmine)). One main guideline is that a `describe` block should be created for each method / scenario under test, and an `it` block should be created for each property being verified.

****Resources:****

1. [Jasmine Style Guide](https://github.com/CareMessagePlatform/jasmine-styleguide)
2. [Official Jasmine documentation](https://jasmine.github.io/api/3.6/global) : This is the official Jasmine documentation for Jasmine 3.6
3. [Introduction to Jasmine 2.0](https://jasmine.github.io/2.0/introduction.html) : This is a good summary / introduction of Jasmine test features

-------------------------------------------------------------------------------------

## Angular testBed utility

Because the above Jasmine framework does not test the DOM, we require the Angular TestBed Utility functions to set up component tests for testing HTML / view changes of components in CATcher.

Steps to set up component tests:
1. Configure the testing module through `TestBed` function `configureTestingModule` with the corresponding component's settings
2. Use `TestBed` function to create the component (fixture) to be tested
3. Observe HTML changes in the fixture during testing of functions by querying HTML elements of the fixture

You can refer to the [AssigneeComponent test](https://github.com/CATcher-org/CATcher/blob/master/tests/app/shared/issue/assignee/assignee.component.spec.ts) under our main repository for more details on how to set up a component test in CATcher.

****Resources:****
1. [Angular Guide - Basics of testing components](https://angular.io/guide/testing-components-basics) : Official Angular developer guide for the basics of component tests
2. [Angular Guide - Component Fixture](https://angular.io/api/core/testing/ComponentFixture) : Official Angular developer guide on `ComponentFixture`
3. [Introduction to Unit Testing in Angular](https://www.digitalocean.com/community/tutorials/angular-introduction-unit-testing#:~:text=Fixtures%20have%20access%20to%20a,Angular%20to%20run%20change%20detection.) : Useful article on how to test component fixtures

-------------------------------------------------------------------------------------

## E2E testing

### Running E2E tests

<tooltip content="end-to-end tests">E2E tests</tooltip> can be executed using `npm run e2e`. You should see CATcher launch in an instance of Google Chrome, with some automated actions occurring on it. Note: Google Chrome needs to be installed on the machine.

Unlike the production version of CATcher, we do not use the actual GitHub API in the E2E tests. Mock data is used to simulate the GitHub API. You can run `npm run ng:serve:test` to run CATcher in this "offline" mode (to further develop or debug the E2E tests).
The following additional parameters would allow for further customisation,

| Additional Parameter | Default | Description | Full Command Example | Command Explanation
| :---: | :---: | :-----: | :-------: | :------: |
| `--protractor-config=e2e/protractor.*.conf.js` | `protractor.conf.js` | Allows selection of the Protractor configuration file | `npm run e2e -- --protractor-config=e2e/protractor.firefox.conf.js` | Runs E2E Tests on the Firefox Browser |
| `--suite=*` | All Suites | Runs E2E Tests for specific suites | `npm run e2e -- --suite=login,bugReporting` | Run E2E Tests from Login and BugReporting Suites only |

<box type="warning" seamless>

Relevant Browsers must be installed prior to running tests (i.e. Chrome, Firefox).
</box>

### Troubleshooting conflicts between the versions of the browser and browser driver

If tests fail on your machine due to mismatches between the versions of the browser and the browser driver, you can use the [`webdriver-manager`](https://github.com/angular/webdriver-manager#readme) tool to install the right version of the driver.  By default, running `webdriver-manager update` updates all drivers to the latest version, but particular versions can be specified as options.

### Protractor Configuration

E2E Tests are run using [Protractor](http://www.protractortest.org/#/) testing framework.

- Protractor primarily requires the `*.conf.js` files to define E2E Testing Environments (this includes Browser Details, Base URL, etc...)
- The base configuration data is stored in `protractor.base.conf.js` which is then extended by separate configuration files for individual browsers as well as the CI/CD pipeline.
- E2E Tests are typically split into `Page-Objects Files` and `Test Files` in accordance with the [Protractor Style Guide](http://www.protractortest.org/#/style-guide) (more information regarding the interaction between the aforementioned filetypes can be found there).
- E2E Tests are also grouped into suites based on the Application's Phase (i.e. Login, Bug-Reporting). Currently defined suite information is located in the `protractor.base.conf.js` file as well.

### How the E2E tests work

E2E Tests are run with the following stages:
1. Build CATcher using `test` architecture
   - Using `test` build configuration located in `angular.json` under `projects.catcher.architect.configurations` we build a version of CATcher within a test environment that replaces `src/environments/environment.ts` with `src/environments/environment.test.ts` on runtime. This file provides data that allows CATcher to switch into "E2E test" mode.
2. Provide Test Environment Information
   - The Test Environment (in `src/environments/environment.test.ts`) provides information such as,
     - Login Credentials (Username).
     - User Role and Team Information.
     - A `test` flag that is set to `true`, so that CATcher switches into "E2E test mode"
3. Mock Service Injections
   - Once CATcher switches to E2E test mode, it creates mocks of some services, in order to simulate behaviour that is outside the scope of E2E Testing. This includes authentication, and communication with GitHub (via its APIs).
   - These Service Injections are carried out in the respective `*-module.ts` files with the help of Factories (located in `/src/app/core/services/factories`) that check the current build environment and make the Service Replacements accordingly.
4. Browser Action Automation using Protractor
   - With the application ready for testing, we then utilize `Protractor` to run test cases that are located in the `/e2e` directory.

