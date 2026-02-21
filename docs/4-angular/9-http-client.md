---
sidebar_position: 9
---

# HTTP Client

Angular provides a client HTTP API for Angular applications — the `HttpClient` service class in `@angular/common/http`. It enables front-end applications to communicate with servers over the HTTP protocol.

Source: [angular.dev/guide/http](https://angular.dev/guide/http)

## Features

- Robust testing utilities
- Request and response interception
- Streamlined error handling
- The ability to request typed response values

## Setting Up HttpClient

Provide `HttpClient` in your application configuration:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
  ],
};
```

### Configuration Options

`provideHttpClient()` accepts optional features:

```typescript
import { provideHttpClient, withInterceptors, withFetch } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor]),
      withFetch(), // Use fetch API instead of XMLHttpRequest
    ),
  ],
};
```

Source: [angular.dev/guide/http/setup](https://angular.dev/guide/http/setup)

## Making Requests

Inject `HttpClient` into a service or component:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = '/api/users';

  // GET request — fetch data
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  // GET with params
  searchUsers(query: string): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl, {
      params: { q: query },
    });
  }

  // POST request — create data
  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  // PUT request — replace data
  updateUser(id: number, user: UpdateUserDto): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  // PATCH request — partial update
  patchUser(id: number, changes: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, changes);
  }

  // DELETE request
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

### Setting Headers

```typescript
import { HttpHeaders } from '@angular/common/http';

const headers = new HttpHeaders({
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${token}`,
});

this.http.get<User[]>('/api/users', { headers });
```

### Requesting Non-JSON Data

```typescript
// Request text response
this.http.get('/api/health', { responseType: 'text' });

// Request blob (binary data)
this.http.get('/api/files/download', { responseType: 'blob' });

// Access the full response (including headers and status)
this.http.get<User[]>('/api/users', { observe: 'response' }).subscribe(response => {
  console.log(response.status);
  console.log(response.headers.get('X-Total-Count'));
  console.log(response.body);
});
```

Source: [angular.dev/guide/http/making-requests](https://angular.dev/guide/http/making-requests)

## Error Handling

```typescript
import { catchError, throwError } from 'rxjs';

getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users').pipe(
    catchError(error => {
      if (error.status === 404) {
        console.error('Resource not found');
        return of([]);
      }
      if (error.status === 0) {
        console.error('Network error — check connectivity');
      }
      return throwError(() => new Error('Something went wrong'));
    }),
  );
}
```

### Retry on Failure

```typescript
import { retry } from 'rxjs';

getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users').pipe(
    retry(3), // Retry up to 3 times
  );
}
```

## Interceptors

Interceptors examine and transform HTTP requests and responses. Angular 19 uses functional interceptors:

```typescript
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const cloned = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` },
    });
    return next(cloned);
  }

  return next(req);
};
```

### Logging Interceptor

```typescript
import { tap } from 'rxjs';

export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const started = performance.now();
  return next(req).pipe(
    tap({
      next: () => {
        const elapsed = performance.now() - started;
        console.log(`${req.method} ${req.urlWithParams} completed in ${elapsed}ms`);
      },
      error: (error) => {
        console.error(`${req.method} ${req.urlWithParams} failed:`, error);
      },
    }),
  );
};
```

### Registering Interceptors

```typescript
provideHttpClient(
  withInterceptors([authInterceptor, loggingInterceptor]),
)
```

Interceptors run in the order they are listed. Request interceptors run first-to-last; response interceptors run last-to-first.

Source: [angular.dev/guide/http/interceptors](https://angular.dev/guide/http/interceptors)

## Reactive Data Fetching with httpResource

`httpResource` integrates `HttpClient` with Angular Signals for reactive data fetching:

```typescript
import { httpResource } from '@angular/common/http';
import { signal } from '@angular/core';

@Component({ ... })
export class UserListComponent {
  searchQuery = signal('');

  usersResource = httpResource<User[]>({
    url: '/api/users',
    params: () => ({ q: this.searchQuery() }),
  });

  // Access in template
  // usersResource.value()    — the data
  // usersResource.status()   — loading state
  // usersResource.error()    — error if any
}
```

`httpResource` automatically re-fetches when its reactive parameters change.

Source: [angular.dev/guide/http/http-resource](https://angular.dev/guide/http/http-resource)

## Testing HTTP Requests

Angular provides `HttpTestingController` for testing HTTP interactions:

```typescript
import { provideHttpClient } from '@angular/common/http';
import { provideHttpClientTesting, HttpTestingController } from '@angular/common/http/testing';
import { TestBed } from '@angular/core/testing';

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
    const mockUsers: User[] = [{ id: 1, name: 'Alice' }];

    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });

    const req = httpTesting.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  afterEach(() => {
    httpTesting.verify(); // Ensure no unexpected requests
  });
});
```

Source: [angular.dev/guide/http/testing](https://angular.dev/guide/http/testing)
