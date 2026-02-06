---
title: "Getting Started with IndexedDB in TypeScript (vs. LocalStorage)"
date: 2025-10-11 12:00:00 +0900
comments: true
categories: javascript
tags: [typescript, indexeddb, localstorage, storage, async, promise]
---

When it comes to client-side storage, LocalStorage and IndexedDB are the most common choices. While LocalStorage is simple to use, it has limitations when dealing with large, structured data. This is where IndexedDB shines as a powerful alternative.

In this post, we'll create a simple service class to wrap IndexedDB's basic CRUD operations using TypeScript and async/await, and we'll compare it with LocalStorage to understand when to choose one over the other.

### LocalStorage vs. IndexedDB: When to Use What?

Choosing the right tool for the job is crucial, as both technologies have distinct advantages and disadvantages.

### **LocalStorage**

- **Advantages**:
    - **Simple API**: Extremely easy to learn and use with intuitive methods like setItem() and getItem().
    - **Synchronous**: The code works synchronously, making the logic straightforward and easy to follow.
- **Disadvantages**:
    - **Small Capacity**: Limited to about 5MB of storage, depending on the browser.
    - **String-Only**: Can only store strings. Objects and arrays must be serialized with JSON.stringify() and deserialized with JSON.parse().
    - **Main Thread Blocking**: Its synchronous nature means I/O operations can block the main thread, potentially impacting UI performance.
    - **No Querying**: Lacks any mechanism to efficiently search for data based on conditions.

### **IndexedDB**

- **Advantages**:
    - **Large Storage Capacity**: Can store hundreds of megabytes or even gigabytes of data (subject to browser policies and user permission).
    - **Versatile Data Types**: Can store complex JavaScript objects, Files (Blobs), and arrays directly without serialization.
    - **Asynchronous**: All operations are non-blocking, ensuring that the UI remains responsive.
    - **Powerful Querying & Indexing**: Allows you to create indexes on properties for very fast data retrieval.
    - **Transactional**: Operations are wrapped in transactions, ensuring data integrity.
- **Disadvantages**:
    - **Complex API**: The native event-based API is far more complex and verbose than LocalStorage. (This is the problem we'll solve in this post.)

### **Conclusion: When is IndexedDB the Better Choice?**

- When dealing with **large datasets** (e.g., offline articles, user-generated content).
- When you need to store and retrieve **structured objects** frequently.
- When building **offline-first applications** or Progressive Web Apps (PWAs).
- When you need **efficient searching or sorting** capabilities on the client-side.
- When you need to store **binary data**, like images or files.

Now, let's simplify the complex IndexedDB API using TypeScript.

### DEMO Implementation

### Step 1: Define the Data Model

Define a User interface for the data we will store.

```jsx
// src/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
}
```

### Step 2: Create the IndexedDB Service Class

Create a DBService class to encapsulate the IndexedDB API.

```jsx
// src/db.service.ts
import { User } from './user.model';

const DB_NAME = 'MyUserDB';
const STORE_NAME = 'users';
const DB_VERSION = 1;

export class DBService {
  private db: IDBDatabase | null = null;

  // 1. Open and initialize the database
  public async openDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      // ... (Code is identical to the previous example)
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

  // 2. Transaction helper
  private getStore(mode: IDBTransactionMode): IDBObjectStore {
    // ... (Code is identical to the previous example)
    if (!this.db) throw new Error('Database not initialized!');
    return this.db.transaction(STORE_NAME, mode).objectStore(STORE_NAME);
  }

  // 3. Implement CRUD methods (wrapped in Promises)
  public async addUser(user: Omit<User, 'id'>): Promise<User> {
    const request = this.getStore('readwrite').add(user);
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve({ ...user, id: request.result as number });
      request.onerror = () => reject(request.error);
    });
  }
  
  // ... Other CRUD methods (getUser, updateUser, deleteUser) are identical to the previous example
}
```

### Step 3: Example of Using the Service

Here is an example of how to use the DBService.

```jsx
// src/main.ts
import { DBService } from './db.service';

(async () => {
  const dbService = new DBService();
  try {
    await dbService.openDB();
    // 1. Add a user
    const newUser = await dbService.addUser({ name: 'Alice', email: 'alice@example.com' });
    console.log('User added:', newUser);
    // 2. Read, update, delete users...
  } catch (error) {
    console.error('An error occurred:', error);
  }
})();
```

### Final Thoughts

LocalStorage is suitable for storing small amounts of simple data, like user settings or authentication tokens. However, when your application needs to manage large, complex datasets on the client-side efficiently, IndexedDB is the clear winner. By encapsulating its complex API with TypeScript and async/await, you can leverage its powerful features in a much more intuitive way.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/TypeScript%EB%A1%9C-IndexedDB-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-feat.-LocalStorage%EC%99%80-%EB%B9%84%EA%B5%90)
<br/>