# React JS — Interview Notes

## 1. What is React?
React is a JavaScript library for building user interfaces using reusable components and declarative rendering.

**Memory:** React = Components + State + Props + Declarative UI

## 2. Why React?
- Component-based architecture
- Reusable UI pieces
- Declarative programming model
- Efficient UI updates
- Strong ecosystem
- Hooks for state and side effects

## 3. Component
A component is a reusable unit of UI and behavior.

### Functional component
```jsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>;
}
```

Prefer function components and Hooks in modern React code.

## 4. Props vs State

| Props | State |
|---|---|
| Passed by parent | Managed by component/owner |
| Read-only from child perspective | Can change over time |
| Used to configure a component | Represents changing UI data |
| Parent -> child data flow | Triggers re-render when updated |

**Memory:** Props = input, State = internal/changeable data.

## 5. One-way Data Flow
React generally follows one-way data flow:

**Parent -> Child via props**

A child can communicate upward by receiving a callback function as a prop and invoking it.

## 6. JSX
JSX is a syntax extension that lets you describe UI using HTML-like syntax inside JavaScript.

```jsx
const element = <h1>Hello</h1>;
```

JSX is transformed into JavaScript expressions by the build tooling.

## 7. Virtual DOM
The Virtual DOM is an in-memory representation of UI that React uses as part of its rendering process.

React compares the new rendered tree with the previous one and applies the necessary updates to the host environment.

**Interview line:**
> React does not blindly update the entire DOM; it determines the required changes and commits them.

## 8. Reconciliation
Reconciliation is the process React uses to compare rendered output and determine what should change.

Important ideas:
- Element type matters
- `key` helps identify list items
- Component identity affects whether state is preserved/reset

## 9. Keys in Lists ⭐
Keys help React identify list items between renders.

```jsx
users.map(user => (
  <User key={user.id} user={user} />
));
```

### Why avoid array index as key?
If list order changes, an index key can associate the wrong state/component instance with an item.

**Memory:** Key = stable identity.

## 10. State with `useState`

```jsx
const [count, setCount] = useState(0);
```

Use functional updates when the new state depends on the previous state:

```jsx
setCount(prev => prev + 1);
```

## 11. State is a Snapshot
A render sees a snapshot of state for that render. Calling a state setter schedules an update; it does not mutate the current render's state variable in place.

## 12. `useEffect` ⭐⭐⭐
Used to synchronize a component with external systems or perform side effects.

```jsx
useEffect(() => {
  const id = setInterval(() => {
    // side effect
  }, 1000);

  return () => clearInterval(id);
}, []);
```

### Dependency array
- No dependency array -> effect runs after every render
- `[]` -> runs after initial mount for a given mount lifecycle
- `[a, b]` -> reruns when dependencies change

### Cleanup
Cleanup runs before the effect is re-run and when the component is unmounted.

**Important:** Do not use `useEffect` for calculations that can be derived directly during render.

## 13. `useEffect` vs `useLayoutEffect`
- `useEffect` is for effects that do not need to block browser painting.
- `useLayoutEffect` runs synchronously after DOM mutations and before the browser paints.

Use `useLayoutEffect` only when you need layout measurement or a pre-paint DOM adjustment.

## 14. `useMemo`
Memoizes a calculated value.

```jsx
const filtered = useMemo(() => {
  return products.filter(p => p.price > 1000);
}, [products]);
```

Use it for an expensive calculation when memoization is actually useful.

**Memory:** `useMemo` = memoized value.

## 15. `useCallback`
Memoizes a function reference.

```jsx
const handleClick = useCallback(() => {
  save(id);
}, [id]);
```

Useful mainly when function identity affects child rendering or another dependency.

**Memory:** `useCallback` = memoized function.

## 16. `React.memo`
Memoizes a component so React can skip a re-render when its props are considered unchanged.

```jsx
const User = React.memo(function User({ name }) {
  return <div>{name}</div>;
});
```

It does not prevent every possible re-render and does not stop updates caused by the component's own state or consumed context.

## 17. `useRef`
Stores a mutable value that persists across renders without causing a re-render when changed.

Also used to access DOM nodes.

```jsx
const inputRef = useRef(null);

<input ref={inputRef} />
```

**Memory:** `useState` -> render trigger; `useRef` -> persistent mutable value without render trigger.

## 18. `useContext`
Reads a value from a React context.

