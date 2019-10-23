### Step 1: Create the component `contactsSimilar`

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name **`contactsSimilar`**
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
  ```
    sfdx force:lightning:component:create --type lwc -n contactsSimilar -d force-app/main/default/lwc
  ```

NOTE: [SFDX Command Line Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)

### Step 2: Add APEX method to `ContactsController.cls`

Add this method:

```javascript
    @AuraEnabled(cacheable=true)
    public static List<Contact> getSimilarContacts(String accountId, String recordId) {
        List<Contact> contacts = [
            SELECT Id, Name, Title, Phone, Email, Picture__c
            FROM Contact
            WHERE AccountId = :accountId
            AND Id != :recordId
        ];
        return contacts;
    }
```

### Step 3: update imports in `contactsSimilar.js`

Add this code

```javascript
import {LightningElement, wire, api, track} from 'lwc';
import getSimilarContacts from '@salesforce/apex/ContactController.getSimilarContacts';
import {getRecord, getFieldValue} from 'lightning/uiRecordApi';

import CONTACT_ACCOUNT_FIELD from '@salesforce/schema/Contact.AccountId';
import CONTACT_ID_FIELD from '@salesforce/schema/Contact.Id';
import CONTACT_NAME_FIELD from '@salesforce/schema/Contact.Name';
import CONTACT_IMAGE_FIELD from '@salesforce/schema/Contact.Picture__c';
import CONTACT_REPORTSTO_ID from '@salesforce/schema/Contact.ReportsToId';
import CONTACT_EMAIL_FIELD from '@salesforce/schema/Contact.Email';

const CONTACT_FIELDS = [
  CONTACT_ACCOUNT_FIELD,
  CONTACT_REPORTSTO_ID,
  CONTACT_ID_FIELD,
  CONTACT_NAME_FIELD,
  CONTACT_IMAGE_FIELD,
  CONTACT_EMAIL_FIELD,
];

export default class ContactsSimilar extends LightningElement {
  @api recordId;
  @track accountId;
  @track similarContacts;
  @track contactsList;

  @wire(getRecord, {recordId: '$recordId', fields: CONTACT_FIELDS})
  currentUser({error, data}) {
    if (data) {
      this.accountId = getFieldValue(data, CONTACT_ACCOUNT_FIELD);
    } else if (error) {
      window.console.log('Error: ' + error);
    }
  }

  @wire(getSimilarContacts, {
    accountId: '$accountId',
    recordId: '$recordId',
  })
  getSimilarContacts({error, data}) {
    if (data) {
      this.similarContacts = data;
    } else if (error) {
      window.console.log('Error: ' + error);
    }
  }

  get Name() {
    return getFieldValue(this.contactsList, CONTACT_NAME_FIELD);
  }
}
```

### Step 4: Edit the `contactSimilar.html`

Add this code:

```html
<template>
  <lightning-card title="Similar Contacts">
    <div class="slds-m-around_medium">
      <ul class="slds-list_vertical slds-has-dividers_top-space">
        <template
          for:each="{similarContacts}"
          for:item="contact"
          if:true="{similarContacts}"
        >
          <li key="{contact.Id}" class="slds-list__item">
            <c-contacts-card contact="{contact}"></c-contacts-card>
          </li>
        </template>
      </ul>
    </div>
  </lightning-card>
</template>
```

### Step 5: Edit the `contactsSimilar.js-meta.xml`

Add this code

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata" fqn="contacstSimilar">
    <apiVersion>47.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
    </targets>
</LightningComponentBundle>
```

### Step 6: Create the component `contactCard`

- Create a new component:
  - Visual Studio Code:
    - `CMD + SHIFT + P` (Mac OS)
    - `CNTRL + SHIFT + P` (Windows)
      - Select: `SFDX: Create Lightning Web Component`
      - Enter the name **`contactCard`**
      - Accept the default directory `force-app/main/default/lwc`
  - Command Line:
  ```
    sfdx force:lightning:component:create --type lwc -n contactCard -d force-app/main/default/lwc
  ```

### Step 6: Edit the `contactCard.js`

Add this code

```javascript
import {LightningElement, track, api} from 'lwc';
import {ShowToastEvent} from 'lightning/platformShowToastEvent';

export default class ContactsCard extends LightningElement {
  @track editMode = false;
  @api contact;

  handleEditClick() {
    this.editMode = true;
  }

  handleEditSuccess() {
    const evt = new ShowToastEvent({
      title: 'Success!',
      message: 'The record has been successfully saved.',
      variant: 'success',
    });
    this.dispatchEvent(evt);
    this.editMode = false;
  }

  handleEditError() {
    const evt = new ShowToastEvent({
      title: 'Error!',
      message: 'An error occurred while attempting to save the record.',
      variant: 'error',
    });
    this.dispatchEvent(evt);
    this.editMode = false;
  }

