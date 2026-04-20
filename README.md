# React Performance 2026

## React Performance Labs: Running Examples

This project includes multiple React examples demonstrating performance optimization techniques with. To switch between examples:

```bash
npm run examples
```

This will launch an interactive menu where you can:

- Use **↑↓ arrow keys** to navigate through available examples
- Press **Enter** to select an example
- Press **Ctrl+C** or **Esc** to quit

The CLI will automatically update `index.html` to load the selected example. After selecting, run `npm run dev` to start the development server.

Once you've selected an example, start the server:

```bash
npm run dev
```

## Available Examples

We won't use _all_ of the examples today, but we have a number to choose from depending on where our conversation leads us.

### React Hooks

- **Effect Essentials** - Common useEffect pitfalls and how to fix them (dependencies, race conditions, cleanup)

### Performance Optimization

- **Deep Thoughts** - Performance optimization with useCallback, React.memo, and useReducer
- **Memo Mania** - Deep dive into memoization (useMemo, React.memo, useCallback)
- **Throwing Shapes** - useDeferredValue for responsive UI with expensive rendering

### State Management

- **Local State** - State colocation principles and best practices
- **Anti-Social Network** - Optimistic UI updates with useOptimistic
- **Contextual** - Context API re-render problems and solutions

### Concurrent Features

- **Prime Time** - useTransition vs useDeferredValue for concurrent rendering
- **Pokamoka** - Pokemon filtering with useTransition

### Code Splitting & Loading

- **Lazy Loading** - React.lazy() and Suspense for code splitting
- **Suspenseful** - Modern data fetching with Suspense and the use() hook (React 19)

# NOTES

<details>
<summary><strong>React Performance Principles &amp; Hard Problems (click to expand)</strong></summary>

<br />

### Golden Rules of React Performance

Not doing stuff is faster than doing stuff.

Code that doesn't have to run is way faster than code that has to run. Every step closer to never re-rendering makes an application more performant.

- Memoization and caching are themselves doing something.  
  If the cost of caching saves you from having to do more stuff, then it works.

  But if you're doing more stuff than it saves you, then it doesn't work and can actually be slower than not doing it at all.

---

### Second Rule for React Performance

Feeling fast is pretty much as good as actually being fast.  
This means techniques like preloading stuff or optimistically updating the UI when the server hasn't responded yet don't technically make things faster, BUT they make the application feel faster to the user, which is equally valuable.

---

### The Two Hardest Problems in Computer Science

The two hardest problems are naming things and cache invalidation.

Cache invalidation is particularly problematic because if something isn't loading because it thinks it's already memoized when it has actually changed, these bugs are incredibly hard to solve.

They're often caused by overusing `React.memo` on everything, which makes the system check unnecessarily if something needs to be done, making it slower than not doing it at all.

---

### The Purpose of React's Transition APIs (`useTransition`, `useDeferredValue`)

These APIs allow developers to specify what is important and urgent versus what can wait.  
They enable prioritizing critical work first and deferring less important updates, solving the problem of doing everything with equal priority.  
They can solve specific classes of performance problems with relatively simple code, often just a few lines.

</details>

## Anatomy of a Re-render

State typically changes for 1 of 3 reasons

- STATE CHANGED

- CONTEXT CHANGED

- It's PARENT CHANGED || Its props changed

### React's Rendering Cycle - How everything goes down.

Something changed, typically triggered by useState or useReducer

RENDER PHASE => COMMIT PHASE => CLEAN UP PHASE

- There're REALLY ONLY 2 TYPES OF STATE CHANGES:

1. Necessary
2. UN-necessary

Necessary RE-RENDERS come in 2 flavors:

1. URGENT
2. NON-URGENT

## SOME THEMES

- for web performance... and life.

- Not doing stuff is _way faster_ than doing stuff. _(Component hierarchy & state management)_
- Seeing if you can skip doing stuff is _sometimes_ less work than doing stuff. _(Memoization + Compiler)_
- You can _put off_ doing stuff. _(Suspense + Transitions)_
- Load as much as you need and as little as you can get away with. _(Lazy loading + bundle optimization)_

## React Fiber