```jsx
const theme = useContext(ThemeContext);
```

Useful for values shared across a component subtree, such as theme, locale, or authenticated-user context.

## 19. Context API
Context helps avoid passing the same data through many intermediate components (prop drilling).

Use it carefully because consumers can re-render when the context value changes.

## 20. Prop Drilling
Passing props through intermediate components that do not need the data themselves.

Solutions can include:
- Component composition
- Context
- State-management libraries when the problem is broader

## 21. Controlled vs Uncontrolled Components

### Controlled
Form data is controlled by React state.

```jsx
<input value={name} onChange={e => setName(e.target.value)} />
```

### Uncontrolled
DOM keeps the value; access it using a ref when required.

```jsx
<input defaultValue="John" ref={inputRef} />
```

**Memory:** Controlled = React owns value; Uncontrolled = DOM owns value.

## 22. Forms
Common interview topics:
- Controlled fields
- Validation
- Submission handling
- Error display
- Resetting forms
- Dynamic fields

## 23. Event Handling
React event handlers are passed as functions:

```jsx
<button onClick={handleClick}>Save</button>
```

Do not call the handler during render:

```jsx
// Wrong for an event handler
<button onClick={handleClick()}>Save</button>
```

## 24. Conditional Rendering
Common patterns:

```jsx
{isLoggedIn && <Dashboard />}
```

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

## 25. Lifting State Up ⭐
When multiple sibling components need the same state, move the state to their nearest common parent and pass the value/callbacks down.

**Memory:** Shared state -> move up.

## 26. Derived State
Avoid storing data in state if it can be calculated from existing props/state.

Bad idea:
```text
firstName + lastName stored separately as fullName state
```

Better:
```jsx
const fullName = `${firstName} ${lastName}`;
```

## 27. Hooks Rules ⭐⭐⭐
Hooks must:
1. Be called at the top level of a React function component or custom Hook.
2. Not be called conditionally, inside loops, or inside nested functions.
3. Be called consistently so React can preserve Hook state correctly.

## 28. Custom Hooks
A custom Hook extracts reusable stateful logic.

```jsx
function useOnlineStatus() {
  const [online, setOnline] = useState(true);
  // reusable logic
  return online;
}
```

A custom Hook name conventionally starts with `use`.

## 29. Rules of Hooks — Interview Trick
**Do not do:**
```jsx
if (loggedIn) {
  useEffect(() => {});
}
```

**Do:**
```jsx
useEffect(() => {
  if (loggedIn) {
    // logic
  }
}, [loggedIn]);
```

## 30. State Batching
React may batch multiple state updates so that related updates can be processed together, reducing unnecessary renders.

When updates depend on previous state, use updater functions:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
```

## 31. Functional State Update
Use this when next state depends on previous state:

```jsx
setCount(prev => prev + 1);
```

This avoids relying on a stale captured value.

## 32. Stale Closures ⭐⭐⭐
A closure can capture values from an older render.

Common fixes:
- Correct effect dependencies
- Functional state updates
- Refs when appropriate

## 33. Component Lifecycle with Function Components
Think in terms of:
- Render
- Commit
- Effects/cleanup
- Re-render
- Unmount

Modern React interviews should prefer Hooks and render/commit concepts rather than only class lifecycle methods.

## 34. Class Lifecycle — Legacy/Still Asked
Know these names:
- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

Also know `componentDidCatch` / error boundaries.

## 35. Error Boundaries ⭐⭐⭐
Error boundaries catch rendering/lifecycle errors in descendant components and display fallback UI.

Traditionally implemented with class components.

They do not catch every kind of error, such as event-handler errors or arbitrary asynchronous errors.

## 36. `useReducer`
Useful when state transitions are more complex or related updates can be expressed as actions.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

**Memory:** Many related state transitions -> consider reducer.

## 37. `useState` vs `useReducer`
- `useState` -> simple/local state
- `useReducer` -> complex transitions/action-based updates

## 38. React Rendering
A state/prop/context update can trigger a render. Rendering means React calculates what the UI should look like; the commit phase applies required host environment changes.

**Important:** A render does not necessarily mean the DOM changes.

## 39. Re-render vs DOM Update
A component can re-render while React determines that no DOM change is necessary.

**Memory:** Render != DOM update.

## 40. Referential Equality
Objects, arrays, and functions created during render usually have new references.

This matters for:
- `React.memo`
- dependency arrays
- `useMemo`
- `useCallback`

## 41. Immutability ⭐⭐⭐
Do not mutate state directly.

Wrong:
```jsx
user.name = 'Alex';
```

Better:
```jsx
setUser(prev => ({ ...prev, name: 'Alex' }));
```

For arrays:
```jsx
setItems(prev => [...prev, newItem]);
```

**Memory:** New reference -> predictable update detection.

## 42. Why not mutate state?
Direct mutation can make change detection and reasoning about state transitions unreliable and can break memoization assumptions.

## 43. Reconciliation and Component Identity
Changing a component's type or key can cause React to treat it as a different identity and reset its state.

Stable component structure and keys help preserve state where intended.

## 44. `key` and State Reset
Changing a key can intentionally reset a component's state:

```jsx
<Form key={userId} />
```

When `userId` changes, React can treat it as a new component identity.

## 45. Fragment
A Fragment groups multiple elements without adding an extra DOM node.

```jsx
<>
  <Header />
  <Content />
