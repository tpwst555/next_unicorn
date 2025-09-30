This document outlines strict rules and patterns that must be followed for all code contributions.

# Svelte 5 Syntax

## Runes Usage Rules

### $state
- ✅ **MUST** use `$state()` for all reactive variables
- ✅ **MUST** use `$state.raw()` for large datasets that don't need deep reactivity
- ✅ **MUST** use `$state<Type>()` with explicit TypeScript types
- ❌ **NEVER** use plain `let` for reactive data
- ❌ **NEVER** export raw state variables directly - wrap in functions/classes
- ❌ **NEVER** try to make a `$state` variable non-reactive after creation

```svelte
<!-- ✅ CORRECT -->
<script>
  let count = $state(0);
  let todos = $state<Todo[]>([]);
  let largeData = $state.raw(initialData);
</script>

<!-- ❌ INCORRECT -->
<script>
  let count = 0; // Missing reactivity
  export let todos = $state([]); // Exporting raw state
</script>
```

### $derived
- ✅ **MUST** use for all computed values
- ✅ **MUST** contain only pure, side-effect-free expressions
- ✅ **MUST** use `$derived.by()` for complex computations requiring multiple statements
- ❌ **NEVER** use `$effect` to compute derived values
- ❌ **NEVER** perform async operations inside `$derived`
- ❌ **NEVER** modify state inside `$derived`

```svelte
<!-- ✅ CORRECT -->
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  let complex = $derived.by(() => {
    const temp = count * 2;
    return temp + 10;
  });
</script>

<!-- ❌ INCORRECT -->
<script>
  let count = $state(0);
  let doubled = $state(0);
  $effect(() => {
    doubled = count * 2; // Should use $derived
  });
</script>
```