React Fiber is a cooperatively scheduled rendering engine that changes how React figures out what changes to make to the DOM. It allows React to be smarter about rendering by being able to pause, interrupt, or restart work based on priority, rather than blocking the main thread until completion.

React Fiber enables React to distinguish between urgent and non-urgent updates by checking in periodically to determine if the current work is still valuable or if there are more important tasks to handle. It can pause or stop less important work to handle urgent updates, then either resume or restart the previous work.

### Early versions of React

Early versions of React would start at the top of the component tree and go through the entire tree in a blocking manner. Once the process started, it would block the main thread and run to completion, regardless of what happened, which could cause performance issues in larger applications

### React Fiber help improve Performance even when not explicitly used

React Fiber comes up for air regularly (approximately every 16.6 milliseconds to match the browser's 60 frames per second) to allow other operations in the application to execute, such as CSS animations, promise resolutions, and other JavaScript operations. This prevents blocking the main thread and improves overall application responsiveness.

### React Fiber sometimes do more work but still feel faster to users?

React Fiber may technically do more work by stopping current tasks to respond to higher-priority updates and then having to redo some work. However, this approach feels faster to users because it prioritizes responding to important interactions immediately rather than completing less important work first, even though the total computational work may be greater.

React Fiber aims to check approximately every 5 milliseconds to determine if it should pause and let other things happen. However, this timing can vary if there's something expensive happening that blocks it.

## Rendering Phase

If something MORE IMPORTANT comes along, it can turn its focus and put the LESS important stuff on PAUSE or TOSS IT OUT for now, and start over later.

Inside of each RENDER there are 2 trees.

### One thing at a time - The rendering phase

- Take the NEXT Fiber

- Run beginWork(fiber, renderLanes) -- Basically CALL the component function, derive the children.

- If there are children, descend. If not, bubble up via `completeWork` to finalize the node, update the DOM, collect any effects.

- Along the way - ask SHOULD I YIELD? -- yes? Pause for a moment and let the browser have the wheel back for a second.

- PICK UP where you leff off

### two trees that React Fiber maintains during the rendering process?

React Fiber maintains the current tree (which is what is referenced in the DOM) and a work-in-progress tree (which is what React is in the middle of updating).

This allows React to swap back to the current tree if it needs to abandon work in progress when something more important comes along.

### What happens to work-in-progress when React Fiber determines that something more important needs to be processed?

When something more important comes along, React Fiber can abandon the work-in-progress tree and swap back to the current tree (which is not half-updated). It can then either pick up where it left off later or start over if new state changes have made the previous work irrelevant.

## Commit phase

The render phase is when React figures out what the changes meant by working with the virtual DOM.

The commit phase is when React actually makes those changes happen to the real DOM.

## the fundamental rule for managing state placement in React applications to avoid unnecessary re-renders?

Keep state as HIGH as you need it and as LOW as you can get away with.

Push state down to the lowest possible component level where it's actually needed, rather than keeping it at higher levels in the component tree.

## What are the two main options for addressing performance issues caused by deeply nested component trees with shared state?

Either push all the state down to lower-level components where it's actually needed, or flatten the component tree by inlining components instead of having them as separate nested components.

The browser paints every 16.6 milliseconds. If you can get all your React rendering work done in under 16 milliseconds, the performance impact is imperceptible to users, meaning optimizations beyond that threshold may not be worthwhile

## What approach should be taken when deciding whether to optimize React component performance?

Measure first to identify actual problems, then solve significant issues while keeping an eye on potential future problems.

Don't over-optimize imperceptible performance issues, but also don't ignore patterns that could become major problems as the codebase grows over time.

### the lowest risk and most effective strategy for preventing unnecessary re-renders in React?

Pushing state down to where it's needed is the easiest, most effective, and lowest risk strategy.

This approach keeps state local to the components that actually need it, preventing unnecessary re-renders in unrelated parts of the component tree.

# MEMOIZATION

## the difference between useMemo and useCallback hooks in React?

useMemo is used to memoize a computed value - if none of the dependencies have changed, it returns the cached value instead of recomputing it. useCallback is used to memoize a function definition - if none of the dependencies have changed, it returns the same function reference instead of creating a new one. This prevents unnecessary function recreation and helps avoid garbage collection overhead.

The three main memoization tools are: useCallback (a hook for memoizing functions), useMemo (a hook for memoizing expensive computations), and React.memo (a higher-order component that wraps a component to prevent unnecessary re-renders).

#### a pure component in React, and why does this concept matter for memoization?

A pure component is one where the same inputs (props) always produce the same outputs. This means the component's output is determined solely by its props, without side effects from hooks like useState or useContext. This matters for memoization because React.memo only works effectively with pure components - components with internal state can change independently of their props, making memoization ineffective.

## React Compiler

React Compiler analyzes code at BUILD time to automatically identify and apply performance optimizations,

specifically handling memoization that developers would otherwise need to implement manually using hooks like useMemo, useCallback, and React.memo.

### How does React Compiler's approach to memoization differ from traditional React.memo in terms of efficiency?

React Compiler uses simpler, cheaper abstractions by storing direct value references in arrays and performing direct equality checks, rather than instantiating new functions and calling comparison methods.

It checks stored references directly, avoiding the overhead of function calls and object comparisons that React.memo requires.

The "use compiler" directive is a string literal placed at the top of a file to opt components into React Compiler.

It's designed as a string because it doesn't affect runtime code for anything that doesn't recognize it, making it backward compatible with legacy code. This pattern originates from the ES5 days and allows new tooling to detect and process it without breaking existing code.

# TRANSITION

## 2 more hooks

- startTransition() -- is used when TRIGGERING an UPDATE (i.e setState) in an EVENT HANDLER

Give you the ability to say like: "THIS thing we're going to do is LOW PRIORITY", THEN pass it a function and navigate through that.

- useDeferredValue() -- used when receiving NEW data from parent component (or earlier hook in the same component)

Changing of THIS VALUE is going to rtrigger something REALLY EXPENSIVE

## Differences between 2 hooks

useTransition is used when you control the code and allows you to pass a function that marks certain operations as low priority.

useDeferredValue is used when you don't control the code and works with a value that may trigger expensive operations, allowing you to defer updates to that value.

## In React Fiber's prioritization system, what does marking something as a transition indicate?

Marking something as a transition indicates that it is not urgent and can be placed in a lower priority lane.

This allows React to prioritize more urgent updates, like user input, while still processing the less urgent work when resources are available.

## the benefit of separating urgent from non-urgent updates in React applications?

Separating urgent from non-urgent updates allows the application to feel more responsive by prioritizing immediate user interactions (like input and hover events) while deferring expensive operations that can wait.

This prevents the UI from freezing or lagging during user interactions.

## When should you choose useTransition over useDeferredValue?

You should reach for useTransition first when you control the code.

If that doesn't fit due to constraints where you only have access to a value and don't have total control of the situation, then use useDeferredValue.

## Why is it important to prioritize reflecting user input back to them in React applications?

Reflecting user input back to them is high priority because if the UI starts lagging and freezing, preventing users from seeing the results of their actions (like typing, hovering, or animations), it creates an infuriating user experience.

Users expect immediate feedback from their interactions.

## What is the target time per frame to achieve 60 frames per second in a web application?

16.6 milliseconds per frame is needed to achieve 60 frames per second.

## What is the main benefit of React Fiber's architecture when handling user interactions during expensive computations?

React Fiber allows high-priority interactions (like keystrokes) to be processed immediately without waiting for lower-priority tasks (like searches) to complete.

It can interrupt ongoing work when new, higher-priority events occur.

## Why might showing a loading indicator be better than no feedback during a long-running operation?

A loading indicator provides visual feedback that the application is working on the task, which is better than no response at all.

No feedback makes the application feel unresponsive and broken.

## What performance problem occurs when fuzzy searching across a large dataset with multiple fields on each keystroke?

Searching through thousands of items across multiple fields can cause the CPU to max out, creating noticeable lag where user input is delayed from appearing on screen, sometimes taking a second or more instead of the required 16.6 milliseconds per frame.

### useTransition

useTransition returns an array with two values: isPending (a boolean indicating whether a transition is in progress) and startTransition (a function to mark state updates as low priority). The isPending boolean can be used to show loading indicators, while startTransition wraps state updates that should be processed at lower priority.

startTransition tells React to treat the wrapped state update as low priority work.

This means React will update high-priority items (like direct user interactions) first,

and then perform the transition update when the high-priority queue is clear. It's similar to requestIdleCallback, telling React to perform the update when it has time available.

### When using useDeferredValue, how can you determine if a transition is still pending without the isPending value from useTransition?

You can determine if a transition is pending by comparing the original value with the deferred value. If they are not equal, it means the deferred value has not yet caught up to the most recent value, implying a transition is still in progress. For example: const isPending = inputQuery !== deferredInputQuery.

- useTransition is typically used when you control the state update directly and want to mark it as LOW priority.

- useDeferredValue is used when you're on the receiving end of a value (like in a child component receiving props) and want to DEFER UPDATES to that value. Most commonly, you'll use useTransition when you control the state updates.

## useDeferredValue

### When should you use useDeferredValue instead of useTransition in React?

useDeferredValue should be used when you don't have full control over the value that's changing.

Such as when receiving a value as a prop from a library, parent component, or in a microfrontend architecture.

useTransition is preferred when you control the code and can DIRECTLY MANAGE the state updates.

## How does useDeferredValue handle incoming prop changes that would trigger expensive operations?

useDeferredValue returns the last known good value that was computed, even if a new value is received.

It keeps the old value until React Fiber has a chance to process low priority work, preventing immediate re-computation of expensive queries or operations.

The new value is then applied when React processes low priority tasks.

### What is the relationship between useDeferredValue and React Fiber's work scheduling?

useDeferredValue works with React Fiber's priority system by treating updates from the deferred value as LOW priority work.

When a new value comes in, React Fiber saves it as a unit of work (fiber of work) to be processed LATER when handling low priority tasks, rather than immediately triggering CASCADING UPDATES.

### What is the primary difference in control between useTransition and useDeferredValue?

useTransition gives you control over when to set both urgent and non-urgent state updates,
allowing you to explicitly manage which updates happen immediately and which are deferred.

useDeferredValue is for situations where you're on the receiving end of a value (like a prop) and don't control WHEN or HOW it changes.

### How does useDeferredValue prevent cascading updates from props?

When a prop that would normally trigger cascading updates is passed to useDeferredValue, it treats all downstream effects as a low priority transition task. 

This prevents the immediate cascade of re-renders and computations, DERERING THEM UNTIL React has TIME to process low priority work.

### How does React Suspense handle promises differently from traditional useEffect patterns?

With Suspense, Fiber works through the component tree and when it encounters an unresolved promise, it pauses that part of the tree and shows the Suspense fallback component instead.

Once the promise resolves, React can complete that part of the tree. This is different from useEffect, which runs after the render and commit phases are complete — meaning React has no awareness of async operations during rendering itself.

### What types of network and server issues can affect application performance beyond slow code execution?

Network and server issues include: servers being slow due to high load or being backed up, intermittent internet connections (like on public transit), and poor connectivity in general.

These communication-related issues are separate from code execution performance and require different optimization strategies like optimistic updates and offline capabilities.

### What are the benefits of using optimistic updates even when server operations have inherent latency?

Even when server responses take time (like AI chat responses that naturally take a second or two),
optimistic updates improve user experience by immediately showing the user's action in the UI.

For example, in a chat application, the user's message can appear in the chat window immediately upon hitting enter, while the actual server processing and response happen in the background.

This provides immediate feedback and makes the application feel more responsive.

## Optimistic Update

The useOptimistic hook function receives the current state (the existing list of posts) and the new optimistic item (the new post being added). 
It works similarly to useReducer, where you provide a function that takes the current state and the new item to determine what to display optimistically. 

Optimistic updates must be wrapped in a transition because they are considered low-priority updates compared to other renders. React will show an error if an optimistic update occurs outside of a transition or action, prompting you to wrap the update in startTransition to properly manage the update priority and pending states.

When the server request succeeds, the temporary optimistic item is swapped out with the real item returned from the server. 

When the request fails, the optimistic item is removed from the display and the error may bubble up to an error boundary, while React reverts to the original state it kept a copy of.

