### E2E in Angular
2 ways to write e2e
1) selenium-webdriver
2) Angular's Protractor test framework

Protractor wraps the selenium-webdrive APIs to make async calls appear sync.
Also Protractor will wait for Angular to finish updating the page.
Protractor exposes some of the selenium-webdriver APIs via `browser.driver`.


```ts
// import { By, WebElement } from 'selenium-webdriver'
// selenium `By` is different from protractor `by`
import { browser, by, element } from 'protractor'
// describe, it omitted
browser.get('/')
expect(browser.getCurrentUrl()).toEqual(browser.baseUrl + '/')
```

### element, by
`element` and `by` almost always goes together.
`element` takes output of `by` and return Protractor `ElementFinder` object which is a wrapper of web elements.
Along with returning element wrapper(`ElementFinder`), `element` can be used to set a context for a by selector.
For example;
```
let loginElement = element(by.tagName('app-login'))
let inputSelector = by.tagName('input')
let inputWithinLoginElement = loginElement.element(inputSelector)
```
Above is possible because Protractor doesn't actaully find the element until there's a call that interacts with it.
i.e. calling `sendKeys()`, or `click()`

### Test scenario

```ts
describe('creating the new contact', () => {
  beforeAll(() => {
    browser.get('/')
  })
  
  it('should click add contact button', () => {
  
  })
  
  it('should write a name', () => {
  
  })
  
  it('should click the create button', () => {
  
  })
  
})

```

### Page Objects
Often tests break when someone changes id or class name.
We have to manually update all of our tests from a single class name change.

Page objects groups typical interactions.
id, class, or other identifiers can be arbitrary, i.e. not a good representation of the selected element in question.

Since Protractor do not actually find element when `element` function is called (finds only when interaction method is called), we can set selection in a constructor of page object.

```ts
import { browser, by, element, ElementFinder } from 'protractor';
import { ContactDetailPageObject } from './contact-detail.po'

export class ContactListPageObject {
  plusButton: ElementFinder;
 
  constructor() {
    // maybe this.selectors = {addContact: 'add-contact'}
    this.plusButton = element(by.id('add-contact'));
  }
 
  clickPlusButton() {
    this.plusButton.click();
    return new ContactDetailPageObject();
  }
 
  navigateTo() {
    browser.get('/');
  }
}
```

Instead of importing protractor functions we import page object
```ts
import { ContactListPageObject } from './po/contact-list.po.ts';
import { NewContactPageObject } from './po/new-contact.po.ts', , Contact }
 
describe('contact list', () => {
  let contactList: ContactListPageObject;
  let newContact: NewContactPageObject;
 
  beforeAll(() => {
    contactList = new ContactListPageObject();
  });
 
  describe('add a new contact', () => {
    beforeAll(() => {
      contactList.navigateTo();
    });
   
    it('should click the + button', () => {
      newContact = contactList.clickPlusButton();
      expect(newContact.getCurrentUrl()).toBe(browser.baseUrl + '/add');
    });
 
    it('should fill out form for a new contact', () => {
      newContact
        .setContactInfo('Mr. Newton', 'mr.newton@example.com', null);
      expect(newContact.getName()).toBe('Mr. Newton');
      expect(newContact.getEmail()).toBe('mr.newton@example.com');
      expect(newContact.getPhone()).toBe('');
    });
 
    it('should click the create button', () => {
      contactList = newContact.clickCreateButton();
      expect(contactList.getCurrentUrl()).toBe(browser.baseUrl + '/');
    });
  });
```

We could create BasePageObject to be used to extend other pageObjects to remove DRY codes.

Good Exmples;
https://github.com/testing-angular-applications/testing-angular-applications/tree/master/chapter08/e2e






























