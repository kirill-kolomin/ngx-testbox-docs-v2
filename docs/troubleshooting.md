---
sidebar_position: 4
---

# Troubleshooting

This page covers common issues you may encounter when using ngx-testbox.

## setInterval prevents stabilization

### Symptom
- Sync (`fakeAsync`): `MaximumAttemptsToStabilizeFixtureReachedError`
- Async (`runTasksUntilStableAsync`): `LongRunningComponentError`
- Or: "1 periodic timer(s) still in the queue"

### Diagnosis
Enable `debug: true` to see a console warning with the stack trace of where `setInterval` is called:

```typescript
runTasksUntilStable(fixture, {debug: true});
// or
await runTasksUntilStableAsync(fixture, {debug: true});
```

### Solutions
1. Mock the `setInterval` usage in your tests.
2. Run it outside the Angular zone:
   ```typescript
   this.ngZone.runOutsideAngular(() => setInterval(...));
   ```

:::warning
In Angular 17 and earlier, `runOutsideAngular` + `fakeAsync` may still leave periodic timers in the queue.
Use `discardPeriodicTasks()` at the end of the test as a workaround.
Starting from Angular 18 this is resolved.
:::

---

## Unused HTTP instruction error

### Symptom
`HttpInstructionWasNotExecutedDuringFixtureStabilizationError`

### Cause
You provided an instruction but your component never made the matching HTTP request.

### Solutions
- Remove the unused instruction.
- Verify your component actually triggers the request (check `ngOnInit`, event handlers, etc.).
- If the request is conditional, only add the instruction in the relevant test case.
- Double check similar URLs of your instructions. The most specific URL checkers must be defined closer to the beginning of http instructions' array.

---

## Unhandled HTTP request error

### Symptom
`NoMatchingHttpInstructionForRequestFoundError`

### Cause
Your component made an HTTP request that did not match any of the provided instructions.

### Solutions
- Add the missing instruction to `httpCallInstructions`.
- Double-check the URL string (exact match vs `.includes()` behavior) and the HTTP method.
- Use `debug: true` to inspect the pending request URL and method.

---

## Promise in fakeAsync response getter

### Symptom
`CannotUsePromiseResponseWithinFakeAsync`

### Cause
You passed a response getter that returns a `Promise` to `runTasksUntilStable` (which runs inside `fakeAsync`).

### Solutions
- Switch to the [async/await approach](Approaches/async-approach.md) (`runTasksUntilStableAsync`).
- Make the response getter synchronous.

---

## Timeout in async/await tests

### Symptom
`LongRunningComponentError`

### Cause
The component did not stabilize within `componentLongRunTimeout` (default: 10 seconds).

### Solutions
- Check for leaked `setInterval` or `setTimeout` calls (see above).
- Increase `componentLongRunTimeout` if the component legitimately takes longer:
  ```typescript
  await runTasksUntilStableAsync(fixture, {
    componentLongRunTimeout: 30_000,
  });
  ```

### Zoneless delayed work is not keeping the fixture unstable

If your app is zoneless and a timeout or interval should count toward fixture stability, use Angular's [`PendingTasks`](https://angular.dev/api/core/PendingTasks).

This is especially important when delayed work happens before the first HTTP request is created. Without `PendingTasks`, `runTasksUntilStableAsync` can finish early because Angular may consider the fixture stable even though your component still has delayed work to do.

If the timer comes from a third-party library and you cannot reasonably instrument it with `PendingTasks`, mock that library or the code path that depends on it in the test.

If you see `HttpInstructionWasNotExecutedDuringFixtureStabilizationError` in this situation, it usually means the component never reached the delayed code path during stabilization.

---

## HTTP requests don't affect zone stability

In some Angular versions, the fixture may report as stable even while HTTP requests are pending.
Ngx-testbox handles this internally by checking the HTTP testing queue directly, so this is not a concern for users.
