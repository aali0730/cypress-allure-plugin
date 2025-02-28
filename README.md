# cypress-allure-plugin

> Plugin for integrating allure reporter in Cypress with support of Allure API.

![Build][gh-image]
[![Downloads][downloads-image]][npm-url]
[![semantic-release][semantic-image]][semantic-url]  
[![version][version-image]][npm-url]
[![License][license-image]][license-url]

## Installation

-   [Allure binary](https://docs.qameta.io/allure/#_get_started): [directly from allure2](https://github.com/allure-framework/allure2#download) or [allure-commandline npm package](https://www.npmjs.com/package/allure-commandline).

-   [Java 8](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html) (required to run allure binary)

-   There is no need to set this plugin as reporter in Cypress or use any other allure reporters. Just download:

    -   using yarn:

    ```bash
    yarn add -D @shelex/cypress-allure-plugin
    ```

    -   using npm:

    ```
    npm i -D @shelex/cypress-allure-plugin
    ```

## Setup

-   Connect plugin in `cypress/plugins/index.js`. Take into account that Cypress generate plugins file with `module.exports` on the first initialization but you should have only one export section. In order to add Allure writer task just replace it or add writer task somewhere before returning config:

    -   as only plugin:

    ```js
    const allureWriter = require('@shelex/cypress-allure-plugin/writer');

    module.exports = (on, config) => {
        allureWriter(on, config);
        return config;
    };
    ```

    -   if you have webpack or other preprocessors please set allure writer before returning "config":

    ```js
    module.exports = (on, config) => {
        on('file:preprocessor', webpackPreprocessor);
        allureWriter(on, config);
        return config;
    };
    ```

-   Register commands in `cypress/support/index.js` file:

    -   with `import`:

    ```js
    import '@shelex/cypress-allure-plugin';
    ```

    -   with `require`:

    ```js
    require('@shelex/cypress-allure-plugin');
    ```

-   for IntelliSense (autocompletion) support in your IDE add on top of your `cypress/plugins/index.js` file:

```js
/// <reference types="@shelex/cypress-allure-plugin" />
```

-   for typescript support, update your tsconfig.json:

```json
"include": [
   "../node_modules/@shelex/cypress-allure-plugin/reporter",
   "../node_modules/cypress"
 ]
```

## Configuration

Plugin is customizable via Cypress environment variables:

| env variable name                      | description                                                          | default          |
| :------------------------------------- | :------------------------------------------------------------------- | :--------------- |
| `allure`                               | enable Allure plugin                                                 | false            |
| `allureResultsPath `                   | customize path to allure results folder                              | `allure-results` |
| `tmsPrefix`                            | just a prefix substring or pattern with `*` for links from allure API in tests to test management system  | ``               |
| `issuePrefix`                          | prefix for links from allure API in tests to bug tracking system     | ``               |
| `allureLogCypress`                     | log cypress chainer (commands) and display them as steps in report   | true             |
| `allureOmitPreviousAttemptScreenshots` | omit screenshots attached in previous attempts when retries are used | false            |  
| `allureAddAnalyticLabels` | add framework and language labels to tests (used for allure analytics only) | false            |  

This options could be passed:

-   via `cypress.json`

    ```json
    {
        "env": {
            "allureResultsPath": "someFolder/results",
            // tms prefix used without `*`, equivalent to `https://url-to-bug-tracking-system/task-*`
            "tmsPrefix": "https://url-to-bug-tracking-system/task-",
            "issuePrefix": "https://url-to-tms/tests/caseId-"
            // usage:  cy.allure().issue('blockerIssue', 'AST-111')
            // result: https://url-to-bug-tracking-system/task-AST-111
        }
    }
    ```

-   via `command line`:

    ```js
    yarn cypress run --env allure=true,allureResultsPath=someFolder/results
    ```

-   via `Cypress environment variables`:
    ```js
    Cypress.env('issuePrefix', 'url_to_bug_tracker');
    ```

## Execution

-   be sure your docker or local browser versions are next: Chrome 71+, Edge 79+. Firefox 65+

-   plugin might not be applied to older Cypress versions, 4+ is recommended

-   to enable Allure results writing just pass environment variable `allure=true`, example:

```bash
npx cypress run --env allure=true
```

-   if allure is enabled, you can check gathered data, in cypress window with Chrome Developer tools console:

```js
Cypress.Allure.reporter.runtime.writer;
```

## Examples

See [cypress-allure-plugin-example](https://github.com/Shelex/cypress-allure-plugin-example) project, which is already configured to use this plugin, hosting report as github page and run by github action. It has configuration for basic allure history saving (just having numbers and statuses in trends and history).  
For complete history (allure can display 20 build results ) with links to older reports and links to CI builds check [cypress-allure-historical-example](https://github.com/Shelex/cypress-allure-historical-example) with basic and straightforward idea how to achieve it.

There are also existing solutions that may help you prepare your report infrastructure:
-   [Allure docker service](https://github.com/fescobar/allure-docker-service) - highly customizable feature-rich container  
-   [Allure Server](https://github.com/kochetkov-ma/allure-server) - self-hosted portal with your reports
-   [allure-reports-portal](https://github.com/pumano/allure-reports-portal) - another portal which allows to gather reports for multiple projects in single ui
-   [Github Action](https://github.com/simple-elf/allure-report-action) - report generation + better implementation for historic reports described above
-   [Allure TestOps](https://docs.qameta.io/allure-testops/) - Allure portal for those who want more than report

## How to open report

Assuming allure is already installed:

-   serve report based on current "allure-results" folder: `allure serve`
-   generate new report based on current "allure-results" folder: `allure generate`
-   open generated report from "allure-report" folder: `allure open`

## API

There are three options of using allure api inside tests:

1. Using interface from `Cypress.Allure.reporter.getInterface()` - synchronous

```js
const allure = Cypress.Allure.reporter.getInterface();
allure.feature('This is our feature');
allure.epic('This is epic');
allure.issue('google', 'https://google.com');
```

2. Using Cypress custom commands, always starting from `cy.allure()` - chainer

```js
cy.allure()
    .feature('This is feature')
    .epic('This is epic')
    .issue('google', 'https://google.com')
    .parameter('name', 'value')
    .tag('this is nice tag');