</>
```

## 46. Composition
React favors composition for building reusable components.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

**Memory:** Composition over unnecessary inheritance.

## 47. Higher-Order Component (HOC)
A function that takes a component and returns an enhanced component.

```jsx
const Enhanced = withAuth(User);
```

HOCs are still relevant for understanding existing code, although Hooks are often preferred for reusable logic in modern React.

## 48. Render Props
A pattern where a component receives a function prop used to render or determine content.

```jsx
<DataProvider render={data => <View data={data} />} />
```

Know it mainly for legacy/existing code and interview questions.

## 49. Hooks vs HOC vs Render Props
- Hooks -> reusable logic in modern function components
- HOC -> component enhancement/wrapping
- Render props -> function-based sharing of rendering logic

## 50. `useEffect` Infinite Loop
Common reason:
```jsx
useEffect(() => {
  setValue({});
}, [value]);
```

If the effect changes its own dependency every time, it can loop.

Fix the dependency/logic and avoid unnecessary object recreation.

## 51. Dependency Array
Dependencies should represent values used by the effect that can change and affect its behavior.

Do not blindly add/remove dependencies just to silence a warning; understand the synchronization being modeled.

## 52. Fetching Data in React
Typical flow:
1. Trigger request
2. Track loading
3. Store data
4. Track error
5. Cleanup/cancel where appropriate
6. Render state

Example pattern:
```jsx
useEffect(() => {
  let cancelled = false;

  fetch('/api/users')
    .then(r => r.json())
    .then(data => {
      if (!cancelled) setUsers(data);
    })
    .catch(err => {
      if (!cancelled) setError(err);
    });

  return () => {
    cancelled = true;
  };
}, []);
```

For production applications, use a data-fetching/cache strategy appropriate to the application rather than putting every remote-data problem into raw effects.

## 53. API Calls and Axios/Fetch
Know:
- HTTP methods
- headers
- authentication token handling
- timeout/error handling
- cancellation
- retry strategy
- response normalization

## 54. State Management
React itself provides local state and context. For larger applications, external state libraries may be used depending on requirements.

Be ready to discuss:
- local state
- lifted state
- Context
- Redux-style global state
- server-state/data-fetching libraries

## 55. Redux — Interview Basics ⭐⭐⭐
Core concepts:
- Store
- Action
- Reducer
- Dispatch
- Selector
- Middleware

Flow:
```text
UI -> dispatch(action) -> reducer -> new state -> UI
```

## 56. Redux Reducer
A reducer calculates the next state from current state and an action.

Conceptually:
```js
nextState = reducer(currentState, action);
```

Keep state transitions predictable and side-effect free.

## 57. Redux Middleware
Middleware can intercept actions and is commonly used for logging, async workflows, analytics, or other cross-cutting concerns.

## 58. Global State vs Server State
- Global/client state -> UI preferences, local workflow state, etc.
- Server state -> remote data with caching, freshness, synchronization, loading/error states

Do not put everything into one global store by default.

## 59. Performance Optimization ⭐⭐⭐⭐
Know:
- `React.memo`
- `useMemo`
- `useCallback`
- list keys
- code splitting
- lazy loading
- virtualization for very large lists
- avoid unnecessary state
- avoid unnecessary effects
- stable references where useful
- profiling before optimizing

**Interview line:**
> Optimize based on measured bottlenecks, not by wrapping everything in memoization.

## 60. `React.lazy`
Used for lazy-loading a component.

```jsx
const Admin = lazy(() => import('./Admin'));
```

Usually paired with `Suspense` for the loading UI.

## 61. `Suspense`
Suspense lets a component tree display a fallback while something is not yet ready for the supported Suspense mechanism being used.

```jsx
<Suspense fallback={<Spinner />}>
  <Admin />
