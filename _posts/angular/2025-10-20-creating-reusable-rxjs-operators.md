---
title: Creating Reusable RxJS Operators
date: 2025-10-20 12:00:00 +0900
comments: true
categories: angular
tags: [rxjs]
---

## Creating Reusable RxJS Operators for Angular Applications

When working with state management in RxJS, we often find ourselves using the same operator combinations repeatedly. In Angular projects particularly, patterns like loading state management, API error handling, and form validation keep appearing over and over.

In this article, we'll explore how to create custom RxJS operators to cleanly abstract these repetitive patterns. Using Angular 19's standalone components and new control flow syntax, we'll build practical operators you can immediately use in your projects.

## The Problem: Repetitive RxJS Patterns

When developing Angular applications, you'll frequently encounter code like this:

```tsx
// Repetitive pattern for managing loading states
this.loading = true;
this.http.get('/api/data').pipe(
  tap(() => this.loading = false),
  catchError(err => {
    this.loading = false;
    return throwError(() => err);
  })
).subscribe();
```

Here is another codes with rxJS
```tsx
// Search with form validation and debouncing
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  filter(value => value.length > 2),
  switchMap(value => this.searchService.search(value))
).subscribe();
```

By turning these patterns into custom operators, we can make our code more readable and reusable.

## Basic Structure of RxJS Custom Operators

An RxJS operator is simply a function that takes an Observable and returns a new Observable. With TypeScript's type system, we can ensure type safety as well.

### 1. Basic Operator: Identity

Let's start with the simplest form:

```tsx
import { Observable } from 'rxjs';

// Identity operator that returns the input stream as-is
export function identity<T>(): (source: Observable<T>) => Observable<T> {
  return (source: Observable<T>) => source;
}
```

### 2. Loading State Management Operator

Now let's create a practical operator that automatically manages loading states during API calls:

```tsx
import { Observable, MonoTypeOperatorFunction, defer, finalize } from 'rxjs';
import { signal, WritableSignal } from '@angular/core';

export function withLoading<T>(
  loadingSignal: WritableSignal<boolean>
): MonoTypeOperatorFunction<T> {
  return (source: Observable<T>) =>
    defer(() => {
      loadingSignal.set(true);
      return source.pipe(
        finalize(() => loadingSignal.set(false))
      );
    });
}
```

Usage Example - Standalone Component:

```tsx
import { Component, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { withLoading } from './operators/with-loading';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [],
  template: `
    <div class="user-container">
      @if (loading()) {
        <div class="spinner">Loading...</div>
      } @else {
        <ul>
          @for (user of users(); track user.id) {
            <li>{{ user.name }}</li>
          }
        </ul>
      }

      <button (click)="loadUsers()">Load Users</button>
    </div>
  `
})
export class UserListComponent {
  loading = signal(false);
  users = signal<User[]>([]);

  constructor(private http: HttpClient) {}

  loadUsers() {
    this.http.get<User[]>('/api/users').pipe(
      withLoading(this.loading)// 👈 Automatic loading state management
    ).subscribe(data => this.users.set(data));
  }
}
```

### 3. Error Handling and Retry Operator

Let's create an operator that adds error handling and retry logic to API calls:

```tsx
import { Observable, throwError, timer } from 'rxjs';
import { retry, catchError, tap } from 'rxjs/operators';

interface RetryConfig {
  count: number;
  delay: number;
  onError?: (error: any, attempt: number) => void;
}

export function retryWithDelay<T>(config: RetryConfig) {
  return (source: Observable<T>) => {
    let attempt = 0;

    return source.pipe(
      retry({
        count: config.count,
        delay: (error) => {
          attempt++;
          config.onError?.(error, attempt);
          return timer(config.delay * attempt);// Progressive backoff
        }
      }),
      catchError(error => {
        console.error(`Failed: Still failed after ${attempt} retries`, error);
        return throwError(() => error);
      })
    );
  };
}
```

### 4. Search Input Optimization Operator

Let's create an operator that optimizes search inputs:

```tsx
import { Observable, pipe } from 'rxjs';
import { debounceTime, distinctUntilChanged, filter, map } from 'rxjs/operators';

export function optimizeSearch(
  debounce: number = 300,
  minLength: number = 2
) {
  return pipe(
    debounceTime(debounce),
    map(value => value?.trim() || ''),
    distinctUntilChanged(),
    filter(value => value.length >= minLength)
  );
}
```

**Usage Example - Search Component:**

