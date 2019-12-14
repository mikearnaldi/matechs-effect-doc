---
description: >-
  Semaphore represents a generalized mutex, you can think of it as a physical
  semaphore that manages traffic
---

# Play with Semaphore

## Semaphore

```typescript
export interface Semaphore {
  /**
   * Acquire a permit, blocking if not all are vailable
   */
  readonly acquire: T.Effect<T.NoEnv, never, void>;
  /**
   * Release a permit
   */
  readonly release: T.Effect<T.NoEnv, never, void>;
  /**
   * Get the number of available permits
   */
  readonly available: T.Effect<T.NoEnv, never, number>;

  /**
   * Acquire multiple permits blocking if not all are available
   * @param n
   */
  acquireN(n: number): T.Effect<T.NoEnv, never, void>;
  /**
   * Release mutliple permits
   * @param n
   */
  releaseN(n: number): T.Effect<T.NoEnv, never, void>;
  /**
   * Bracket the given io with acquireN/releaseN calls
   * @param n
   * @param io
   */
  withPermitsN<R, E, A>(n: number, io: T.Effect<R, E, A>): T.Effect<R, E, A>;
  /**
   * withPermitN(1, _)
   * @param n
   */
  withPermit<R, E, A>(n: T.Effect<R, E, A>): T.Effect<R, E, A>;
}

/**
 * Allocate a semaphore.
 *
 * @param n the number of permits
 * This must be non-negative
 */
export function makeSemaphore(n: number): T.Effect<T.NoEnv, never, Semaphore> {
  return T.applySecond(
    sanityCheck(n),
    effect.map(makeRef(right(n) as State), makeSemaphoreImpl)
  );
}
```

## Usage

```typescript
import { effect as T, semaphore as S, exit as E } from "@matechs/effect";
import { Do } from "fp-ts-contrib/lib/Do";
import * as A from "fp-ts/lib/Array";

const program = Do(T.effect)
  .bind("sem", S.makeSemaphore(10)) // create a semaphore with 10 permits
  .bindL("res", ({ sem }) =>
    // this uses parallel instances and concurrency is limited at 10
    A.array.traverse(T.parEffect)(A.range(1, 10000), n =>
      sem.withPermit(T.pure(n + 1)) // use a permit to run op
    )
  )
  .return(s => s.res);

T.run(
  program,
  E.fold(
    arr => {
      console.log(arr.length); // prints 10000
    },
    console.error, // raised
    console.error, // abort
    () => {
      console.error("interrupted");
    }
  )
);
```

