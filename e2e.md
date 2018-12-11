### E2E in Angular
2 ways to write e2e
1) selenium-webdriver
2) Angular's Protractor test framework

Protractor wraps the selenium-webdrive APIs to make async calls appear sync.

```
// `element` takes output of `by` and return Protractor `ElementFinder` object which is a wrapper of web elements
import { browser, by, element } from 'protractor'
// describe, it omitted
browser.get('/')
expect(browser.getCurrentUrl()).toEqual(browser.baseUrl + '/')
```

### Test scenario

```
describe('creating a new contact', () => {
  it('should find the add contact button', () => {
  
  })
})

```