```tsx
@Component({
  selector: 'app-search',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <div class="search-container">
      <input
        type="search"
        [formControl]="searchControl"
        placeholder="Enter search term (min 2 characters)"
      />

      @if (searching()) {
        <div class="searching">Searching...</div>
      }

      <div class="results">
        @for (result of searchResults(); track result.id) {
          <div class="result-item">
            <h3>{{ result.title }}</h3>
            <p>{{ result.description }}</p>
          </div>
        } @empty {
          @if (searchControl.value && !searching()) {
            <p>No results found.</p>
          }
        }
      </div>
    </div>
  `
})
export class SearchComponent {
  searchControl = new FormControl('');
  searchResults = signal<SearchResult[]>([]);
  searching = signal(false);

  constructor(private searchService: SearchService) {
    this.searchControl.valueChanges.pipe(
      optimizeSearch(300, 2),// 👈 Search optimization
      tap(() => this.searching.set(true)),
      switchMap(term =>
        this.searchService.search(term).pipe(
          withLoading(this.searching)// 👈 Loading state management
        )
      )
    ).subscribe(results => this.searchResults.set(results));
  }
}
```

### 5. Caching Operator

Finally, let's create an operator that caches results:

```tsx
import { Observable, of, tap } from 'rxjs';
import { shareReplay } from 'rxjs/operators';

interface CacheConfig {
  ttl?: number;// Time to live in milliseconds
  key?: string;
}

const cache = new Map<string, { data: any; timestamp: number }>();

export function withCache<T>(config: CacheConfig = {}) {
  const { ttl = 5 * 60 * 1000, key = 'default' } = config;// Default 5 minutes

  return (source: Observable<T>) => {
    const cached = cache.get(key);
    const now = Date.now();

    if (cached && now - cached.timestamp < ttl) {
      return of(cached.data as T);
    }

    return source.pipe(
      tap(data => cache.set(key, { data, timestamp: now })),
      shareReplay(1)
    );
  };
}
```

## Putting It All Together: Real-World Example

Now let's combine all our operators to create a component you might use in a real application:

```tsx
@Component({
  selector: 'app-product-dashboard',
  standalone: true,
  imports: [ReactiveFormsModule, CurrencyPipe],
  template: `
    <div class="dashboard">
      <header>
        <h1>Product Dashboard</h1>
        <input
          type="search"
          [formControl]="searchControl"
          placeholder="Search products..."
        />
      </header>

      @if (loading()) {
        <div class="loader">Loading data...</div>
      }

      <div class="products">
        @for (product of filteredProducts(); track product.id) {
          <div class="product-card">
            <h3>{{ product.name }}</h3>
            <p>{{ product.price | currency:'USD' }}</p>
          </div>
        } @empty {
          @if (!loading()) {
            <p>No products found.</p>
          }
        }
      </div>

      @if (error()) {
        <div class="error">
          An error occurred: {{ error() }}
          <button (click)="reload()">Try Again</button>
        </div>
      }
    </div>
  `
})
export class ProductDashboardComponent implements OnInit {
  searchControl = new FormControl('');
  products = signal<Product[]>([]);
  filteredProducts = signal<Product[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.loadProducts();
    this.setupSearch();
  }

  loadProducts() {
    this.error.set(null);

    this.http.get<Product[]>('/api/products').pipe(
      withCache({ key: 'products', ttl: 10 * 60 * 1000 }),// 10-minute caching
      withLoading(this.loading),
      retryWithDelay({
        count: 3,
        delay: 1000,
        onError: (err, attempt) => console.log(`Retry ${attempt}/3`)
      }),
      catchError(err => {
        this.error.set(err.message);
        return of([]);
      })
    ).subscribe(products => {
      this.products.set(products);
      this.filteredProducts.set(products);
    });
  }

  setupSearch() {
    this.searchControl.valueChanges.pipe(
      optimizeSearch(300, 2)
    ).subscribe(searchTerm => {
      const filtered = this.products().filter(product =>
        product.name.toLowerCase().includes(searchTerm.toLowerCase())
      );
      this.filteredProducts.set(filtered);
    });
  }

  reload() {
    this.loadProducts();
  }
}
```

## Conclusion

Custom RxJS operators allow you to encapsulate complex asynchronous logic into reusable units. When combined with Angular's new Signal API, reactive programming becomes even more intuitive.

When creating custom operators, always consider:

- **Single Responsibility Principle**: Each operator should focus on one specific task
- **Type Safety**: Use TypeScript generics to ensure proper type inference
- **Testability**: Design operators to be independently testable
- **Reusability**: Make them generic enough to use throughout your project

By leveraging these custom operators effectively, you can create cleaner and more maintainable Angular applications.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%EC%9E%AC%EC%82%AC%EC%9A%A9-%EA%B0%80%EB%8A%A5%ED%95%9C-RxJS-%ED%8C%A8%ED%84%B4-%EB%A7%8C%EB%93%A4%EA%B8%B0)
<br/>