```

3. Using Cypress-cucumber-preprocessor with cucumber tags:

```feature
@subSuite("someSubSuite")
@feature("nice")
@epic("thisisepic")
@story("cool")
@severity("critical")
@owner("IAMOwner")
@issue("jira","PJD:1234")
@someOtherTagsWillBeAddedAlso
Scenario: Here is scenario
...
```

Allure API available:

-   testID(id: string)
-   epic(epic: string)
-   feature(feature: string)
-   story(story: string)
-   suite(name: string)
-   label(name: LabelName, value: string)
-   parameter(name: string, value: string)
-   testParameter(name: string, value: string)
-   link(url: string, name?: string, type?: LinkType)
-   issue(name: string, url: string)
-   tms(name: string, url: string)
-   description(markdown: string)
-   testDescription(markdown: string)
-   descriptionHtml(html: string)
-   testDescriptionHtml(html: string)
-   owner(owner: string)
-   severity(severity: Severity)
-   tag(tag: string)
-   attachment(name: string, content: Buffer | string, type: ContentType)
-   testAttachment(name: string, content: Buffer | string, type: ContentType)
-   fileAttachment(name: string, path: string, type: ContentType)
-   startStep(name: string)
-   endStep()
-   step(name: string, isParent: boolean)
-   logStep(name: string)

## VS Code for cypress + cucumber

In case you are using VS Code and [Cypress Helper](https://marketplace.visualstudio.com/items?itemName=Shelex.vscode-cy-helper) extension, it has configuration for allure cucumber tags autocompletion available:

```js
"cypressHelper.cucumberTagsAutocomplete": {
        "enable": true,
        "allurePlugin": true,
        "tags": ["focus", "someOtherTag"]
    }
```

## Screenshots and Videos

Screenshots are attached automatically, for other type of content feel free to use `testAttachment` (for current test), `attachment` (for current executable), `fileAttachment` (for existing file).  
Videos are attached for failed tests only from path specified in cypress config `videosFolder` and in case you have not passed video=false to Cypress configuration. However videos will be linked by absolute path, so a trick to solve it is to set `"videosFolder": "allure-results"` (or value from your custom env variable 'allureResultsPath') in cypress.json or configuration options.
Please take into account, that in case spec files have same name, cypress is trying to create subfolders in videos folder, and it is not handled from plugin unfortunately, so video may not have correct path in such edge case.

## Cypress commands

Commands are producing allure steps automatically based on cypress events and are trying to represent how code and custom commands are executed with nested structure.  
Moreover, steps functionality could be expanded with:

-   `cy.allure().step('name')` - will create step "name" for current test. This step will be finished when next such step is created or test is finished.
-   `cy.allure().step('name', false)` OR `cy.allure().logStep('name')` - will create step "name" for current parent step/hook/test. Will be finished when next step is created or test finished.
-   `cy.allure().startStep('name')` - will create step "name" for current cypress command step / current step / current parent step / current hook or test. Is automatically finished on fail event or test end, but I would recommend to explicitly mention `cy.allure().endStep()` which will finish last created step.

## Testing

-   `yarn test:prepare:basic` - generate allure results for tests in `cypress/integration/basic`folder
-   `yarn test:prepare:cucumber` - generate allure results for tests in `cypress/integration/cucumber` folder
-   `test` - run tests from `cypress/integration/results` against these allure results

## Credits

A lot of respect to [Sergey Korol](serhii.s.korol@gmail.com) who made [Allure-mocha](https://github.com/allure-framework/allure-js/tree/master/packages/allure-mocha) reporter. Base integration with Cypress internal mocha runner is based on that solution.

## License

Copyright 2020-2021 Oleksandr Shevtsov <ovr.shevtsov@gmail.com>.  
This project is licensed under the Apache 2.0 License.

[npm-url]: https://npmjs.com/package/@shelex/cypress-allure-plugin
[gh-image]: https://github.com/Shelex/cypress-allure-plugin/workflows/build/badge.svg?branch=master
[types-path]: ./reporter/index.d.ts
[semantic-image]: https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg
[semantic-url]: https://github.com/semantic-release/semantic-release
[license-image]: https://img.shields.io/badge/License-Apache%202.0-blue.svg
[license-url]: https://opensource.org/licenses/Apache-2.0
[version-image]: https://badgen.net/npm/v/@shelex/cypress-allure-plugin/latest
[downloads-image]: https://badgen.net/npm/dt/@shelex/cypress-allure-plugin
