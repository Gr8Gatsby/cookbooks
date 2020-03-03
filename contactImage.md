# Component 1: `contactImage`

This image component will exercise some key concepts of LWC by building an HTML Template, binding data from JavaScript code to the HTML Template, and finally using Saleforce Metadata to expose configurations to Salesforce Admins.

We will build this component first using an Apex Class to provide the data.

The recipes that are helpful for APEX are:
**TODO: add recipes**

### Step 1: Create the component

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name **`contactImageApex`**
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
  ```
    sfdx force:lightning:component:create --type lwc -n contactImage -d force-app/main/default/lwc
  ```

NOTE: [SFDX Command Line Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)

### Step : add method to `ContactController.cls`

Add this method after the `getSingleContact()` method inside of the `ContactController` class, this is located in `force-app/main/default/classes/ContactController.cls`:

```javascript
    @AuraEnabled(cacheable=true)
    public static Contact getSingleContactById(Id recordId) {
        return [
            SELECT Id, Name, Title, Phone, Email, Picture__c
            FROM Contact
            WHERE Id = :recordId
            LIMIT 1
        ];
    }
```

### Step 2: add imports

-In the `contactImage.js` file update the import statment to include all the decrators `@api`, `@wire`, `@track`

to this:

```javascript
import {LightningElement, api, wire, track} from 'lwc';
```

### Step 3: add import to apex

In the `contactImage.js` file add this import:

```javascript
import getSingleContactById from '@salesforce/apex/ContactController.getSingleContactById';
```

### Step 4: add import for scehma

Using Salesforce schema ensure referential integrity and keeps strings out of JavaScript code
Add this import to reference `@salesforce/schema` in your class

```javascript
import CONTACT_IMAGE_FIELD from '@salesforce/schema/Contact.Picture__c';
```

### Step 5: create an object for Styles

In order to expose an ability to admins to configure the image size add this `const`

```javascript
const SIZE_CLASS_MAP = {
  Small: 'slds-avatar_small',
  Medium: 'slds-avatar_medium',
  Large: 'slds-avatar_large',
  Massive: 'x-large-avatar',
};
```

### Step 6: add class properties

Setup the basic properties in the class in order to expose a public API to Admins and track internal states for HTML rendering.

```javascript
    // Automatically provided on record pages
    @api recordId;
    // expose a public API with default
    @api imageSize = 'Small';
    // make contact property reactive
    @track contact;
    // detect Image Load Errors
    @track imgLoadError = false;
```

Conceptually with these APIs the component would look like this in the HTML Document:

```html
<c-contact-image-apex
  record-id="AAAAAAAAAA"
  image-size="Small"
></c-contact-image-apex>
```

### Step 7: add call to `@wire`

```javascript
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
```

### Step 8: add Markup to the HTML Template

open the `contactImageApex.html` file and add this basic markup.

```html
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
```

### Step 9: add methods to class to interact with HTML template

add a method to return the correct styles based on the image-size `@api` attribute

```javascript
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
```

### Step 10: setup the Salesforce meta-data in `js-meta.xml`

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

### Step 11: add CSS

Create a file called `contactImageApex.css` paste in this code. `x-large-avatar` provides the style information for the `Massive` option.

```css
.x-large-avatar {
  width: 96px;
  height: 96px;
  max-width: 100%;
  max-height: 100%;
}

.img-error {
  font-size: 200%;
}
```

### Step 12: add SVG