</Suspense>
```

Know it in the context of lazy loading and modern async rendering/data patterns.

## 62. Code Splitting
Split the application into smaller bundles that can be loaded when required.

Benefits:
- Lower initial JavaScript cost
- Faster initial load for large applications

## 63. Bundle Optimization
Interview topics:
- tree shaking
- code splitting
- lazy loading
- compression
- caching
- avoiding unnecessary dependencies

## 64. Debouncing vs Throttling ⭐⭐⭐
### Debounce
Run after input stops for a specified period.

Use cases:
- search box
- autocomplete

### Throttle
Allow execution at most once per interval.

Use cases:
- scroll
- resize
- mouse movement

**Memory:** Debounce = wait for silence; Throttle = limit frequency.

## 65. Accessibility (a11y)
Know:
- semantic HTML
- labels for inputs
- keyboard navigation
- focus management
- ARIA only when needed
- meaningful alt text

## 66. Security
Frontend security interview topics:
- XSS
- CSRF
- token storage considerations
- input validation
- output encoding
- Content Security Policy
- secure cookies
- CORS understanding

Never treat client-side validation as a replacement for server-side validation/authorization.

## 67. CORS
CORS is a browser security mechanism controlling whether a web page origin can access resources on another origin.

It is enforced by browsers and configured primarily on the server.

## 68. Authentication vs Authorization
- Authentication -> Who are you?
- Authorization -> What are you allowed to do?

Typical flow:
```text
Login -> authenticate -> token/session -> request -> authorize -> response
```

## 69. JWT in React
Frontend may receive an access token and attach it to API requests according to backend security requirements.

```http
Authorization: Bearer <token>
```

Security design depends on token type, storage, refresh strategy, XSS/CSRF risk, and backend architecture.

## 70. Routing
Know common routing concepts:
- client-side routing
- nested routes
- route parameters
- query parameters
- protected routes
- redirects
- lazy-loaded routes

## 71. SPA vs MPA
### SPA
Most navigation is handled in the browser after initial application load.

### MPA
Pages are commonly loaded as separate server responses/documents.

## 72. SSR vs CSR
### CSR
Browser loads JavaScript and renders much of the UI on the client.

### SSR
Server produces HTML for a request, after which client-side JavaScript can hydrate/attach behavior.

Know the trade-offs:
- SEO
- initial load
- server cost
- caching
- interactivity

## 73. Hydration
Hydration is the process of attaching client-side React behavior to server-rendered HTML so it can become interactive.

## 74. React Server Components — Concept
Server Components allow some components to execute on the server and send a representation that reduces the need to ship their implementation to the client.

This is usually discussed in frameworks that support the feature rather than in a plain client-only React setup.

## 75. Client Component vs Server Component
Exact behavior depends on the framework/runtime supporting Server Components. Interview-level idea:
- Server components can access server-side resources and reduce client JavaScript for that component.
- Client components are needed for browser-only interactivity/state/hooks such as `useState` and `useEffect`.

## 76. Environment Variables
Typical frontend build systems expose only explicitly permitted environment variables to client code.

**Critical:** Never put secrets such as private API keys or database credentials in browser-delivered JavaScript.

## 77. Error Handling
Handle:
- API errors
- validation errors
- loading states
- empty states
- network failures
- fallback UI

Do not show raw backend/internal errors directly to users.

## 78. Loading / Error / Empty States
A production component should commonly consider:
```text
loading -> success -> empty
                 \-> error
