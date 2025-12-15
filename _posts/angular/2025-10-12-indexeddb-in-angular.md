---
title: "A Modern Approach to IndexedDB in Angular: Reactive Data with Observables and Signals"
date: 2025-10-12 12:00:00 +0900
comments: true
categories: javascript
tags: [indexeddb, observable, signal, rxjs, standalone]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90%EC%84%9C-IndexedDB%EB%A1%9C-%EB%B0%98%EC%9D%91%ED%98%95-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)
<br/>

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
// src/db.service.ts (Framework-agnostic)
import { User } from './user.model';

const DB_NAME = 'MyUserDB';
const STORE_NAME = 'users';
const DB_VERSION = 1;

export class DBService {
  private db: IDBDatabase | null = null;

  public async openDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      if (this.db) return resolve();
      const request = indexedDB.open(DB_NAME, DB_VERSION);
      request.onerror = () => reject('Error opening database');
      request.onsuccess = () => { this.db = request.result; resolve(); };
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        if (!db.objectStoreNames.contains(STORE_NAME)) {
          db.createObjectStore(STORE_NAME, { keyPath: 'id', autoIncrement: true });
        }
      };
    });
  }

  private getStore(mode: IDBTransactionMode): IDBObjectStore {
    if (!this.db) throw new Error('Database not initialized!');
    return this.db.transaction(STORE_NAME, mode).objectStore(STORE_NAME);
  }

  public async getAllUsers(): Promise<User[]> {
    const request = this.getStore('readonly').getAll();
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  public async addUser(user: Omit<User, 'id'>): Promise<User> {
    const request = this.getStore('readwrite').add(user);
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve({ ...user, id: request.result as number });
      request.onerror = () => reject(request.error);
    });
  }
  
  public async updateUser(user: User): Promise<User> {
    const request = this.getStore('readwrite').put(user);
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve(user);
      request.onerror = () => reject(request.error);
    });
  }

  public async deleteUser(id: number): Promise<void> {
    const request = this.getStore('readwrite').delete(id);
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}
```

### Step 2: The Angular Service Wrapper (The Observable Bridge)

Now, let's create an Angular Injectable service that converts the Promises from DBService into Observables.

```jsx
// src/app/user-db.service.ts (Angular Service)
import { Injectable } from '@angular/core';
import { from, Observable, switchMap, shareReplay } from 'rxjs';
import { DBService } from '../db.service';
import { User } from '../user.model';

@Injectable({ providedIn: 'root' })
export class UserDbService {
  private dbService = new DBService();
  private dbReady$ = from(this.dbService.openDB()).pipe(shareReplay(1));

  getAllUsers(): Observable<User[]> {
    return this.dbReady$.pipe(switchMap(() => from(this.dbService.getAllUsers())));
  }

  addUser(user: Omit<User, 'id'>): Observable<User> {
    return this.dbReady$.pipe(switchMap(() => from(this.dbService.addUser(user))));
  }

  updateUser(user: User): Observable<User> {
    return this.dbReady$.pipe(switchMap(() => from(this.dbService.updateUser(user))));
  }

  deleteUser(id: number): Observable<void> {
    return this.dbReady$.pipe(switchMap(() => from(this.dbService.deleteUser(id))));
  }
}
```

### Step 3: The Reactive Component (Signals & UI)

In our component, we inject UserDbService to manage data. The state of the data is stored in a **Signal** to synchronize it with the UI.

```jsx
// src/app/app.component.ts
import { Component, OnInit, inject, signal } from '@angular/core';
import { UserDbService } from './user-db.service';
import { User } from '../user.model';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit {
  private userDbService = inject(UserDbService);
  users = signal<User[]>([]);

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.userDbService.getAllUsers().subscribe({
      next: (users) => this.users.set(users),
      error: (err) => console.error('Failed to load users', err),
    });
  }

  addUser(): void {
    const name = prompt("Enter user name:");
    const email = prompt("Enter user email:");
    if (!name || !email) return;

    this.userDbService.addUser({ name, email }).subscribe({
      next: (addedUser) => {
        this.users.update((currentUsers) => [...currentUsers, addedUser]);
      },
      error: (err) => console.error('Failed to add user', err),
    });
  }

  updateUser(user: User): void {
    const newName = prompt("Enter new name:", user.name);
    if (!newName) return;

    const updatedUser = { ...user, name: newName };
    this.userDbService.updateUser(updatedUser).subscribe({
      next: (returnedUser) => {
        this.users.update((users) =>
          users.map((u) => (u.id === returnedUser.id ? returnedUser : u))
        );
      },
      error: (err) => console.error('Failed to update user', err),
    });
  }

  deleteUser(id: number): void {
    if (!confirm('Are you sure you want to delete this user?')) return;

    this.userDbService.deleteUser(id).subscribe({
      next: () => {
        this.users.update((users) => users.filter((u) => u.id !== id));
      },
      error: (err) => console.error('Failed to delete user', err),
    });
  }
}
```

### Step 4: The Template (Control Flow)

We use @for to iterate over the users Signal and render the UI.

```jsx
<!-- src/app/app.component.html -->
<main>
  <h1>Angular IndexedDB with Signals & Observables</h1>
  <button (click)="addUser()">Add User</button>
  
  <ul>
    @for (user of users(); track user.id) {
      <li>
        <span>ID: {{ user.id }} | Name: {{ user.name }} | Email: {{ user.email }}</span>
        <div>
          <button (click)="updateUser(user)">Update Name</button>
          <button (click)="deleteUser(user.id)">Delete</button>
        </div>
      </li>
    } @empty {
      <p>No users found in IndexedDB. Add one!</p>
    }
  </ul>
</main>
```

### Conclusion

Integrating IndexedDB with Angular's reactive paradigm creates a powerful and declarative data management architecture.

- **Low-Level Promise Service**: Encapsulates the complexity of IndexedDB.
- **High-Level Observable Service**: Manages asynchronous data flows and connects to the Angular ecosystem.
- **Signals in Components**: Manage UI state and automatically update the view in response to data changes.

This layered approach enhances code reusability and testability, helping you maintain clean and robust client-side data logic.