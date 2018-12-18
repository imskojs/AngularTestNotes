# Unit Test in Angular

### Isolated tests
Don't need any Angular dependencies. Used to ensure that the component is created correctly.
Usually testing ordinary JavaScript Class.

### Shallow tests
Testing parent Component with dependencies injected.
No test for nested child components.

Dependencies that a parent component may use include;
1) Testing module dependencies
2) Pure Angular dependencies
3) Project specific dependencies

### Testing module dependencies
```ts
import {TestBed, async, fakeAsync, ComponentFixture, tick} from '@angular/core/testing'
import { BrowserDynamicTestingModule } from '@angular/platform-browser-dynaminc/testing'
import { RouterTestingModule } from '@angular/router/testing'
```

`async` waits for all asynchronous calls within the Zone. Tests will not execute until calls within the asynchronous test zone have been completed.

`ComponentFixture` creates fixture(fixed state of a set of objects used as a baseline for running tests)

`TestBed` container that connects dependencies.

`fakeAsync` run asynchronous functions as if they were synchronous.  
`tick` is used with `fakeAsync` to forward in milliseconds.

`BrowserDynamicTestingModule` bootstrap the browser to be used for testing.

`RouterTestingModule` set up routing for testing.


### Pure Angular dependencies
```ts
import { FormsModule } from '@angular/forms'
import { DebugElement } from '@angular/core'
import { By } from '@angular/platform-browser'
import { NoopAnimationsModule } from '@angular/platform-browser/animation'
```
These modules and utilities can be used in Angular projects as well as for writing tests.

### Project specific dependencies
```ts
import { MessageModel, UtilityService, SharedCompoentOne } from '../shared'
```
Any project specific dependencies of the component we want to test should be injected.
*Q: 3rd party libraries?*

### fakes
In test, "fake" can be thought of as an object that contains helper methods and simulates real data.   
1) A mock is a fake that simulates the real object and it tracks when it's called and what arguments it receives.
2) A stub is a fake that always returns the same value.

We use fake to eliminate failure of tests caused not by the component we are testing but by the dependencies that
the component uses.  
This implies we should make the test for the dependencies the component uses before testing the component.  
For example, we should test `UserService` before testing `UserComponent` since we will not be using the real `UserService`
but rather will fake it (with a mock).

When creating a fake it is important that the fake object has the same type as the real one. (Typescript helps here)


### Set up gotchas
1) When configuring `TestBed.configureTestingModule`, only use Modules, Components, Providers that the component we are testing uses.
2) Use mocks for Services with `providers[{provide: UserService, useValue: userServiceStub}]`
3) Any component that gets loaded lazily should be included in `TestBed.overrideModule(BrowserDynamicTestingModule, { ... })`