```

This is a common interview scenario.

## 79. Testing React
Know the difference between:
- Unit tests
- Component/integration tests
- End-to-end tests

Important testing principles:
- Test user-visible behavior
- Test accessibility where appropriate
- Avoid tests tightly coupled to implementation details

## 80. Common React Testing Questions
Be ready for:
- how to test a component
- how to mock API calls
- how to test user interactions
- how to test loading/error states
- when to use unit vs integration vs E2E tests

## 81. Common Interview Coding Questions
Practice:
1. Counter with increment/decrement
2. Search/filter list
3. Debounced search box
4. Todo application
5. Pagination
6. Infinite scroll
7. Modal
8. Accordion
9. Tabs
10. Form validation
11. OTP input
12. Autocomplete
13. Stopwatch/timer
14. Shopping cart
15. Login form
16. Fetch API and handle loading/error
17. Parent-child communication
18. Dynamic form fields
19. Protected route
20. Custom Hook

## 82. Pagination
Know the difference:
- Page-number pagination
- Cursor-based pagination
- Server-side pagination
- Client-side pagination

For large datasets, server-side pagination is usually preferred.

## 83. Infinite Scroll
Typical approach:
1. Load first page
2. Detect near-bottom/intersection
3. Fetch next page
4. Append results immutably
5. Handle loading/error/end-of-data

`IntersectionObserver` is commonly useful.

## 84. Virtualization
For very large lists, render only the visible range rather than thousands of DOM nodes at once.

## 85. Optimistic UI
Update the UI immediately before server confirmation, then reconcile success/failure.

Useful for fast-feeling interactions, but requires rollback/error strategy.

## 86. React Strict Mode
Strict Mode enables additional development-time checks and may intentionally re-run certain logic in development to help detect unsafe patterns.

Do not interpret development-only repeated behavior as proof that production will behave identically.

## 87. Why Strict Mode Seems to Run Effects Twice?
In development, Strict Mode can intentionally exercise mount/cleanup behavior to surface effects that are not properly synchronized or cleaned up.

## 88. React Fiber — Interview Concept
Fiber is React's internal architecture for representing and scheduling rendering work.

Interview-level takeaway:
> Fiber helps React break rendering work into units and schedule/coordinate updates.

Avoid claiming that every update is automatically executed on a separate thread.

## 89. Concurrent Rendering — Concept
Modern React can prepare rendering work in a way that allows urgent and non-urgent work to be prioritized and interrupted where supported.

**Important:** Concurrent rendering is not the same thing as making JavaScript execute simultaneously on multiple CPU threads.

## 90. `startTransition` / `useTransition`
Used to mark updates as non-urgent so urgent interactions can stay responsive.

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSearchResults(results);
});
```

## 91. `useDeferredValue`
Lets a non-urgent value lag behind a more urgent one so expensive UI work can be deferred.

Useful for keeping input interactions responsive while rendering expensive results.

## 92. `useId`
Generates stable IDs suitable for associating elements such as labels and inputs, especially useful for accessibility.

It should not be used as a list key.

## 93. `useImperativeHandle`
Customizes the value exposed through a ref from a component, usually in advanced reusable component scenarios.

## 94. `forwardRef`
Historically used to pass refs through a component to a child DOM node/component. Know it mainly for existing code and API compatibility; newer React patterns may differ depending on the React version/framework in use.

## 95. Portal
A portal renders children into a different DOM subtree while preserving React ownership/context relationships.

Common use cases:
- Modal
- Tooltip
- Overlay

```jsx
createPortal(children, domNode)
```

## 96. Refs vs State
**State:** UI-affecting data; updates trigger re-render.

**Ref:** mutable persistent value; changing `.current` does not trigger re-render.

## 97. `useMemo` vs `useCallback`
```text
useMemo     -> memoize a calculated value
useCallback -> memoize a function reference
```

## 98. `React.memo` vs `useMemo`
```text
React.memo -> memoize component rendering based on props
useMemo    -> memoize a value inside a component
```

## 99. `useEffect` vs `useMemo`
- `useEffect` -> synchronize with external systems / side effects
- `useMemo` -> cache a calculation

Do not use `useMemo` as a general replacement for effects.

## 100. Common Mistakes
- Mutating state directly
- Missing/incorrect effect dependencies
- Using array index as key for reorderable lists
- Overusing Context
- Overusing `useMemo`/`useCallback`
- Putting secrets in frontend code
- Treating client validation as security
- Ignoring loading/error/empty states
- Creating effects for derivable values
- Unnecessary global state
- Depending on implementation details in tests

# Rapid-Fire Interview Answers

### What is React?
> A JavaScript library for building component-based user interfaces using declarative rendering.

### Props vs State?
> Props are inputs passed into a component; state is data managed by the component/owner that can change over time.

### What is JSX?
> A syntax extension used to describe UI in JavaScript.

