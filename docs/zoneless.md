---
sidebar_position: 4
---

# Zoneless

If your Angular app is zoneless, use the **async/await approach** for your tests.

Zoneless apps should prefer `runTasksUntilStableAsync` and `predefinedHttpCallInstructionsAsync`.
The `fakeAsync` / `tick()` approach is designed for zone-based tests and is not the right fit for zoneless apps.

## What to use

- Use `async` test functions.
- Use `runTasksUntilStableAsync`.
- Use `predefinedHttpCallInstructionsAsync` for HTTP mocking.
- Pass `advanceTimers` if your test environment uses fake timers.

## Why this matters

Zoneless Angular does not rely on Zone.js to decide when a component is stable.
That makes the async/await stabilization flow the most reliable option for tests that should match zoneless runtime behavior.

## Example

```typescript
it('should stabilize in a zoneless app', async () => {
  await runTasksUntilStableAsync(fixture, {
    httpCallInstructions: [
      predefinedHttpCallInstructionsAsync.get.success('/api/items', () => items),
    ],
  });
});
```

## Troubleshooting: delayed work should affect stability

Sometimes an app schedules work with `setTimeout`, `setInterval`, or another delayed callback before the first HTTP request exists.

If that delayed work should keep the app unstable until it finishes, wrap it with Angular's [`PendingTasks`](https://angular.dev/api/core/PendingTasks).

This works best for application code you control. If the timer lives inside a third-party library and you cannot realistically wrap it with `PendingTasks`, the usual solution is to mock that library or the code path that triggers it in your test.

Example:

```typescript
import {inject, PendingTasks} from '@angular/core';

const pendingTasks = inject(PendingTasks);
const taskCleanup = pendingTasks.add();

setTimeout(() => {
  try {
    // do delayed work that should keep the test unstable
  } finally {
    taskCleanup();
  }
}, 300);
```

Without `PendingTasks`, Angular may consider the fixture stable before that delayed work produces a request or state change.
