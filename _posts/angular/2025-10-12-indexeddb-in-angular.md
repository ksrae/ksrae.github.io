---
title: "A Modern Approach to IndexedDB in Angular: Reactive Data with Observables and Signals"
date: 2025-10-12 12:00:00 +0900
comments: true
categories: javascript
tags: [indexeddb, observable, signal, rxjs, standalone]
---

By combining **RxJS Observables** for handling data streams and **Signals** for automatic UI updates upon state changes, we can manage IndexedDB in a much more powerful and declarative way.

This article will guide you through a complete CRUD example of integrating a Promise-based IndexedDB service into Angular's reactive ecosystem.

### The Plan

1. **Separate Layers**: We'll separate a low-level, framework-agnostic DB service from a high-level Angular service that wraps it with Observables.
2. **Observable Transformation**: Convert the Promise-based methods of the low-level service into RxJS Observables using the from operator.
3. **State Management with Signals**: Within the component, use Signals to manage the data state, ensuring the UI updates automatically whenever the data changes.
4. **Complete Reactive UI**: Render the data from the Signal in the template using the @for syntax and implement full CRUD functionality.

### DEMO Implementation

### Step 1: The Low-Level DB Service (Promise-based)

First, we'll write a pure TypeScript DBService that has no Angular dependencies. All CRUD operations return Promises.

```jsx
// src/app/db.service.ts
import { User } from './user.model';

export class DBService {
  private db: IDBDatabase | null = null;

  // 1. Open Database
  async openDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('MyUserDB', 1);

      request.onerror = () => reject('Error opening DB');
      
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event: any) => {
        const db = event.target.result;
        if (!db.objectStoreNames.contains('users')) {
          db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
        }
      };
    });
  }

  // 2. Helper to convert IDBRequest to Promise 
  private requestToPromise<T>(request: IDBRequest<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  // 3. Get Store Helper
  private getStore(mode: IDBTransactionMode) {
    if (!this.db) throw new Error('DB not initialized');
    return this.db.transaction('users', mode).objectStore('users');
  }

  // --- CRUD Operations ---

  async getAllUsers(): Promise<User[]> {
    const store = this.getStore('readonly');
    return this.requestToPromise(store.getAll());
  }

  async addUser(user: Omit<User, 'id'>): Promise<number> {
    const store = this.getStore('readwrite');
    return this.requestToPromise(store.add(user) as IDBRequest<number>);
  }

  async updateUser(user: User): Promise<void> {
    const store = this.getStore('readwrite');
    await this.requestToPromise(store.put(user));
  }

  async deleteUser(id: number): Promise<void> {
    const store = this.getStore('readwrite');
    await this.requestToPromise(store.delete(id));
  }
}
```

### Step 2: The Angular Service Wrapper (The Observable Bridge)

Now, let's create an Angular Injectable service that converts the Promises from DBService into Observables.

```jsx
// src/app/user-db.service.ts
import { Injectable } from '@angular/core';
import { from, Observable } from 'rxjs';
import { DBService } from './db.service';
import { User } from './user.model';

@Injectable({ providedIn: 'root' })
export class UserDbService {
  private dbService = new DBService();
  private dbReady = this.dbService.openDB(); // Promise that resolves when DB is open

  // Helper: Wait for DB, then run action
  private async perform<T>(action: () => Promise<T>): Promise<T> {
    await this.dbReady;
    return action();
  }

  getAllUsers(): Observable<User[]> {
    return from(this.perform(() => this.dbService.getAllUsers()));
  }

  addUser(user: Omit<User, 'id'>): Observable<number> {
    return from(this.perform(() => this.dbService.addUser(user)));
  }

  updateUser(user: User): Observable<void> {
    return from(this.perform(() => this.dbService.updateUser(user)));
  }

  deleteUser(id: number): Observable<void> {
    return from(this.perform(() => this.dbService.deleteUser(id)));
  }
}
```

### Step 3: The Reactive Component (Signals & UI)

In our component, we inject UserDbService to manage data. The state of the data is stored in a **Signal** to synchronize it with the UI.

```jsx
// src/app/app.component.ts
import { Component, OnInit, inject, signal } from '@angular/core';
import { UserDbService } from './user-db.service';
import { User } from './user.model';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit {
  private userDb = inject(UserDbService);
  
  // Signal for UI state
  users = signal<User[]>([]);

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.userDb.getAllUsers().subscribe(data => {
      this.users.set(data);
    });
  }

  // Now receives data from HTML inputs
  addUser(name: string, email: string) {
    if (!name || !email) return;

    this.userDb.addUser({ name, email }).subscribe(() => {
      this.loadUsers();
    });
  }

  // Simple update logic (e.g., appends ' (Edited)')
  updateUser(user: User) {
    const updatedUser = { ...user, name: user.name + ' (Edited)' };
    
    this.userDb.updateUser(updatedUser).subscribe(() => {
      this.loadUsers();
    });
  }

  deleteUser(id: number) {
    this.userDb.deleteUser(id).subscribe(() => {
      this.loadUsers();
    });
  }
}
```

### Step 4: The Template (Control Flow)

We use @for to iterate over the users Signal and render the UI.

```jsx
<!-- src/app/app.component.html -->
<h1>Angular IndexedDB Simple Example</h1>
<div>
  <input #nameInput placeholder="Name">
  <input #emailInput placeholder="Email">
  
  <button (click)="addUser(nameInput.value, emailInput.value); nameInput.value=''; emailInput.value=''">
    Add User
  </button>
</div>

<hr>

<ul>
  @for (user of users(); track user.id) {
    <li>
      <strong>{{ user.name }}</strong> ({{ user.email }})
      
      <button (click)="updateUser(user)">Mark Edited</button>
      
      @if (user.id; as id) {
        <button (click)="deleteUser(id)">Delete</button>
      }
    </li>
  } @empty {
    <p>No users found.</p>
  }
</ul>
```

### Conclusion

Integrating IndexedDB with Angular's reactive paradigm creates a powerful and declarative data management architecture.

- **Low-Level Promise Service**: Encapsulates the complexity of IndexedDB.
- **High-Level Observable Service**: Manages asynchronous data flows and connects to the Angular ecosystem.
- **Signals in Components**: Manage UI state and automatically update the view in response to data changes.

This layered approach enhances code reusability and testability, helping you maintain clean and robust client-side data logic.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90%EC%84%9C-IndexedDB%EB%A1%9C-%EB%B0%98%EC%9D%91%ED%98%95-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)
<br/>