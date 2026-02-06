---
title: "Shared Module"
date: 2019-07-04 15:44:00 +0900
comments: true
categories: angular
tags: [module]
---

# Sharing Duplicate Modules Using a Shared Module

## 1. When Using a Component in Multiple Modules

- Declaring a component in multiple modules will result in an error.
- The error message suggests declaring the component in a higher-level or separate module and referencing that module instead.
- Declaring a new module every time can make things complex, so a Shared Module should be created to contain commonly used components. Then,modules that need those components can simply reference the Shared Module. (You can also configure all modules to reference the Shared Module.)

## 2. Issues When Declaring a Service in Multiple Modules
- Services are frequently used across multiple modules, but declaring them in multiple modules causes problems.
- If each module declares the service separately, it's equivalent to instantiating a new instance for each module.
- Like components, services should be placed in a Shared Module, which is then referenced by other modules.
- However, if a service is simply declared like this:

```ts
providers: [
  SomeService
]
```

Each module that includes it will instantiate its own instance of the service, leading to unexpected behavior.

- To resolve this, ModuleWithProviders should be used:

```ts
export class SharedModule {
  static forRoot(): ModuleWithProviders {
    return {
      SomeService
    }
  }
}
```

- The root module must reference SharedModule.forRoot(), while other modules should reference SharedModule only.

### 2-1. Services in Angular 6+
Since Angular 6, services are no longer declared in modules but within the service itself.<br/>
A shared service should specify providedIn as either 'root' or SharedModule:

```ts
@Injectable({
  providedIn: 'root' // or sharedModule
})
```

## 3. Sharing in Standalone Components
Although Angular Standalone Components support module-based sharing, it is not recommended.<br/>
Instead, shared functions can be used to achieve similar behavior.

### 3.1 Adding Shared Services to Component Providers
The way a service is used in a component depends on how @Injectable is configured.

- Default Configuration
If @Injectable is used without any configuration, an inject token is only issued to the component where it is declared.

```typescript
// shared/data.service.ts
@Injectable()
export class SharedDataService { }

// feature component
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [CommonModule],
  providers: [SharedDataService]
})
export class FeatureComponent {
  dataService = inject(SharedDataService);
}
```

- Using providedIn: 'root'
When providedIn: 'root' is set, the inject token is globally available, allowing the service to be used anywhere without additional configuration.

```typescript
// shared/data.service.ts
@Injectable({providedIn: 'root'})
export class SharedDataService { }

// feature component
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [CommonModule],
})
export class FeatureComponent {
  dataService = inject(SharedDataService);
}
```

### 3.2 Creating a Shared Function

```typescript
// shared/dependencies.ts
export function getSharedDependencies() {
  return [
    CommonModule,
    SharedButtonComponent,
    // other commonly used standalone components or modules
  ] as const;
}

// Using shared dependencies
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [...getSharedDependencies()]
})
export class FeatureComponent { }
```

While this approach does not allow bundling both components and services like a Shared Module, it provides a way to return a list of dependencies, which can be included in imports and providers.<br/>
This method offers flexibility, making it easier to modify and extend shared dependencies dynamically.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Shared-Module)
<br/>