Create a unique icon for this control. SLDS has [some icons](https://lightningdesignsystem.com/icons/) that can be used.

Create a file called `contactImageApex.svg` and paste in this code

```html
<svg
  xmlns="http://www.w3.org/2000/svg"
  viewBox="0 0 24 24"
  id="new_person_account"
>
  <path
    d="M19.2 15.3c-1.3-.5-1.5-1-1.5-1.6 0-.5.3-1.1.8-1.5.8-.7 1.2-1.7 1.2-2.8 0-2.2-1.4-4-3.8-4s-3.8 1.8-3.8 4c0 1.2.5 2.1 1.2 2.8.5.4.9 1 .9 1.5 0 .6-.3 1.1-1.6 1.6-2 .9-3.8 1.9-3.9 3.6 0 1.2.9 2.3 2 2.3H21c1.2 0 2-1.1 2-2.3.1-1.7-1.8-2.7-3.8-3.6zM10.4 17"
  ></path>
  <path
    d="M10.8 12.5c-.1-.2-.9-1.1-.8-3.6 0-2.4 1.1-3 1.1-3V3.5c0-.5-.4-.7-.7-.7H1.6s-.7.3-.7.7v16.1h5c.1-4.1 4.9-5.8 4.9-5.8.6-.3.1-1.1 0-1.3zm-5.9 5.1c0 .4-.3.7-.7.7h-.8c-.4 0-.7-.3-.7-.7v-.8c0-.4.3-.7.7-.7h.8c.4 0 .7.3.7.7v.8zm0-3.7c0 .5-.3.8-.7.8h-.8c-.4 0-.7-.3-.7-.8v-.7c0-.4.3-.7.7-.7h.8c.4 0 .7.3.7.7v.7zm0-3.7c0 .5-.3.8-.7.8h-.8c-.4 0-.7-.3-.7-.8v-.7c0-.4.3-.7.7-.7h.8c.4 0 .7.3.7.7v.7zm0-3.6c0 .4-.3.7-.7.7h-.8c-.4 0-.7-.3-.7-.7v-.7c0-.5.3-.8.7-.8h.8c.4 0 .7.3.7.8v.7zM9 13.9c0 .5-.3.8-.7.8h-.7c-.4 0-.8-.3-.8-.8v-.7c0-.4.4-.7.8-.7h.7c.4 0 .7.3.7.7v.7zm0-3.7c0 .5-.3.8-.7.8h-.7c-.4 0-.8-.3-.8-.8v-.7c0-.4.4-.7.8-.7h.7c.4 0 .7.3.7.7v.7zm0-3.6c0 .4-.3.7-.7.7h-.7c-.4 0-.8-.3-.8-.7v-.7c0-.5.4-.8.8-.8h.7c.4 0 .7.3.7.8v.7z"
  ></path>
</svg>
```

### Step 13: Write a test

```javascript
sfdx force:lightning:lwc:test:create -f force-app/main/default/lwc/contactImageApex/contactImageApex.js
```

or create a folder inside of the component directory called `__tests__`

create a file called `contactImageApex.test.js`

and paste in this code to start

```javascript
import {createElement} from 'lwc';

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
      is: ContactImageApex,
    });
    document.body.appendChild(element);
    expect(1).toBe(2);
  });
});
```

### Step 14: add final jest test code

Create a directory `data`

In the `data` directory create a file called `contactDataApex.json`

add this `.json` data:

```json
{
  "Id": "0031h00000T7oscAAB",
  "Name": "Amy Taylor",
  "Title": "VP of Engineering Core",
  "Phone": "4152568563",
  "Email": "amy@demo.net",
  "Picture__c": "https://s3-us-west-1.amazonaws.com/sfdc-demo/people/amy_taylor.jpg"
}
```

Edit `contactImageApex.test.js` to be:

```javascript
import {createElement} from 'lwc';
import {registerApexTestWireAdapter} from '@salesforce/sfdx-lwc-jest';
import getSingleContactById from '@salesforce/apex/ContactController.getSingleContactById';

import ContactImageApex from 'c/contactImageApex';

// Realistic data with a contacts
const mockGetContact = require('./data/contactDataApex.json');

// Register as Apex wire adapter.
const getContactAdapter = registerApexTestWireAdapter(getSingleContactById);

describe('c-contact-image-apex', () => {
  afterEach(() => {
    // The jsdom instance is shared across test cases in a single file so reset the DOM
    while (document.body.firstChild) {
      document.body.removeChild(document.body.firstChild);
    }

    // Prevent data saved on mocks from leaking between tests
    jest.clearAllMocks();
  });

  it('has an ALT tag on the image', () => {
    const element = createElement('c-contact-image-apex', {
      is: ContactImageApex,
    });
    document.body.appendChild(element);

    getContactAdapter.emit(mockGetContact);

    return Promise.resolve().then(() => {
      const imageElement = element.shadowRoot.querySelectorAll('img');
      expect(imageElement[0].hasAttribute('alt')).toBe(true);
    });
  });
});
```

### Step 15: run the tests

In a terminal window type

```
npm run test:unit --watch
```
