# AngularTestNotes
Notes from learning Angular tests


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
```
import {TestBed, async, fakeAsync, ComponentFixture, tick} from '@angular/core/testing
import { BrowserDynamicTestingModule } from '@angular/platform-browser-dynaminc/testing'
import { RouterTestingModule } from '@angular/router/testing'
```

`async` waits for all asynchronous calls within the Zone.

`ComponentFixture` creates fixture(fixed state of a set of objects used as a baseline for running tests)

`TestBed` container that connects dependencies.

`fakeAsync` run asynchronous functions as if they were synchronous.  
`tick` is used with `fakeAsync` to forward in milliseconds.

`BrowserDynamicTestingModule` bootstrap the browser to be used for testing.

`RouterTestingModule` set up routing for testing.


### Pure Angular dependencies
```
import { DebugElement } from '@angular/core'
import { By } from '@angular/platform-browser'
import { NoopAnimationsModule } from '@angular/platform-browser/animation'
```
These utilities can be used in Angular projects as well as for writing tests.

### Project specific dependencies