  handleEditCancel(event) {
    event.preventDefault();
    this.editMode = false;
  }
}
```

### Step 7: Edit `contactCard.html`

Add this code:

```html
<template>
  <!-- Use SLDS Grid: https://www.lightningdesignsystem.com/utilities/grid/ -->
  <div class="slds-grid slds-hint-parent">
    <!-- TODO: Add Contact Image -->

    <!-- View Form -->
    <lightning-record-view-form
      object-api-name="Contact"
      record-id="{contact.Id}"
    >
      <lightning-layout multiple-rows>
        <lightning-layout-item size="6">
          <lightning-output-field
            field-name="FirstName"
          ></lightning-output-field>
        </lightning-layout-item>
        <lightning-layout-item size="6">
          <lightning-output-field
            field-name="LastName"
          ></lightning-output-field>
        </lightning-layout-item>
        <lightning-layout-item size="6">
          <lightning-output-field
            field-name="ReportsToId"
          ></lightning-output-field>
        </lightning-layout-item>
        <lightning-layout-item size="6">
          <lightning-output-field field-name="Email"></lightning-output-field>
        </lightning-layout-item>
      </lightning-layout>
    </lightning-record-view-form>
    <!-- TODO: Add Edit Form -->

    <!-- TODO: Handle Edit States in edit Form-->

    <!-- TODO: Add Edit Button -->
  </div>
</template>
```

### Step 7: allow edit mode

Replace with this code:

```html
<template>
  <!-- Use SLDS Grid: https://www.lightningdesignsystem.com/utilities/grid/ -->
  <div class="slds-grid slds-hint-parent">
    <!-- TODO: Add Contact Image -->

    <!-- View Form -->
    <template if:false="{editMode}">
      <lightning-record-view-form
        object-api-name="Contact"
        record-id="{contact.Id}"
      >
        <lightning-layout multiple-rows>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="FirstName"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="LastName"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="ReportsToId"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field field-name="Email"></lightning-output-field>
          </lightning-layout-item>
        </lightning-layout>
      </lightning-record-view-form>
    </template>
    <!-- *NEW* Edit Form -->
    <template if:true="{editMode}">
      <lightning-record-edit-form
        object-api-name="Contact"
        record-id="{contact.Id}"
        onsuccess="{handleEditSuccess}"
        onerror="{handleEditError}"
      >
        <lightning-layout multiple-rows>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="FirstName"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="LastName"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="ReportsToId"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field field-name="Email"></lightning-input-field>
          </lightning-layout-item>
        </lightning-layout>
        <!-- *NEW* Handle Edit States -->
        <lightning-layout-item size="12">
          <div class="slds-m-top_large slds-grid slds-grid_align-center">
            <lightning-button
              variant="neutral"
              label="Cancel"
              title="Cancel"
              type="text"
              onclick="{handleEditCancel}"
              class="slds-m-right_small"
            ></lightning-button>
            <lightning-button
              variant="brand"
              label="Submit"
              title="Submit"
              type="submit"
            >
            </lightning-button>
          </div>
        </lightning-layout-item>
      </lightning-record-edit-form>
    </template>
    <!-- *NEW* Edit Button -->
    <template if:false="{editMode}">
      <lightning-button-icon
        icon-name="utility:edit"
        class="slds-col_bump-left"
        icon-class="slds-button__icon_hint"
        variant="bare"
        alternative-text="Edit Record"
        onclick="{handleEditClick}"
      ></lightning-button-icon>
    </template>
  </div>
</template>
```

### Step 8: Final UI!

Replace with this HTML

```html
<template>
  <!-- Use SLDS Grid: https://www.lightningdesignsystem.com/utilities/grid/ -->
  <div class="slds-grid slds-hint-parent">
    <!-- *NEW* Contact Image -->
    <div class="slds-media__figure">
      <c-contact-image
        record-id="{contact.Id}"
        image-size="Large"
      ></c-contact-image>
    </div>
    <!-- View Form -->
    <template if:false="{editMode}">
      <lightning-record-view-form
        object-api-name="Contact"
        record-id="{contact.Id}"
      >
        <lightning-layout multiple-rows>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="FirstName"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="LastName"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field
              field-name="ReportsToId"
            ></lightning-output-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-output-field field-name="Email"></lightning-output-field>
          </lightning-layout-item>
        </lightning-layout>
      </lightning-record-view-form>
    </template>
    <!-- Edit Form -->
    <template if:true="{editMode}">
      <lightning-record-edit-form
        object-api-name="Contact"
        record-id="{contact.Id}"
        onsuccess="{handleEditSuccess}"
        onerror="{handleEditError}"
      >
        <lightning-layout multiple-rows>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="FirstName"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="LastName"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field
              field-name="ReportsToId"
            ></lightning-input-field>
          </lightning-layout-item>
          <lightning-layout-item size="6">
            <lightning-input-field field-name="Email"></lightning-input-field>
          </lightning-layout-item>
        </lightning-layout>
        <!-- Handle Edit States -->
        <lightning-layout-item size="12">
          <div class="slds-m-top_large slds-grid slds-grid_align-center">
            <lightning-button
              variant="neutral"
              label="Cancel"
              title="Cancel"
              type="text"
              onclick="{handleEditCancel}"
              class="slds-m-right_small"
            ></lightning-button>
            <lightning-button
              variant="brand"
              label="Submit"
              title="Submit"
              type="submit"
            >
            </lightning-button>
          </div>
        </lightning-layout-item>
      </lightning-record-edit-form>
    </template>
    <!-- Edit Button -->
    <template if:false="{editMode}">
      <lightning-button-icon
        icon-name="utility:edit"
        class="slds-col_bump-left"
        icon-class="slds-button__icon_hint"
        variant="bare"
        alternative-text="Edit Record"
        onclick="{handleEditClick}"
      ></lightning-button-icon>
    </template>
  </div>
</template>
```
