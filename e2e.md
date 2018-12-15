# E2E in Angular
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
However, I believe e2e test is procedural. every line of code represents user's behaviour in some way.  
eg)
```ts
// Selecting of an element can represent user seeing the element.
const $usernameInput = $('.login-username-input');

// Typing is represented with sendKeys method.
$usernameInput.sendKeys('imskojs');

// Where as by using PageObject procedure is aggregated. Which reduces code but at a cost of not aggregating procedures
const loginPage = new LoginPageObject()
loginPage.login()
```

### Expected Conditions `EC`

#### `EC` facts
* `EC` can be assigned to variables.
* `EC` holds reference to the `browser` object.
* Because of above reason get the new reference to `EC` after restarting or new-windowing the browser.
* Because of above reason use `browser.ExpectedConditions` rather than `protractor.ExpectedConditions`.
* `EC` is a function that returns a `Promise` that returns `Boolean` when resolved.



### `browser.wait`'s custom conditions (other than `EC`)
`browser.wait` will intermittently call the call back function until it returns true.
This call back function can return a promise. *Q: calls promise returning call back repeatedly? how does that work?*
I guess when the callback returns a promise `browser.wait` will wait for the returned promise to resolve.  

#### browser.executeScript
`WebDriver` gets the DOM element from our app using `browser.driver.executeScript` which returns DOM wrapped class `WebElement` class, which protractor wraps to give us `ElementFinder` instance which we use to call methods on such as `sendKeys()`.  
We can get [WebElement](https://seleniumhq.github.io/selenium/docs/api/java/org/openqa/selenium/WebElement.html) from 
[ElementFinder](http://www.protractortest.org/#/api?view=ElementFinder) using `ElementFinder`'s `getWebElement`.

We could use `browser.driver.executeScript` manually;
```ts
function $getOrderBook() {
  return document.querySelectorAll('.order-book');
}
browser.wait(() => {
  // $getOrderBook is run in the browser context where our real web app is running.
  return browser.dirver.executeScript($getOrderBook)
    .then((orderBooks: WebElement[]) => {
      return orderBooks.length >= 2;
    })
})
```

### Angular Zone
Protractor waits until there are no more tasks pending in the Angular Zone.
So if you poll http requests using setInterval or recursive setTimeout, they will constantly creates tasks in Angular Zone.
As Protractor waits for the pending tasks in Angular Zone, it will wait forever for interval to be cleared.
We can put this polling logic outside of angular zone by;
`ngZone.runOutsideAngular(() => { /* logic here */ })`

So for Example if we have;
```ts
// all imports omitted
@Component({
  // omitted
});
class SomeComponent implements OnInit {
  ngOnInit() {
    interval(500).pipe(
      // Some logic
    ).subscribe(x => this.x = x)
  }
}
```
Protractor will wait forever.
But if we change the code to;
```ts
// all imports omitted
@Component({
  // omitted
});
class SomeComponent implements OnInit {
  
  constructor(private zone: NgZone) {}
  
  ngOnInit() {
    this.zone.runOutsideAngular(() => {
      interval(500).pipe(
        // Some logic
      ).subscribe(x => {
        this.zone.run(() => {
          this.x = x;
        })
      })
    })    
  }
}
```
Above code will run `interval` outside the angular zone but when we get the result we will run it in a zone `zone.run`.
This way `interval` resides in browser eventloop and not angular zone.

### WebDriver Control Flow
Protractor's methods seems to run as though they are syncronous. It was made possible with managed promise.
managed promise don't run async task, it adds tasks to run on Webdriver's control flow.
control flow runs all the added tasks at the end of `it()` block.

*NOTE: managed promise is a superset of normal promise, meaning we can either use it like sync or treat it like a normal promise*  

All these work to make the code look syncronous. But there are caveats such as;  
* Use of control flow prevents us from debugging Protractor tests with standard Node.js tools.

#### We should use `async/await`
We can disable WebDriver Control Flow by adding;  
`SELENIUM_PROMISE_MANAGER: false` to our Protractor config.  
*NOTE: Selenium >= 4.x removes control flow permanently, so above option is the only option*  
Now with our new knowledge the code would look something like;
```ts
import { browser, by, element } from 'protractor'
describe('Buying Page', async () => {
  
  const buyButton = $('.buy-button');
  const listContainer = $('list-container');
  
  await browser.get('/');
  await buyButton.click();
  const textInContainer = await listContainer.getText();
  
  expect(await browser.getCurrentUrl()).toEqual(browser.baseUrl + '/')
  expect(textInContainer).toContain('1000원');
  
});
```

More over we can use Chrome DevTools to debug our e2e test!



Good Exmples;  
https://github.com/testing-angular-applications/testing-angular-applications/tree/master/chapter08/e2e






























