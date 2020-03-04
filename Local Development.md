
# Pre-reqs
1. [Install latest Node](https://nodejs.org/en/download/)
1. [Install Git](https://git-scm.com/downloads) 
1. [Setup Dev Environment](https://trailhead.salesforce.com/en/content/learn/projects/quickstart-vscode-salesforce/vscode-salesforce-ready)

# Setup the beta plugin:
Command `sfdx plugins:install @salesforce/lwc-dev-server`

# Syncronize LWC-Recipes
`git clone https://github.com/trailheadapps/lwc-recipes.git`

# Launch VS Code
Goto diectory: `cd lwc-recipes`
launch VS Code: `code .`

# Setup local project
Open Terminal `CTRL + ~`
Install local deps from NPM `npm i` or `yarn`

# Setup SFDX
`SFDX: Authorize a Dev Hub`

# Create a Scratch org
`SFDX: Create a Default Scratch Org...`

# Push the code
CMD + Shift + P: `SFDX: Push Source to Default Scratch Org`
Terminal Command: `sfdx force:source:push`

# Assign the **recipes** permission set to the default user:
Terminal Command: `sfdx force:user:permset:assign -n recipes`

# Load sample data
Terminal Command: `sfdx force:data:tree:import --plan ./data/data-plan.json`

# Start the dev server
Command `sfdx force:lightning:lwc:start`

# Make a quick "record page"
`SFDX: Create New Lightning Web Component`


``` javascript
import {LightningElement, api, wire, track} from 'lwc';
import {getRecord, getFieldValue} from 'lightning/uiRecordApi';
import CONTACT_NAME from '@salesforce/schema/Contact.Name';
import CONTACT_PICTURE from '@salesforce/schema/Contact.Picture__c';
import CONTACT_TITLE from '@salesforce/schema/Contact.Title';
import CONTACT_PHONE from '@salesforce/schema/Contact.Phone';

const FIELDS = [CONTACT_NAME, CONTACT_PHONE, CONTACT_PICTURE, CONTACT_TITLE];

export default class RecordPage extends LightningElement {
  @api recordId = '0036300000YDRSUAA5';
  @track contact = {};

  @wire (getRecord, {
    recordId: '$recordId',
    fields: FIELDS,
  })
  wiredData({error, data}) {
    if (error) {
      //TODO :error message
    } else if (data) {
      this.contact.Name = getFieldValue (data, CONTACT_NAME);
      this.contact.Phone = getFieldValue (data, CONTACT_PHONE);
      this.contact.Title = getFieldValue (data, CONTACT_TITLE);
      this.contact.Picture__c = getFieldValue (data, CONTACT_PICTURE);
    }
  }
}
```

``` html
<template>
    <c-contact-tile contact={contact}></c-contact-tile>
</template>
```

# More complex record page
``` html
<template>
    <template if:true={recordId}>
        <select id="recordIds" onchange={handleRecordChange}>
            <template for:each={people} for:item="Person">
                <option key={Person.Id} value={Person.Id}>{Person.Name}</option>
            </template>

        </select>

        <div class="break"></div>

        <c-contact-image-apex image-size={imageSize} record-id={recordId}></c-contact-image-apex>

        <select id="imageSize" onchange={handleSizeChange}>
            <option value="Small">Small</option>
            <option value="Medium">Medium</option>
            <option value="Large">Large</option>
            <option value="Massive">Massive</option>
        </select>

        <div class="break"></div>

        <c-contact-tile contact={contact}></c-contact-tile>

        <div class="break"></div>

        <lightning-record-edit-form record-id={recordId} object-api-name="Contact">
            <lightning-messages>
            </lightning-messages>
            <lightning-input-field field-name="Name">
            </lightning-input-field>
            <lightning-input-field field-name="Title">
            </lightning-input-field>
            <lightning-input-field field-name="Phone">
            </lightning-input-field>
            <lightning-button class="slds-m-top_small" variant="brand" type="submit" name="update" label="Update">
            </lightning-button>
        </lightning-record-edit-form>
    </template>
</template>
```
# Add CSS

``` css
.break{
    width: 100%;
    height: 1px;
    background-color: red;
    margin-top: 15px;
    margin-bottom: 15px;
}
```