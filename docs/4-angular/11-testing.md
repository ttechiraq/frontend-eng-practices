---
sidebar_position: 11
---

# Testing

Testing your Angular application helps ensure it works as expected. Unit tests are crucial for catching bugs early, ensuring code quality, and facilitating safe refactoring.

Source: [angular.dev/guide/testing](https://angular.dev/guide/testing)

## Vitest Setup (Default in Angular CLI)

New Angular CLI projects use the [Vitest](https://vitest.dev/) testing framework by default. Vitest runs your unit tests in a Node.js environment using `jsdom` for DOM simulation, providing faster test execution by avoiding the overhead of launching a browser.

### Running Tests

```bash
ng test
```

The `ng test` command builds the application in watch mode and launches the Vitest test runner. Example output:

```
 ✓ src/app/app.spec.ts (3)
   ✓ AppComponent should create the app
   ✓ AppComponent should have as title 'my-app'
   ✓ AppComponent should render title

 Test Files  1 passed (1)
      Tests  3 passed (3)
   Start at  18:18:01
   Duration  2.46s
```

### Configuration

Vitest configuration is handled by the CLI through `angular.json`:

| Option | Description |
|---|---|
| `browsers` | Run tests in a real browser (e.g., `["chromium"]`) |
| `coverage` | Enable code coverage reporting |
| `providersFile` | Path to global test providers |
| `setupFiles` | Global setup files (polyfills, mocks) |
| `include` | Glob patterns for test files (default: `['**/*.spec.ts', '**/*.test.ts']`) |
| `exclude` | Glob patterns for files to exclude |

### Global Test Providers

Create a providers file for shared test configuration:

```typescript
// src/test-providers.ts
import { Provider } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideHttpClientTesting } from '@angular/common/http/testing';

const testProviders: Provider[] = [
  provideHttpClient(),
  provideHttpClientTesting(),
];

export default testProviders;
```

Reference it in `angular.json`:

```json
{
  "architect": {
    "test": {
      "options": {
        "providersFile": "src/test-providers.ts"
      }
    }
  }
}
```

## Testing Components

### Basic Component Test

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { GreetingComponent } from './greeting.component';

describe('GreetingComponent', () => {
  let component: GreetingComponent;
  let fixture: ComponentFixture<GreetingComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [GreetingComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(GreetingComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display the greeting', () => {
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('h1')?.textContent).toContain('Hello');
  });
});
```

### Testing with Inputs

```typescript
it('should display the user name', () => {
  fixture.componentRef.setInput('name', 'Alice');
  fixture.detectChanges();

  const compiled = fixture.nativeElement as HTMLElement;
  expect(compiled.textContent).toContain('Alice');
});
```

### Testing Events

```typescript
it('should emit close event on button click', () => {
  const closeSpy = vi.fn();
  component.closed.subscribe(closeSpy);

  const button = fixture.nativeElement.querySelector('button');
  button.click();

  expect(closeSpy).toHaveBeenCalled();
});
```

Source: [angular.dev/guide/testing/components-basics](https://angular.dev/guide/testing/components-basics)

## Testing Services

### Simple Service Test

```typescript
import { TestBed } from '@angular/core/testing';
import { CalculatorService } from './calculator.service';

describe('CalculatorService', () => {
  let service: CalculatorService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(CalculatorService);
  });

  it('should add two numbers', () => {
    expect(service.add(2, 3)).toBe(5);
  });
});
```

### Service with Dependencies

Use test doubles to isolate the service under test:

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpTesting: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(),
        provideHttpClientTesting(),
      ],
    });

    service = TestBed.inject(UserService);
    httpTesting = TestBed.inject(HttpTestingController);
  });

  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'Alice' }];

    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });

    const req = httpTesting.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  afterEach(() => {
    httpTesting.verify();
  });
});
```

Source: [angular.dev/guide/testing/services](https://angular.dev/guide/testing/services)

## Testing Pipes

Pipes are the easiest Angular artifacts to test because they are pure functions:

```typescript
import { TruncatePipe } from './truncate.pipe';

describe('TruncatePipe', () => {
  const pipe = new TruncatePipe();

  it('should truncate long strings', () => {
    expect(pipe.transform('Hello World', 5)).toBe('Hello...');
  });

  it('should not truncate short strings', () => {
    expect(pipe.transform('Hi', 5)).toBe('Hi');
  });
});
```

Source: [angular.dev/guide/testing/pipes](https://angular.dev/guide/testing/pipes)

## Testing Directives

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { Component } from '@angular/core';
import { HighlightDirective } from './highlight.directive';

@Component({
  imports: [HighlightDirective],
  template: `<p appHighlight="yellow">Test</p>`,
})
class TestHostComponent {}

describe('HighlightDirective', () => {
  let fixture: ComponentFixture<TestHostComponent>;

  beforeEach(() => {
    fixture = TestBed.createComponent(TestHostComponent);
    fixture.detectChanges();
  });

  it('should highlight the element', () => {
    const p = fixture.nativeElement.querySelector('p');
    expect(p.style.backgroundColor).toBe('yellow');
  });
});
```

Source: [angular.dev/guide/testing/attribute-directives](https://angular.dev/guide/testing/attribute-directives)

## Component Harnesses

Component harnesses provide a stable testing API for interacting with components. They abstract away DOM details so tests are less brittle.

```typescript
import { HarnessLoader } from '@angular/cdk/testing';
import { TestbedHarnessEnvironment } from '@angular/cdk/testing/testbed';
import { MatButtonHarness } from '@angular/material/button/testing';

describe('MyComponent', () => {
  let loader: HarnessLoader;

  beforeEach(() => {
    const fixture = TestBed.createComponent(MyComponent);
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should click the submit button', async () => {
    const button = await loader.getHarness(MatButtonHarness.with({ text: 'Submit' }));
    await button.click();
    // Assert expected behavior
  });
});
```

Source: [angular.dev/guide/testing/component-harnesses-overview](https://angular.dev/guide/testing/component-harnesses-overview)

## Testing Routing

Use `RouterTestingHarness` for testing routed components:

```typescript
import { RouterTestingHarness } from '@angular/router/testing';
import { provideRouter } from '@angular/router';

describe('UserDetailComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        provideRouter([
          { path: 'users/:id', component: UserDetailComponent },
        ]),
      ],
    });
  });

  it('should display user details', async () => {
    const harness = await RouterTestingHarness.create();
    const component = await harness.navigateByUrl('/users/1', UserDetailComponent);
    harness.detectChanges();

    expect(component.userId()).toBe('1');
  });
});
```

Source: [angular.dev/guide/routing/testing](https://angular.dev/guide/routing/testing)

## Code Coverage

Generate a code coverage report by adding the `--coverage` flag:

```bash
ng test --coverage
```

The report is generated in the `coverage/` directory.

Source: [angular.dev/guide/testing/code-coverage](https://angular.dev/guide/testing/code-coverage)

## Running Tests in CI

Most CI servers set `CI=true`, which `ng test` detects to run in non-interactive, single-run mode. If your CI server does not set this variable:

```bash
ng test --no-watch --no-progress
```

### Running Tests in a Real Browser

Install a browser provider for Vitest browser mode:

```bash
# Playwright
npm install --save-dev @vitest/browser-playwright playwright

# Run tests in Chromium
ng test --browsers=chromium
```

Source: [angular.dev/guide/testing](https://angular.dev/guide/testing)
