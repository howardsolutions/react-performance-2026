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
1) Necessary 
2) UN-necessary

Necessary RE-RENDERS come in 2 flavors: 

1) URGENT 
2) NON-URGENT

## SOME THEMES

- for web performance... and life.

- Not doing stuff is *way faster* than doing stuff. *(Component hierarchy & state management)*
- Seeing if you can skip doing stuff is *sometimes* less work than doing stuff. *(Memoization + Compiler)*
- You can *put off* doing stuff. *(Suspense + Transitions)*
- Load as much as you need and as little as you can get away with. *(Lazy loading + bundle optimization)*

## React Fiber

React Fiber is a cooperatively scheduled rendering engine that changes how React figures out what changes to make to the DOM. It allows React to be smarter about rendering by being able to pause, interrupt, or restart work based on priority, rather than blocking the main thread until completion.

React Fiber enables React to distinguish between urgent and non-urgent updates by checking in periodically to determine if the current work is still valuable or if there are more important tasks to handle. It can pause or stop less important work to handle urgent updates, then either resume or restart the previous work.

### Early versions of React
Early versions of React would start at the top of the component tree and go through the entire tree in a blocking manner. Once the process started, it would block the main thread and run to completion, regardless of what happened, which could cause performance issues in larger applications

### React Fiber help improve Performance even when not explicitly used

React Fiber comes up for air regularly (approximately every 16.6 milliseconds to match the browser's 60 frames per second) to allow other operations in the application to execute, such as CSS animations, promise resolutions, and other JavaScript operations. This prevents blocking the main thread and improves overall application responsiveness.

### React Fiber sometimes do more work but still feel faster to users?
React Fiber may technically do more work by stopping current tasks to respond to higher-priority updates and then having to redo some work. However, this approach feels faster to users because it prioritizes responding to important interactions immediately rather than completing less important work first, even though the total computational work may be greater.