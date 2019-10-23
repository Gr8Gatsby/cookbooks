# Contact Cookbook

This short cookbook is going to use the [LWC Recipes](https://github.com/trailheadapps/lwc-recipes) project to build a few new components for the Contact object. The core concepts that you will learn are:

- HTML Templates
  - Directives
    - if
    - iterator
    - for-each
  - Databinding Syntax
- JavaScript
  - Decorators
    - api
    - track
    - wire
  - import `@salesforce/schema`
  - import `@salesforce/apex`
  - getters
  - event handling
- Salesforce Metadata
  - `<isExposed>`
  - `<masterLabel>`
  - `@recordId` from RecordPage Context
  - `<targets>`
  - `<targetConfig>`

# Component : `contactCard`

# Component : `contactReportsTo`

This breadcrumb based component will show the reporting chain from the current Contact upto the highest manager. This component will leverage a child component

# Component : `contactReportsToCrumb`

This component will render a breadcrumb of a person in the `contactReportsTo`

#### Components:

- contactImage
- contactImageApex
- contactReportsTo
- contactReportsToCrumb