### $effect
- ✅ **MUST** return cleanup functions for subscriptions/timers
- ✅ **MUST** use `untrack()` for untracked dependencies
- ✅ **MUST** handle errors appropriately
- ✅ **MUST** wrap async operations in an IIFE since $effect cannot be async
- ❌ **NEVER** rely on effects for SSR (they don't run server-side)
- ❌ **NEVER** unconditionally modify tracked state (causes infinite loops)
- ❌ **NEVER** use for computing derived values
- ❌ **NEVER** use async functions directly with $effect

```svelte
<!-- ✅ CORRECT -->
<script>
  import { untrack } from 'svelte';
  
  let count = $state(0);
  
  $effect(() => {
    const timer = setInterval(() => {
      untrack(() => {
        console.log(count);
      });
    }, 1000);
    
    return () => clearInterval(timer); // Cleanup
  });
  
  // Async operations in $effect
  $effect(() => {
    (async () => {
      const data = await fetchData();
      // handle data
    })();
  });
</script>

<!-- ❌ INCORRECT -->
<script>
  $effect(() => {
    count = count + 1; // Infinite loop!
  });
  
  // Cannot use async directly
  $effect(async () => { // ❌ Error!
    await fetchData();
  });
</script>
```

### $props
- ✅ **MUST** destructure with TypeScript interface
- ✅ **MUST** provide default values in destructuring
- ✅ **MUST** use `$bindable()` explicitly for two-way binding
- ❌ **NEVER** use `export let` in Svelte 5 mode
- ❌ **NEVER** mutate non-bindable props

```svelte
<!-- ✅ CORRECT -->
<script lang="ts">
  interface Props {
    count: number;
    title?: string;
    value: string;
  }
  
  let { count, title = 'Default', value = $bindable() }: Props = $props();
</script>

<!-- ❌ INCORRECT -->
<script>
  export let count; // Old syntax
  let props = $props();
  props.value = 'new'; // Mutating non-bindable prop
</script>
```

## Event Handlers
- ✅ **MUST** use lowercase `onclick`, `oninput`, etc.
- ✅ **MUST** handle preventDefault/stopPropagation in handler functions
- ❌ **NEVER** use `on:click` (colon syntax)
- ❌ **NEVER** use event modifiers like `|preventDefault`

```svelte
<!-- ✅ CORRECT -->
<button onclick={(e) => {
  e.preventDefault();
  handleClick();
}}>Click me</button>

<!-- ❌ INCORRECT -->
<button on:click|preventDefault={handleClick}>Click me</button>
```

## Snippets and Rendering
- ✅ **MUST** use `{#snippet}` instead of slots
- ✅ **MUST** use optional chaining when rendering snippets
- ✅ **MUST** type snippets with `Snippet` or `Snippet<[Type]>`
- ❌ **NEVER** use `<slot>` in new components
- ❌ **NEVER** assume snippets are always provided

```svelte
<!-- ✅ CORRECT -->
{#snippet header(title: string)}
  <h1>{title}</h1>
{/snippet}

{@render header?.('Welcome')}

<!-- ❌ INCORRECT -->
<slot name="header" />
{@render header('Welcome')} <!-- Missing optional chaining -->
```

# Component Patterns

## State Management
- ✅ **MUST** use `.svelte.js/.svelte.ts` files for shared reactive state
- ✅ **MUST** encapsulate state with getter/setter patterns
- ✅ **MUST** use classes with `$state` for complex state
- ✅ **MUST** use `fromStore()` and `toStore()` utilities when interoperating with legacy store code
- ❌ **AVOID** mixing stores and runes in new code - prefer runes for consistency
- ❌ **NEVER** export raw `$state` variables

```js
// ✅ CORRECT - counter.svelte.js
export function createCounter() {
  let count = $state(0);
  
  return {
    get value() { return count; },
    increment() { count++; },
    reset() { count = 0; }
  };
}

// ✅ CORRECT - Interoperating with store-based libraries
import { fromStore, toStore } from 'svelte/store';
import { someLibraryStore } from 'third-party-lib';

// Convert store to rune-compatible state
let storeValue = fromStore(someLibraryStore);

// ❌ INCORRECT
export let count = $state(0); // Exporting raw state
```

## Component Lifecycle
- ✅ **MUST** use `$effect` for reactive lifecycle logic that responds to state changes
- ✅ **MUST** use `onMount` for one-time client-side initialization (e.g., setting up third-party libraries, initial data fetching)
- ✅ **MUST** include cleanup functions in `$effect` if setting up subscriptions, timers, or event listeners
- ❌ **NEVER** use `beforeUpdate`/`afterUpdate` in runes mode
- ❌ **NEVER** use async functions directly in `$effect` (wrap in IIFE if needed)
- ❌ **NEVER** rely on effect order for initialization

## Props Validation
- ✅ **MUST** validate required props
- ✅ **MUST** handle edge cases (null, undefined, empty arrays)
- ✅ **MUST** provide meaningful error messages
- ❌ **NEVER** assume props are always valid

## Lifecycle Patterns Example

```svelte
<script>
  import { onMount } from 'svelte';
  
  // Use onMount for one-time initialization
  onMount(async () => {
    // Can be async
    const data = await fetchInitialData();
    // Initialize third-party library
    initializeChart(chartElement);
    
    return () => {
      // Cleanup on unmount
      destroyChart();
    };
  });
  
  // Use $effect for reactive updates
  let chartData = $state([]);
  
  $effect(() => {
    // Runs when chartData changes
    if (chartElement && chartData.length > 0) {
      updateChart(chartElement, chartData);
    }
  });
</script>
```

# Performance

## Reactivity Optimization
- ✅ **MUST** use `$state.raw()` for large, immutable datasets
- ✅ **MUST** use keyed `{#each}` blocks with stable IDs
- ✅ **MUST** prefer `$derived` over `$effect` for computations
- ❌ **NEVER** create unnecessary reactive proxies
- ❌ **NEVER** use index as key in `{#each}` blocks

```svelte
<!-- ✅ CORRECT -->
{#each items as item (item.id)}
  <Item {item} />
{/each}

<!-- ❌ INCORRECT -->
{#each items as item, i (i)}
  <Item {item} />
{/each}
```

## Bundle Size
- ✅ **MUST** use dynamic imports for heavy components
- ✅ **MUST** implement route-based code splitting
- ✅ **MUST** tree-shake unused code
- ❌ **NEVER** import entire libraries when only specific functions needed
- ❌ **NEVER** bundle development-only code in production

## Rendering Optimization
- ✅ **MUST** use `$derived` for expensive computations (auto-memoized)
- ✅ **MUST** implement virtual scrolling for large lists
- ✅ **MUST** lazy-load images with `loading="lazy"`
- ❌ **NEVER** render thousands of DOM nodes without virtualization
- ❌ **NEVER** perform heavy computations in templates

# Common Antipatterns

## Reactivity Mistakes
- ❌ **NEVER** use `$effect` for derived state
- ❌ **NEVER** forget cleanup functions in effects
- ❌ **NEVER** create circular dependencies
- ❌ **NEVER** mutate state without `$state`
- ❌ **NEVER** use native Set/Map instead of SvelteSet/SvelteMap for reactive collections

```svelte
<!-- ❌ INCORRECT - Common mistakes -->
<script>
  import { SvelteSet, SvelteMap } from 'svelte/reactivity';
  
  // Using effect for derived state
  let doubled = $state(0);
  $effect(() => {
    doubled = count * 2; // Should be $derived
  });
  
  // Missing cleanup
  $effect(() => {
    const timer = setInterval(...); // No cleanup!
  });
  
  // Using native collections (not reactive!)
  let items = $state(new Set()); // ❌ Should use SvelteSet
  let data = $state(new Map()); // ❌ Should use SvelteMap
  
  // ✅ CORRECT
  let items = $state(new SvelteSet());
  let data = $state(new SvelteMap());
</script>
```

## Memory Leaks
- ❌ **NEVER** create subscriptions without cleanup
- ❌ **NEVER** hold references to unmounted components
- ❌ **NEVER** forget to remove event listeners
- ❌ **NEVER** create infinite effect loops

## TypeScript Errors
- ❌ **NEVER** use `any` type
- ❌ **NEVER** ignore TypeScript errors
- ❌ **NEVER** use type assertions instead of proper typing
- ❌ **NEVER** skip interface definitions for props

# Accessibility

## Semantic HTML
- ✅ **MUST** use semantic HTML elements
- ✅ **MUST** provide alt text for images
- ✅ **MUST** use proper heading hierarchy (h1 → h2 → h3)
- ✅ **MUST** associate labels with form inputs
- ❌ **NEVER** skip heading levels
- ❌ **NEVER** use placeholder as label replacement

## Keyboard Navigation
- ✅ **MUST** support keyboard for all interactive elements
- ✅ **MUST** provide visible focus indicators
- ✅ **MUST** implement focus traps for modals
- ✅ **MUST** use `tabindex="0"` for custom interactive elements
- ❌ **NEVER** use positive tabindex values
- ❌ **NEVER** remove focus outlines without replacement

## ARIA
- ✅ **MUST** use appropriate ARIA roles
- ✅ **MUST** provide aria-labels for icon buttons
- ✅ **MUST** use aria-live for dynamic content
- ✅ **MUST** implement aria-expanded for collapsibles
- ❌ **NEVER** use ARIA to fix bad HTML
- ❌ **NEVER** change native semantics unnecessarily

```svelte
<!-- ✅ CORRECT -->
<button aria-label="Close dialog" onclick={closeDialog}>
  <Icon name="close" />
</button>

<div role="status" aria-live="polite">
  {message}
</div>

<!-- ❌ INCORRECT -->
<div onclick={handleClick}>Click me</div> <!-- Not keyboard accessible -->
<img src="logo.png" /> <!-- Missing alt -->
```

## Color and Contrast
- ✅ **MUST** meet WCAG AA contrast ratios (4.5:1 normal, 3:1 large text)
- ✅ **MUST** not convey information through color alone
- ✅ **MUST** support dark mode preferences
- ❌ **NEVER** use color as the only differentiator

# Testing

## Unit Testing Rules
- ✅ **MUST** test all exported functions
- ✅ **MUST** test error states
- ✅ **MUST** test edge cases
- ✅ **MUST** achieve 80% code coverage minimum
- ❌ **NEVER** test implementation details
- ❌ **NEVER** test framework internals

## Component Testing
- ✅ **MUST** use @testing-library/svelte
- ✅ **MUST** test user interactions
- ✅ **MUST** test accessibility with appropriate queries
- ✅ **MUST** test async behavior
- ❌ **NEVER** test internal component state directly
- ❌ **NEVER** use arbitrary test IDs when semantic queries work

```js
// ✅ CORRECT
import { render, fireEvent } from '@testing-library/svelte';

test('increments count on click', async () => {
  const { getByRole, getByText } = render(Counter);
  const button = getByRole('button', { name: /increment/i });
  await fireEvent.click(button);
  expect(getByText('1')).toBeInTheDocument();
});

// ❌ INCORRECT
test('sets internal state', () => {
  const component = render(Counter);
  component.count = 5; // Testing internals
});
```

# Migration from Svelte 4

## Component Creation
- ✅ **MUST** use `mount()` instead of `new Component()`
- ✅ **MUST** use `unmount()` instead of `$destroy()`
- ✅ **MUST** use `render()` from 'svelte/server' for SSR
- ❌ **NEVER** use class instantiation
- ❌ **NEVER** access component instances directly

```js
// ✅ CORRECT - Svelte 5
import { mount, unmount } from 'svelte';
const app = mount(App, { target: document.body });
unmount(app);

// ❌ INCORRECT - Svelte 4 style
const app = new App({ target: document.body });
app.$destroy();
```

## Event Handling Migration
- ✅ **MUST** convert `on:event` to `onevent`
- ✅ **MUST** handle modifiers in event handler
- ✅ **MUST** convert dispatch to callback props
- ❌ **NEVER** use createEventDispatcher in Svelte 5
- ❌ **NEVER** use event modifiers

## Store to Rune Migration
- ✅ **MUST** use `fromStore()` to convert stores to rune-compatible state
- ✅ **MUST** use `toStore()` to convert runes to stores when needed for compatibility
- ✅ **MUST** migrate bottom-up (leaf components first)
- ✅ **PREFER** runes for new code, stores only for compatibility

```js
import { fromStore, toStore } from 'svelte/store';
import { myLegacyStore } from './stores';

// Convert store to rune
let storeValue = fromStore(myLegacyStore);

// Convert rune to store (for compatibility)
let count = $state(0);
let countStore = toStore(() => count, (v) => count = v);
```

## Slots to Snippets
- ✅ **MUST** convert `<slot>` to snippets
- ✅ **MUST** convert slot props to snippet parameters
- ✅ **MUST** handle optional snippets with optional chaining
- ❌ **NEVER** mix slots and snippets

# TypeScript Guidelines

## Type Definitions

### Primitive Types
- ✅ **MUST** use lowercase primitives (`string`, `number`, `boolean`)
- ✅ **MUST** explicitly type function parameters
- ✅ **MUST** explicitly type function return values
- ❌ **NEVER** use uppercase primitives (`String`, `Number`, `Boolean`)
- ❌ **NEVER** use `Function` type - use specific function signatures
- ❌ **NEVER** use `Object` type - use `object` or specific interface

```typescript
// ✅ CORRECT
function processData(input: string, count: number): boolean {
  return input.length > count;
}

const handler: (event: MouseEvent) => void = (e) => {
  console.log(e.clientX);
};

// ❌ INCORRECT
function processData(input: String, count: Number): Boolean { // Wrong primitives
  return input.length > count;
}

const handler: Function = (e) => { }; // Too generic
```

### Interfaces vs Types
- ✅ **MUST** use interfaces for object shapes
- ✅ **MUST** use types for unions, intersections, and aliases
- ✅ **MUST** use PascalCase for types and interfaces
- ❌ **NEVER** use `interface` for function types - use type aliases
- ❌ **NEVER** mix interface and type for the same purpose

```typescript
// ✅ CORRECT
interface User {
  id: string;
  name: string;
  email: string;
}

type Status = 'pending' | 'active' | 'disabled';
type AsyncFunction<T> = () => Promise<T>;
type UserWithStatus = User & { status: Status };

// ❌ INCORRECT
type User = { // Should use interface for object shapes
  id: string;
  name: string;
};

interface AsyncFunction { // Should use type for functions
  (): Promise<void>;
}
```

### Generics
- ✅ **MUST** use descriptive generic names when single letter isn't clear
- ✅ **MUST** provide generic constraints when applicable
- ✅ **MUST** use default generic types where sensible
- ❌ **NEVER** use single letters for complex generic relationships
- ❌ **AVOID** overly complex generic types - if more than a few parameters are needed, consider refactoring

```typescript
// ✅ CORRECT
interface ApiResponse<TData = unknown, TError = Error> {
  data?: TData;
  error?: TError;
  loading: boolean;
}

function updateRecord<T extends { id: string }>(record: T): T {
  return { ...record, updatedAt: Date.now() };
}

// ❌ INCORRECT
interface Response<T, U, V, W, X> { // Too complex - consider refactoring
  data: T;
  meta: U;
  error: V;
  status: W;
  headers: X;
}
```

## Type Safety Patterns

### Null and Undefined Handling
- ✅ **MUST** use optional chaining (`?.`) for nullable access
- ✅ **MUST** use nullish coalescing (`??`) for default values
- ✅ **MUST** explicitly handle null/undefined in types
- ❌ **NEVER** use non-null assertion (`!`) without certainty
- ❌ **NEVER** assume values are defined without checking

```typescript
// ✅ CORRECT
interface Config {
  api?: {
    endpoint?: string;
    timeout?: number;
  };
}

function getEndpoint(config: Config): string {
  return config.api?.endpoint ?? 'https://default.api.com';
}

// ❌ INCORRECT
function getEndpoint(config: Config): string {
  return config.api!.endpoint!; // Dangerous non-null assertions
}
```

### Type Guards
- ✅ **MUST** use type guards for runtime type checking
- ✅ **MUST** use `is` keyword for custom type guards
- ✅ **MUST** validate external data with type guards
- ❌ **NEVER** cast without validation
- ❌ **NEVER** trust external data types

```typescript
// ✅ CORRECT
interface Cat {
  type: 'cat';
  meow(): void;
}

interface Dog {
  type: 'dog';
  bark(): void;
}

type Animal = Cat | Dog;

function isCat(animal: Animal): animal is Cat {
  return animal.type === 'cat';
}

function handleAnimal(animal: Animal) {
  if (isCat(animal)) {
    animal.meow(); // Type-safe
  } else {
    animal.bark(); // Type-safe
  }
}

// ❌ INCORRECT
function handleAnimal(animal: Animal) {
  (animal as Cat).meow(); // Unsafe cast
}
```

### Const Assertions
- ✅ **MUST** use `as const` for literal types
- ✅ **MUST** use `readonly` for immutable properties
- ✅ **MUST** use `ReadonlyArray<T>` or `readonly T[]` for immutable arrays
- ❌ **NEVER** mutate readonly properties
- ❌ **NEVER** cast away const assertions

```typescript
// ✅ CORRECT
const ROUTES = {
  HOME: '/',
  ABOUT: '/about',
  CONTACT: '/contact'
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES];

const config = {
  endpoints: ['api/v1', 'api/v2'] as readonly string[],
  timeout: 5000
} as const;

// ❌ INCORRECT
const ROUTES = {
  HOME: '/',
  ABOUT: '/about'
}; // Missing 'as const', will be typed as { HOME: string; ABOUT: string; }
```

## Enums and Unions

### String Literal Unions vs Enums
- ✅ **MUST** prefer const objects with `as const` over enums
- ✅ **MUST** use string literal unions for simple cases
- ✅ **MUST** use const enums if enums are necessary
- ❌ **NEVER** use numeric enums without explicit values
- ❌ **NEVER** mix string and number in enums

```typescript
// ✅ CORRECT - Preferred approach
const Status = {
  PENDING: 'pending',
  ACTIVE: 'active',
  DISABLED: 'disabled'
} as const;

type Status = typeof Status[keyof typeof Status];

// ✅ CORRECT - Simple union
type Theme = 'light' | 'dark' | 'auto';

// ❌ INCORRECT
enum Status {
  Pending, // Implicit numeric value
  Active,  // Implicit numeric value
  Disabled = 'disabled' // Mixed types!
}
```

## Error Handling Types

### Result Types
- ✅ **MUST** use Result/Either pattern for expected errors
- ✅ **MUST** type error cases explicitly
- ✅ **MUST** use discriminated unions for error types
- ❌ **NEVER** throw errors in pure functions
- ❌ **NEVER** use `try/catch` for control flow

```typescript
// ✅ CORRECT
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

function parseJSON<T>(json: string): Result<T, SyntaxError> {
  try {
    return { success: true, data: JSON.parse(json) };
  } catch (error) {
    return { success: false, error: error as SyntaxError };
  }
}

// Usage
const result = parseJSON<User>(jsonString);
if (result.success) {
  console.log(result.data.name); // Type-safe access
} else {
  console.error(result.error.message);
}

// ❌ INCORRECT
function parseJSON(json: string): any {
  try {
    return JSON.parse(json);
  } catch {
    return null; // Loss of error information
  }
}
```

## Utility Types

### Built-in Utility Types
- ✅ **MUST** use TypeScript utility types appropriately
- ✅ **MUST** prefer utility types over manual type manipulation
- ❌ **NEVER** reimplement built-in utility types

```typescript
// ✅ CORRECT
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

// Use utility types
type PublicUser = Omit<User, 'password'>;
type UpdateUser = Partial<User>;
type CreateUser = Omit<User, 'id'> & { confirmPassword: string };
type UserKeys = keyof User;
type ReadonlyUser = Readonly<User>;

// ❌ INCORRECT
// Manually recreating utility types
type PublicUser = {
  id: string;
  name: string;
  email: string;
  // Manually omitting password
};
```

### Custom Utility Types
- ✅ **MUST** create reusable utility types for common patterns
- ✅ **MUST** document complex utility types
- ❌ **NEVER** create overly complex utility types

```typescript
// ✅ CORRECT
// Deep partial type for nested objects
type DeepPartial<T> = T extends object ? {
  [P in keyof T]?: DeepPartial<T[P]>;
} : T;

// Nullable type
type Nullable<T> = T | null | undefined;

// Async function type
type AsyncFn<TArgs extends any[] = any[], TReturn = void> = 
  (...args: TArgs) => Promise<TReturn>;

// Value of object type
type ValueOf<T> = T[keyof T];
```

## Type Imports/Exports

### Import Types
- ✅ **MUST** use `type` imports for type-only imports
- ✅ **MUST** separate type and value imports
- ✅ **MUST** use `import type` for circular dependency prevention
- ❌ **NEVER** mix type and value exports in the same statement

```typescript
// ✅ CORRECT
import type { User, Config } from './types';
import { processUser, validateConfig } from './utils';

export type { User };
export { processUser };

// ❌ INCORRECT
import { User, processUser } from './module'; // Mixed imports
export { User, processUser }; // Mixed exports
```

## Assertion Functions

### Type Assertions
- ✅ **MUST** validate before asserting types
- ✅ **MUST** use assertion functions for runtime validation
- ✅ **MUST** document why assertion is safe
- ❌ **NEVER** use `as any` as an escape hatch
- ❌ **NEVER** double cast (`as unknown as T`)
- ❌ **NEVER** disable TypeScript strictness without justification

```typescript
// ✅ CORRECT
function assertDefined<T>(value: T | null | undefined, message: string): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(message);
  }
}

function processUser(user: User | null) {
  assertDefined(user, 'User must be defined');
  // user is now typed as User
  console.log(user.name);
}

// ❌ INCORRECT
function processUser(user: unknown) {
  const typedUser = user as any as User; // Double cast
  console.log(typedUser.name);
}
```

## Common TypeScript Antipatterns

### Any Usage
- ❌ **NEVER** use `any` type
- ❌ **NEVER** use `@ts-ignore` or `@ts-expect-error` without justification
- ❌ **NEVER** disable type checking for files

```typescript
// ❌ INCORRECT - All of these are forbidden
let data: any = fetchData();
// @ts-ignore
const result = undefinedFunction();
// @ts-nocheck (at top of file)
```

### Type Casting Abuse
- ❌ **NEVER** cast to bypass type checking
- ❌ **NEVER** use assertions to "fix" type errors
- ❌ **NEVER** cast function parameters to avoid proper typing

### Index Signatures
- ✅ **MUST** prefer Map/Set over objects with index signatures
- ✅ **MUST** use `Record<K, V>` for simple key-value pairs
- ❌ **NEVER** use broad index signatures without validation

```typescript
// ✅ CORRECT
const userMap = new Map<string, User>();
const config: Record<string, string> = {
  apiUrl: 'https://api.example.com'
};

// ❌ INCORRECT
interface Data {
  [key: string]: any; // Too permissive
}
```

## TypeScript + Svelte 5 Integration

### Component Props Typing
- ✅ **MUST** use interface for component props
- ✅ **MUST** type `$props()` with explicit interface
- ✅ **MUST** type snippet parameters
- ✅ **MUST** type event handlers properly

```typescript
// ✅ CORRECT
interface Props {
  count: number;
  title?: string;
  onUpdate: (value: number) => void;
  children: Snippet;
  header?: Snippet<[string]>;
}

let { 
  count, 
  title = 'Default',
  onUpdate,
  children,
  header
}: Props = $props();
```

### Rune Typing
- ✅ **MUST** explicitly type `$state` for complex types
- ✅ **MUST** type `$derived` when type can't be inferred
- ✅ **MUST** type custom stores and state functions

```typescript
// ✅ CORRECT
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

// Explicit typing for complex state
let todos = $state<Todo[]>([]);
let filter = $state<'all' | 'active' | 'completed'>('all');

// Derived with explicit type when needed
let filteredTodos = $derived<Todo[]>(
  todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  })
);

// State function with proper types
export function createTodoStore() {
  let todos = $state<Todo[]>([]);
  
  return {
    get todos() { return todos; },
    add(text: string): void {
      todos = [...todos, { 
        id: crypto.randomUUID(), 
        text, 
        completed: false 
      }];
    },
    toggle(id: string): void {
      todos = todos.map(t => 
        t.id === id ? { ...t, completed: !t.completed } : t
      );
    }
  };
}
```

### Event Handler Typing
- ✅ **MUST** type event parameters explicitly
- ✅ **MUST** use proper HTML event types
- ✅ **MUST** type custom event detail

```typescript
// ✅ CORRECT
function handleClick(event: MouseEvent): void {
  console.log(event.clientX);
}

function handleInput(event: Event & { currentTarget: HTMLInputElement }): void {
  const value = event.currentTarget.value;
}

function handleSubmit(event: SubmitEvent): void {
  event.preventDefault();
  const form = event.currentTarget as HTMLFormElement;
  const formData = new FormData(form);
}

// Custom events
interface CustomEventDetail {
  value: string;
  timestamp: number;
}

function handleCustom(event: CustomEvent<CustomEventDetail>): void {
  console.log(event.detail.value);
}
```

## TypeScript Testing Patterns

### Test File Types
- ✅ **MUST** type test fixtures and mocks
- ✅ **MUST** use proper typing for test utilities
- ✅ **MUST** maintain type safety in tests
- ❌ **NEVER** use `any` in test files
- ❌ **NEVER** skip TypeScript checks in tests

```typescript
// ✅ CORRECT - Properly typed tests
import { describe, it, expect, vi } from 'vitest';
import type { User } from '$types/user';

// Typed mock data
const mockUser: User = {
  id: '123',
  name: 'Test User',
  email: 'test@example.com'
};

// Typed mock functions
const mockFetch = vi.fn<[string, RequestInit?], Promise<Response>>();

// Typed test utilities
function createMockResponse<T>(data: T): Response {
  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json' }
  });
}

describe('UserService', () => {
  it('should fetch user with proper types', async () => {
    mockFetch.mockResolvedValueOnce(
      createMockResponse<User>(mockUser)
    );
    
    const result = await fetchUser('123');
    expect(result).toEqual(mockUser);
  });
});

// ❌ INCORRECT - Lost type safety
const mockUser: any = { /* data */ };
const mockFetch = vi.fn(); // Untyped mock
```

### Component Test Types
- ✅ **MUST** type component props in tests
- ✅ **MUST** type DOM queries properly
- ✅ **MUST** use proper event types

```typescript
// ✅ CORRECT
import { render, fireEvent } from '@testing-library/svelte';
import type { RenderResult } from '@testing-library/svelte';
import Button from './Button.svelte';

describe('Button Component', () => {
  let component: RenderResult;
  
  beforeEach(() => {
    component = render(Button, {
      props: {
        label: 'Click me',
        disabled: false,
        onClick: vi.fn<[MouseEvent], void>()
      }
    });
  });
  
  it('should handle click events', async () => {
    const button = component.getByRole('button') as HTMLButtonElement;
    await fireEvent.click(button);
    expect(button.disabled).toBe(false);
  });
});
```

## TypeScript + Svelte Gotchas

### Common Type Issues
- ⚠️ **WARNING**: `$state` creates proxies - type may not match runtime
- ⚠️ **WARNING**: Event handlers need `currentTarget` not `target`
- ⚠️ **WARNING**: Store types differ from rune types
- ⚠️ **WARNING**: `$derived` type inference can fail with complex expressions

```typescript
// Common gotchas and solutions

// 1. Proxy vs Plain Object
interface User {
  name: string;
}
let user = $state<User>({ name: 'Alice' });
// user is a Proxy at runtime, but TypeScript sees User

// 2. Event target typing
function handleInput(e: Event) {
  // ❌ INCORRECT
  const value = (e.target as HTMLInputElement).value;
  
  // ✅ CORRECT
  const value = (e.currentTarget as HTMLInputElement).value;
}

// 3. Complex derived types
// Sometimes needs explicit typing
let complex = $derived<string[]>(() => {
  // Complex logic that TypeScript can't infer
  return someComplexComputation();
});

// 4. Snippet typing with generics
interface Props<T> {
  items: T[];
  renderItem: Snippet<[T]>;
}

// 5. Two-way binding - use $bindable() in destructuring
interface Props {
  value: string;  // Regular prop
  count: number;  // Will be made bindable in destructuring
}
let { value, count = $bindable() }: Props = $props();
```

### Module Resolution Issues
- ⚠️ **WARNING**: `.svelte.ts` files need special handling
- ⚠️ **WARNING**: Dynamic imports need explicit types
- ⚠️ **WARNING**: JSON imports require `resolveJsonModule`

```typescript
// ✅ CORRECT - Importing .svelte.ts modules
import { createStore } from './store.svelte.js'; // Note: .js extension
import type { StoreType } from './store.svelte.js';

// ✅ CORRECT - Dynamic imports with types
const module = await import('./module.js') as { default: ComponentType };
```

## TypeScript Performance

### Type Complexity
- ✅ **MUST** simplify overly complex type computations
- ✅ **MUST** avoid deeply nested conditional types
- ✅ **MUST** limit recursive type depth
- ❌ **NEVER** create infinite type recursion
- ❌ **NEVER** use excessive type computation

```typescript
// ❌ AVOID - Too complex
type DeepReadonly<T> = T extends any[] ? DeepReadonlyArray<T[number]> :
  T extends object ? DeepReadonlyObject<T> :
  T extends Function ? T :
  T;

// ✅ PREFER - Simpler approach
type SimpleReadonly<T> = Readonly<T>;
```

# Code Quality

## TypeScript-Specific Requirements
- ✅ **MUST** enable strict mode (never disable without justification)
- ✅ **MUST** define interfaces for all props
- ✅ **MUST** type all function parameters and returns
- ✅ **MUST** use generics for reusable components
- ❌ **NEVER** use `any` type
- ❌ **NEVER** ignore TypeScript errors
- ❌ **NEVER** use `//@ts-ignore`

## Code Style
- ✅ **MUST** follow Prettier configuration
- ✅ **MUST** use consistent naming conventions
- ✅ **MUST** keep components under 200 lines
- ✅ **MUST** extract complex logic to separate files
- ❌ **NEVER** commit commented-out code
- ❌ **NEVER** use console.log in production code

## Error Handling
- ✅ **MUST** handle all error states
- ✅ **MUST** provide user-friendly error messages
- ✅ **MUST** use error boundaries for component errors
- ✅ **MUST** log errors appropriately
- ❌ **NEVER** show raw error messages to users
- ❌ **NEVER** silently fail

# Security

## XSS Prevention
- ✅ **MUST** sanitize user input before rendering HTML
- ✅ **MUST** use DOMPurify or similar for sanitization
- ✅ **MUST** validate and escape user input
- ❌ **NEVER** use `{@html}` with unsanitized content
- ❌ **NEVER** trust user input

```svelte
<!-- ✅ CORRECT -->
<script>
  import DOMPurify from 'isomorphic-dompurify';
  let userContent = $state('');
  let sanitized = $derived(DOMPurify.sanitize(userContent));
</script>
{@html sanitized}

<!-- ❌ INCORRECT -->
{@html userContent} <!-- XSS vulnerability! -->
```

## Data Validation
- ✅ **MUST** validate all form inputs
- ✅ **MUST** validate API responses
- ✅ **MUST** use schema validation (Zod, Yup, etc.)
- ❌ **NEVER** trust client-side validation alone
- ❌ **NEVER** process unvalidated data

## Authentication & Authorization
- ✅ **MUST** check permissions server-side
- ✅ **MUST** use secure session management
- ✅ **MUST** implement CSRF protection
- ❌ **NEVER** store sensitive data in localStorage
- ❌ **NEVER** expose API keys in client code
