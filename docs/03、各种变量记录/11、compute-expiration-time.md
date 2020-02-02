packages/react-reconciler/ReactFiberWorkLoop.js

``` javascript
export const LOW_PRIORITY_EXPIRATION = 5000;
export const LOW_PRIORITY_BATCH_SIZE = 250;

// Compute an expiration time based on the Suspense timeout.
let expirationTime = computeSuspenseExpiration(
  currentTime,
  suspenseConfig.timeoutMs | 0 || LOW_PRIORITY_EXPIRATION,
);

if (suspenseConfig !== null) {
  // Compute an expiration time based on the Suspense timeout.
  expirationTime = computeSuspenseExpiration(
    currentTime,
    suspenseConfig.timeoutMs | 0 || LOW_PRIORITY_EXPIRATION,
  );
} else {
  // Compute an expiration time based on the Scheduler priority.
  switch (priorityLevel) {
    case ImmediatePriority:
      expirationTime = Sync;
      break;
    case UserBlockingPriority:
      // TODO: Rename this to computeUserBlockingExpiration
      expirationTime = computeInteractiveExpiration(currentTime);
      break;
    case NormalPriority:
    case LowPriority: // TODO: Handle LowPriority
      // TODO: Rename this to... something better.
      expirationTime = computeAsyncExpiration(currentTime);
      break;
    case IdlePriority:
      expirationTime = Idle;
      break;
    default:
      invariant(false, 'Expected a valid priority level');
  }
}

export function computeAsyncExpiration(
  currentTime: ExpirationTime,
): ExpirationTime {
  return computeExpirationBucket(
    currentTime,
    LOW_PRIORITY_EXPIRATION,
    LOW_PRIORITY_BATCH_SIZE,
  );
}

export function computeSuspenseExpiration(
  currentTime: ExpirationTime,
  timeoutMs: number,
): ExpirationTime {
  // TODO: Should we warn if timeoutMs is lower than the normal pri expiration time?
  return computeExpirationBucket(
    currentTime,
    timeoutMs,
    LOW_PRIORITY_BATCH_SIZE,
  );
}

export const HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150;
export const HIGH_PRIORITY_BATCH_SIZE = 100;

// We intentionally set a higher expiration time for interactive updates in
// dev than in production.
//
// If the main thread is being blocked so long that you hit the expiration,
// it's a problem that could be solved with better scheduling.
//
// People will be more likely to notice this and fix it with the long
// expiration time in development.
//
// In production we opt for better UX at the risk of masking scheduling
// problems, by expiring fast.
export function computeInteractiveExpiration(currentTime: ExpirationTime) {
  return computeExpirationBucket(
    currentTime,
    HIGH_PRIORITY_EXPIRATION,
    HIGH_PRIORITY_BATCH_SIZE,
  );
}

function ceiling(num: number, precision: number): number {
  return (((num / precision) | 0) + 1) * precision;
}

function computeExpirationBucket(
  currentTime,
  expirationInMs,
  bucketSizeMs,
): ExpirationTime {
  // 1073741821 - ((((1073741821 - currentTime + expirationInMs / 10) / 25) | 0) + 1) * 25
  return (
    MAGIC_NUMBER_OFFSET -
    ceiling(
      MAGIC_NUMBER_OFFSET - currentTime + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  );
}
```

(((num / 25) | 0) + 1) * 25