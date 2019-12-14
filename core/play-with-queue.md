---
description: Queue represents a concurrent queue
---

# Play with Queue

## Queue

```typescript
export interface ConcurrentQueue<A> {
  readonly take: T.Effect<T.NoEnv, never, A>;
  offer(a: A): T.Effect<T.NoEnv, never, void>;
}

/**
 * Create an unbounded concurrent queue
 */
export function unboundedQueue<A>(): T.Effect<
  T.NoEnv,
  never,
  ConcurrentQueue<A>
>

/**
 * Create a bounded queue with the given capacity that drops older offers
 * @param capacity
 */
export function slidingQueue<A>(
  capacity: number
): T.Effect<T.NoEnv, never, ConcurrentQueue<A>>

/**
 * Create a dropping queue with the given capacity that drops offers on full
 * @param capacity
 */
export function droppingQueue<A>(
  capacity: number
): T.Effect<T.NoEnv, never, ConcurrentQueue<A>>

/**
 * Create a bounded queue that blocks offers on capacity
 * @param capacity
 */
export function boundedQueue<A>(
  capacity: number
): T.Effect<T.NoEnv, never, ConcurrentQueue<A>>
```

## Usage

```typescript
import { effect as T, queue as Q, ref as R } from "@matechs/effect";
import { Do } from "fp-ts-contrib/lib/Do";
import { pipe } from "fp-ts/lib/pipeable";

const program: T.Effect<unknown, never, void> = Do(T.effect)
  .bind("queue", Q.unboundedQueue<number>()) // create a queue
  .bind("ref", R.makeRef(0)) // create a reference
  .bindL("fiberOffer", ({ queue, ref }) => 
    T.fork( // fork a new fiber that offer n every second
      T.forever(
        T.delay(
          pipe(
            ref.modify(n => [n, n + 1]),
            T.chain(queue.offer)
          ),
          1000
        )
      )
    )
  )
  .bindL("fiberTake", ({ queue }) =>
    T.fork( // fork a new fiber that process each n and logs
      T.forever(
        pipe(
          queue.take,
          T.chain(n =>
            T.sync(() => {
              console.log(`got: ${n}`);
            })
          )
        )
      )
    )
  )
  .doL(({ fiberOffer, fiberTake }) =>
    // interrupt producer fiber after 5 seconds
    T.effect.chain(T.delay(fiberOffer.interrupt, 5000), _ =>
      // interrupt consumer fiber after 1 second
      T.delay(fiberTake.interrupt, 1000)
    )
  )
  .return(_ => {}); // return void

T.run(program);

// prints:
// got: 0
// got: 1
// got: 2
// got: 3
```

