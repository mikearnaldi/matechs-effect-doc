---
description: Ref represents an immutable reference to a mutable value
---

# Play with Ref

## Ref

```typescript
// describe a reference to A
export interface Ref<A> {
  // read the current value
  readonly get: T.Effect<T.NoEnv, never, A>;
  
  // set the value
  set(a: A): T.Effect<T.NoEnv, never, A>;
  
  // update the value
  update(f: FunctionN<[A], A>): T.Effect<T.NoEnv, never, A>;
  
  // update the value and return B
  modify<B>(f: FunctionN<[A], readonly [B, A]>): T.Effect<T.NoEnv, never, B>;
}

// create a new reference
export const makeRef = <A>(initial: A): T.Effect<T.NoEnv, never, Ref<A>>
```

## Usage

```typescript
import { effect as T, ref as R } from "@matechs/effect";
import { Do } from "fp-ts-contrib/lib/Do";

const program: T.Effect<unknown, never, number> = Do(T.effect)
  .bind("ref", R.makeRef(0)) // create a new ref<number>
  .doL(({ ref }) => ref.update(n => n + 1)) // increment
  .bindL("value", ({ ref }) => ref.get) // read
  .return(s => s.value); // return
```

