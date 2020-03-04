### Step 1: Create the component `contactsSimilar`

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name **`contactReportsTo`**
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
  ```
    sfdx force:lightning:component:create --type lwc -n contactReportsTo -d force-app/main/default/lwc
  ```

NOTE: [SFDX Command Line Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)

### Step 2: Edit the `contactReportsTo.js`

replace the code with this

```javascript
import {LightningElement, wire, track, api} from 'lwc';
import {getRecord, getFieldValue} from 'lightning/uiRecordApi';

// Import Salesforce Schema
import CONTACT_ID_FIELD from '@salesforce/schema/Contact.Id';
import CONTACT_NAME_FIELD from '@salesforce/schema/Contact.Name';
import CONTACT_IMAGE_FIELD from '@salesforce/schema/Contact.Picture__c';
import CONTACT_REPORTSTO_ID from '@salesforce/schema/Contact.ReportsToId';
//import CONTACT_REPORTSTO_MANGER_ID from '@salesforce/schema/User.ReportsToId.ReportsToId.Name';
//Check out WireGetObjectInfo Recipe to see Object relationshipName

const CONTACT_FIELDS = [
  CONTACT_REPORTSTO_ID,
  CONTACT_ID_FIELD,
  CONTACT_NAME_FIELD,
  CONTACT_IMAGE_FIELD,
];

export default class ReportsTo extends LightningElement {
  @api recordId;
  @track organization = [];
  @track reportsToId;

  // Find if the current Contact has a ReportsToId
  @wire(getRecord, {
    recordId: '$recordId',
    fields: CONTACT_FIELDS,
    //optionalFields: [CONTACT_REPORTSTO_MANGER_ID]
  })
  processCurrentContact({error, data}) {
    if (data) {
      this.organization = [];
      // Add person to the organization
      this.addPersonToOrganization({
        Id: this.recordId,
        Name: getFieldValue(data, CONTACT_NAME_FIELD),
        ReportsToId: getFieldValue(data, CONTACT_REPORTSTO_ID),
      });
      this.error = undefined;
    } else if (error) {
      this.handleError(error);
    }
  }

  @wire(getRecord, {recordId: '$reportsToId', fields: CONTACT_FIELDS})
  processRecursiveContacts({error, data}) {
    if (data) {
      this.addPersonToOrganization({
        Id: getFieldValue(data, CONTACT_ID_FIELD),
        Name: getFieldValue(data, CONTACT_NAME_FIELD),
        ReportsToId: getFieldValue(data, CONTACT_REPORTSTO_ID),
      });
      this.error = undefined;
    } else if (error) {
      this.handleError(error);
    }
  }

  addPersonToOrganization(person) {
    if (!this.organization.includes(person.Id)) {
      this.organization.unshift(person);
      this.reportsToId = person.ReportsToId;
    }
  }

  handleError(error) {
    this.error = `There was an error: ${error.body}`;
  }
}
```

### Step 3: Edit the `contactReportsTo.html`

Replace with

```html
<template>
  <template if:true={organization.length}>
    <div class="container">
      <template iterator:it={organization}>
        <span key={it.value.Id}>
          <c-contact-reports-to-person
            contact={it.value}
            is-first={it.first}
          ></c-contact-reports-to-person>
        </span>
      </template>
    </div>
  </template>
  <template if:false={organization.length}>
    No report to set
  </template>
  <template if:true={error}>
    There was an error!
  </template>
</template>
```

### Step 4: Edit the `contactReportsTo.js-meta.xml`

Replace xml with

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata" fqn="reportsTo">
    <apiVersion>47.0</apiVersion>
    <isExposed>true</isExposed>
    <masterLabel>Reporting Breadcrumb</masterLabel>
    <targets>
        <target>lightning__RecordPage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <objects>
                <object>Contact</object>
            </objects>
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

### Step 5: Create `contactReportsTo.svg`

Paste this SVG into the file

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" id="groups">
  <path
    d="M7.3 12.9c-.6-.9-.9-2.1-.9-3.3 0-2.1.8-3.9 2.2-4.9-.4-.9-1.4-1.5-2.6-1.5-2 0-3.1 1.7-3.1 3.6 0 1 .3 1.9 1 2.5.3.3.7.8.7 1.3s-.2.9-1.4 1.4c-1.6.7-3.2 1.8-3.2 3.3 0 1 .7 1.8 1.7 1.8h1.5c.2 0 .4-.2.6-.4.7-1.3 2.1-2.2 3.3-2.8.4-.1.5-.7.2-1zm13.5-.9c-1.1-.5-1.3-.9-1.3-1.4s.3-1 .7-1.3c.7-.7 1-1.5 1-2.5 0-1.9-1.1-3.6-3.2-3.6-1.2 0-2.1.6-2.6 1.5 1.4 1 2.2 2.8 2.2 4.9 0 1.2-.3 2.4-.9 3.3-.3.4-.1.9.2 1 1.2.6 2.6 1.5 3.3 2.8.2.2.4.4.6.4h1.5c1 0 1.7-.8 1.7-1.8 0-1.5-1.5-2.6-3.2-3.3zm-5.7 3.4c-1.3-.6-1.5-1.1-1.5-1.6 0-.6.4-1.1.8-1.4.7-.7 1.2-1.7 1.2-2.8 0-2.1-1.3-3.9-3.6-3.9S8.5 7.5 8.5 9.6c0 1.1.5 2.1 1.2 2.8.4.4.8.9.8 1.4 0 .6-.2 1-1.5 1.6-1.8.8-3.6 1.6-3.6 3.3 0 1.1.8 2 1.8 2h9.6c1.1 0 1.9-.9 1.9-2 0-1.6-1.8-2.5-3.6-3.3z"
  ></path>
</svg>
```

### Step 6: create `contactReportsTo.css`

Paste this CSS into the file

```css
c-contact-reports-to-person {
  padding-right: 5px;
  padding-left: 5px;
  display: flex;
  flex-direction: column;
}

.container {
  display: flex;
  flex-direction: row;
}
```

### Step 7: Create a New Component `reportsToPerson`

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name **`contactReportsToPerson`**
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
  ```
    sfdx force:lightning:component:create --type lwc -n contactReportsToPerson -d force-app/main/default/lwc
  ```

NOTE: [SFDX Command Line Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)

### Step 8: Edit `contactReportsToPerson.js`

Replace with

```javascript
import {LightningElement, api} from 'lwc';
import {NavigationMixin} from 'lightning/navigation';

export default class ReportsToPerson extends NavigationMixin(LightningElement) {
  @api contact;
  @api isFirst;

  navigateToContact() {
    this[NavigationMixin.Navigate]({
      type: 'standard__recordPage',
      attributes: {
        recordId: this.contact.Id,
        objectApiName: 'Contact',
        actionName: 'view',
      },
    });
  }
}
```

### Step 9: Edit `contactReportsToPerson.html`

Replace with

```html
<template>
  <div class="container">
    <template if:false={isFirst}
      ><span class="divider">&lt;</span></template
    >
    <c-contact-image
      record-id={contact.Id}
      image-size="Small"
    ></c-contact-image>
    <a href="#" onclick={navigateToContact}>{contact.Name}</a>
  </div>
</template>
```
