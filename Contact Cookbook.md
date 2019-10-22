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

# Component 1: `contactImage`

This image component will exercise some key concepts of LWC by building an HTML Template, binding data from JavaScript code to the HTML Template, and finally using Saleforce Metadata to expose configurations to Salesforce Admins.

We will build this component first using an Apex Class to provide the data.

The recipes that are helpful for APEX are:
**TODO: add recipes**

### Step : Create the component

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name `contactImageApex`
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
    sfdx force:lightning:component:create --type lwc -n contactImage -d force-app/main/default/lwc

NOTE: [SFDX Command Line Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)

### Step : add method to `ContactController.cls`

Add this method after the `getSingleContact()` method inside of the `ContactController` class, this is located in `force-app/main/default/classes/ContactController.cls`:

    @AuraEnabled(cacheable=true)
    public static Contact getSingleContactById(Id recordId) {
        return [
            SELECT Id, Name, Title, Phone, Email, Picture__c
            FROM Contact
            WHERE Id = :recordId
            LIMIT 1
        ];
    }

### Step : add imports

-In the `contactImage.js` file update the import statment to include all the decrators `@api`, `@wire`, `@track`

to this:
`import { LightningElement, api, wire, track } from 'lwc';`

### Step : add import to apex

In the `contactImage.js` file add this import:
`import getSingleContactById from '@salesforce/apex/ContactController.getSingleContactById';`

### Step : add import for scehma

Using Salesforce schema ensure referential integrity and keeps strings out of JavaScript code
Add this import to reference `@salesforce/schema` in your class

`import CONTACT_IMAGE_FIELD from '@salesforce/schema/Contact.Picture__c';`

### Step : create an object for Styles

In order to expose an ability to admins to configure the image size add this `const`

    const SIZE_CLASS_MAP = {
        Small: 'slds-avatar_small',
        Medium: 'slds-avatar_medium',
        Large: 'slds-avatar_large',
        Massive: 'x-large-avatar'
    };

### Step : add class properties

Setup the basic properties in the class in order to expose a public API to Admins and track internal states for HTML rendering.

    // Automatically provided on record pages
    @api recordId;
    // expose a public API with default
    @api imageSize = 'Small';
    // make contact property reactive
    @track contact;
    // detect Image Load Errors
    @track imgLoadError = false;

Conceptually with these APIs the component would look like this in the HTML Document:

`<c-contact-image-apex record-id='AAAAAAAAAA' image-size='Small'></c-contact-image-apex>`

### Step : add call to `@wire`

     // Get data from APEX
    @wire(getSingleContactById, { recordId: '$recordId' })
    getContactData({ error, data }) {
        if (data) {
            this.contact = data;
            this.imgLoadError = false;
        } else if (error) {
            // TODO handle error
            this.contact = undefined;
        }
    }

### Step : add Markup to the HTML Template

open the `contactImageApex.html` file and add this basic markup.

     <template if:true={contact}>
        <span class={style}>
            <template if:false={imgLoadError}>
                <img onerror={handleImgError} src={pictureUrl} alt={pictureAlt} />
            </template>
            <template if:true={imgLoadError}>
                <div class="img-error">☹️</div>
            </template>
        </span>
    </template>

### Step : add methods to class to interact with HTML template

add a method to return the correct styles based on the image-size `@api` attribute

    // Return the correct style based on the @api attribute image-size
    get style() {
        return `slds-avatar ${SIZE_CLASS_MAP[this.imageSize]}`;
    }

    // Get the URL of the image to display
    get pictureUrl() {
        if (this.contact) {
            return this.contact.Picture__c;
        }

        return null;
    }

    // Generate an alt tag for the image
    get pictureAlt() {
        return `Image of ${this.contact.Name}`;
    }

    // Handle the case where there is an image load error
    handleImgError(evt) {
        window.console.log(evt.target);
        this.imgLoadError = true;
    }

### Step : setup the Salesforce meta-data in `js-meta.xml`

    <?xml version="1.0" encoding="UTF-8"?>
    <LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata" fqn="contactImageApex">
        <apiVersion>47.0</apiVersion>
        <isExposed>true</isExposed>
        <masterLabel>Contact Image Apex</masterLabel>
        <targets>
            <target>lightning__RecordPage</target>
        </targets>
        <targetConfigs>
            <targetConfig targets="lightning__RecordPage">
                <property name="imageSize" label="Image Size" type="String" datasource="Small, Medium, Large, Massive"/>
                <objects>
                <object>Contact</object>
            </objects>
            </targetConfig>
        </targetConfigs>
    </LightningComponentBundle>

### Step : add CSS

### Step : add SVG

### Step : Write a test

`sfdx force:lightning:lwc:test:create -f force-app/main/default/lwc/contactImageApex/contactImageApex.js`

or create a folder inside of the component directory called `__tests__`

create a file called `contactImageApex.test.js`

and paste in this code to start

    import { createElement } from 'lwc';

    import ContactImageApex from 'c/contactImageApex';

    describe('c-contact-image-apex', () => {
        afterEach(() => {
        // The jsdom instance is shared across test cases in a single file so reset the DOM
        while (document.body.firstChild) {
        document.body.removeChild(document.body.firstChild);
        }
        });

        it('TODO: test case generated by CLI command, please fill in test logic', () => {
            const element = createElement('c-contact-image-apex', {
                is: ContactImageApex
            });
            document.body.appendChild(element);
            expect(1).toBe(2);
        });
    });

# Component 2: `contactReportsTo`

This breadcrumb based component will show the reporting chain from the current Contact upto the highest manager. This component will leverage a child component

# Component 3: `contactReportsToCrumb`

This component will render a breadcrumb of a person in the `contactReportsTo`

#### Components:

- contactImage
  - APEX
  - getRecord UIAPI
  - Schema
  - @wire, @track, @api
  - JEST test
  - JS-META.XML
-
