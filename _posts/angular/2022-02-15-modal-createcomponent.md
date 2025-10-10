---
title: "Create Modal with createComponent"
date: 2022-02-15 00:00:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent, modal]
---

createComponent provides lower-level control over dynamic components than ngComponentOutlet, making it ideal for building complex features like a full-fledged Modal system.

# Defining a Modal
Unlike a simple tooltip, a modal system has several requirements:
- Service-Based Invocation: Modals should be launchable from anywhere in the application, not just tied to a specific host element's event. A reusable Injectable service is the perfect solution.
- Layer Management: It should be possible to stack modals on top of each other. (While this post covers a basic single-modal implementation, the architecture should be extensible.)
- Returning a Result: When a modal closes, it should be able to return a value (e.g., 'confirm', 'cancel', or custom data). The calling code needs to be able to await this result.

# Basic Operation Plan
- Create a ModalService: This centralized service will handle dynamic component creation (createComponent), return an Observable for the result, and manage closing the modal.
- Set up a ModalHost: In AppComponent, we'll define a dedicated location (ViewContainerRef) for rendering modals and register it with the ModalService.
- Create Dynamic Modal Components: Build standalone modal components that interact with the user and pass results back to the service via a Subject.
- Create a Container Component: This component will inject the ModalService to open modals and will store the returned result in a Signal to display in the UI.

## 1. Creating the ModalService
This is the heart of our system. It manages the creation, destruction, and result handling of all modals.

```Ts
// src/app/modal.service.ts
import { Injectable, ViewContainerRef, ComponentRef, Type } from '@angular/core';
import { Subject, Observable } from 'rxjs';
import { first, tap } from 'rxjs/operators';

// An interface that modal components should implement
export interface ModalComponent<R> {
  result$: Subject<R>;
}

@Injectable({ providedIn: 'root' })
export class ModalService {
  private hostVcr?: ViewContainerRef;
  private componentRef?: ComponentRef<ModalComponent<any>>;

  // Register the ViewContainerRef where modals will be rendered
  registerHost(vcr: ViewContainerRef): void {
    this.hostVcr = vcr;
  }

  // Use generics to open any component and return any type of result
  open<T extends ModalComponent<R>, R>(component: Type<T>): Observable<R> {
    if (!this.hostVcr) {
      throw new Error('Modal host container is not registered!');
    }
    // Clear previous modal (single modal policy)
    this.hostVcr.clear();

    const result$ = new Subject<R>();
    
    // Dynamically create the component
    this.componentRef = this.hostVcr.createComponent(component);
    // Inject the result$ Subject into the component's instance
    this.componentRef.instance.result$ = result$;

    // Return an observable that closes the modal after the first result
    return result$.asObservable().pipe(
      first(),
      tap(() => this.close())
    );
  }

  close(): void {
    if (this.componentRef) {
      this.componentRef.destroy();
      this.componentRef = undefined;
    }
  }
}
```

## 2. Creating the Components
### AppComponent (Setting up the Modal Host)
At the top level of our application, we tell the ModalService where it should render modals.

```Ts
// src/app/app.component.ts
import { Component, ViewChild, ViewContainerRef, AfterViewInit, inject } from '@angular/core';
import { ContainerComponent } from './container.component'; // Our example calling component
import { ModalService } from './modal.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ContainerComponent],
  template: `
    <app-container></app-container>
    <!-- All modals will be rendered here -->
    <ng-container #modalHost></ng-container>
  `,
})
export class AppComponent implements AfterViewInit {
  @ViewChild('modalHost', { read: ViewContainerRef })
  modalHostVcr!: ViewContainerRef;

  private modalService = inject(ModalService);
  
  ngAfterViewInit(): void {
    // Register the host ViewContainerRef with the service
    this.modalService.registerHost(this.modalHostVcr);
  }
}
```

### Dynamic Modal Components (AModalComponent, BModalComponent)
These components implement the ModalComponent interface, giving them a result$ property.

```Ts
// src/app/a-modal.component.ts
import { Component } from '@angular/core';
import { Subject } from 'rxjs';
import { ModalComponent } from './modal.service';

@Component({
  standalone: true,
  template: `
    <div class="modal">
      <p>Modal A: Are you sure?</p>
      <button (click)="onConfirm()">Confirm</button>
      <button (click)="onCancel()">Cancel</button>
    </div>
  `,
  styles: `.modal { border: 1px solid blue; padding: 20px; background: white; }`
})
export class AModalComponent implements ModalComponent<boolean> {
  result$!: Subject<boolean>;

  onConfirm(): void {
    this.result$.next(true);
    this.result$.complete();
  }
  onCancel(): void {
    this.result$.next(false);
    this.result$.complete();
  }
}
```
(BModalComponent can be created similarly.)

### Container Component (The Modal Caller)
Now, any component that needs to open a modal only needs to inject the ModalService.

```Ts
// src/app/container.component.ts
import { Component, inject, signal } from '@angular/core';
import { ModalService } from './modal.service';
import { AModalComponent } from './a-modal.component';
import { BModalComponent } from './b-modal.component';

@Component({
  selector: 'app-container',
  standalone: true,
  template: `
    <button (click)="openModal('A')">Show Modal A</button>
    <button (click)="openModal('B')">Show Modal B</button>
    <p>Last Modal Result: {{ modalResult() }}</p>
  `
})
export class ContainerComponent {
  private modalService = inject(ModalService);
  modalResult = signal<any>('N/A');

  openModal(type: 'A' | 'B'): void {
    const modalToOpen = type === 'A' ? AModalComponent : BModalComponent;
    
    this.modalService.open(modalToOpen).subscribe(result => {
      // Store the returned result in our Signal after the modal closes
      this.modalResult.set(result);
    });
  }
}
```

# Result
This architecture provides a clean and reusable foundation for a modal system by leveraging Angular's DI.
- Separation of Concerns: The component opening a modal doesn't need to know where or how it's rendered. It only needs to know about the ModalService.
- Type Safety: Generics ensure that the type of the opened component and its return value are clearly defined.
- Extensibility: This structure can be easily extended to handle features like modal stacking, animations, or centralized state management.