### What is Virtual DOM?
> An in-memory representation of UI used during React's rendering/reconciliation process.

### What is reconciliation?
> React's process of comparing rendered output and determining the necessary updates.

### Why are keys needed?
> Keys provide stable identity for list items so React can correctly reconcile them.

### `useState`?
> Adds state to a function component and returns the current state plus a setter.

### `useEffect`?
> Synchronizes a component with external systems and handles side effects/cleanup.

### `useRef`?
> Stores a persistent mutable value or DOM reference without causing a render when `.current` changes.

### `useMemo`?
> Memoizes a calculated value.

### `useCallback`?
> Memoizes a function reference.

### `React.memo`?
> Lets React skip rendering a memoized component when its props have not meaningfully changed.

### Controlled component?
> Its form value is controlled by React state.

### Context?
> Provides values to a component subtree without passing props through every intermediate component.

### What is lifting state up?
> Moving shared state to the nearest common parent.

### What is a custom Hook?
> A reusable function that uses Hooks to share stateful logic.

### What is code splitting?
> Loading JavaScript in smaller chunks instead of sending the entire application bundle initially.

### What is hydration?
> Attaching client-side behavior to server-rendered HTML.

### Debounce vs throttle?
> Debounce waits until activity stops; throttle limits how frequently a function runs.

### Render vs DOM update?
> A render computes the next UI; React may determine that no DOM change is necessary.

# Scenario-Based Interview Questions

## 1. Search box is making an API call on every keystroke. What will you do?
Use debouncing, cancel/ignore stale requests, show loading/error states, and consider a server-side search strategy.

## 2. Large list is slow. How will you optimize?
Profile first, then consider virtualization, pagination, memoization where useful, stable keys, and reducing unnecessary renders.

## 3. Child re-renders too often. What will you check?
Check changing object/function references, parent updates, context changes, and whether `React.memo` plus stable props is actually beneficial.

## 4. API data is needed by many pages. Where should it live?
Treat it as server state and use an appropriate caching/data-fetching architecture rather than automatically placing it into generic global UI state.

## 5. Form value is not updating correctly. What will you check?
Check controlled vs uncontrolled usage, `value`/`defaultValue`, event handler, state update, and whether state is being mutated directly.

## 6. Component loses state unexpectedly. What will you check?
Check whether its type or `key` changed, whether its position in the tree changed, and whether the component is being remounted.

# Ultimate Memory Map

```text
REACT
│
├── Core
│   ├── Components
│   ├── JSX
│   ├── Props
│   ├── State
│   └── One-way data flow
│
├── Rendering
│   ├── Render
│   ├── Commit
│   ├── Reconciliation
│   ├── Keys
│   └── Component identity
│
├── Hooks
│   ├── useState
│   ├── useEffect
│   ├── useRef
│   ├── useContext
│   ├── useMemo
│   ├── useCallback
│   ├── useReducer
│   ├── useTransition
│   ├── useDeferredValue
│   ├── useId
│   └── Custom Hooks
│
├── Forms
│   ├── Controlled
│   ├── Uncontrolled
│   └── Validation
│
├── State
│   ├── Local
│   ├── Lifted
│   ├── Context
│   └── Global/Server State
│
├── Performance
│   ├── memo
│   ├── memoization
│   ├── Code splitting
│   ├── Lazy loading
│   ├── Virtualization
│   └── Profiling
│
├── Advanced
│   ├── Suspense
│   ├── Portals
│   ├── Error Boundaries
│   ├── SSR/CSR
│   ├── Hydration
│   └── Concurrent rendering concepts
│
└── Production
    ├── Security
    ├── Accessibility
    ├── Testing
    ├── API integration
    ├── Routing
    └── Error/Loading/Empty states
```

# 2-Minute Final Revision

> React is a component-based JavaScript library for declarative UI development. Props are inputs from a parent, while state represents changing data. React follows one-way data flow. JSX describes the UI. During updates, React renders and reconciles the result, then commits the required host changes. Keys provide stable identity for list items. Hooks such as `useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, `useContext`, and `useReducer` solve common state, effect, reference, memoization, context, and state-transition problems. Avoid direct mutation, unnecessary effects, unstable keys, and premature memoization. For production systems, also know routing, API handling, authentication, CORS, accessibility, performance, testing, code splitting, SSR/CSR, hydration, and error/loading/empty states.
