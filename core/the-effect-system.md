---
description: Description of the core component of the system
---

# The Effect System

The main type you will be working with is:

```typescript
type Effect<S, R, E, A>

type Async<A> = Effect<unknown, unknown, never, A>;
type AsyncE<E, A> = Effect<unknown, unknown, E, A>;
type AsyncR<R, A> = Effect<unknown, R, never, A>;
type AsyncRE<R, E, A> = Effect<unknown, R, E, A>;

type Sync<A> = Effect<never, unknown, never, A>;
type SyncE<E, A> = Effect<never, unknown, E, A>;
type SyncR<R, A> = Effect<never, R, never, A>;
type SyncRE<R, E, A> = Effect<never, R, E, A>;
```

The Effect signature reads as follows:

```text
Effect<S, R, E, A> is an effectful computation that 

can be Syncronious or Asyncronious (S) and
requires an environment of type R to run and
can produce either an error of type E or
a success reponse of type A
```

## Package Exports

```typescript
import * as I from "io-ts";
import * as IT from "io-ts-types";
import * as NT from "newtype-ts";
import * as MN from "monocle-ts";
import * as MO from "./morphic";

export {
  I, // io-ts
  IT, // io-ts-types
  NT, // newtype-ts
  MN, // monocle-ts
  MO // morphic-ts
};

export {
  A, // fp-ts Array
  CRef, // Concurrent Reference
  E, // fp-ts augumented Either
  Ex, // Exit
  F, // fp-ts Function
  M, // Managed
  NEA, // fp-ts NonEmptyArray
  O, // fp-ts augumented Option
  Q, // Queue
  RT, // Retry
  Rec, // Recursion Schemes
  Ref, // Reference
  S, // Stream
  SE, // StreamEither
  Sem, // Semaphore
  Service, // FreeEnv Service Definition & Derivation
  T, // Effect
  U, // Type Utils
  combineProviders, // Combine Providers
  eq, // fp-ts Eq
  flow, // fp-ts flow
  flowF, // fluent flow - not limited to 10
  magma, // fp-ts Magma
  map, // fp-ts Map
  monoid, // fp-ts Monoid
  pipe, // fp-ts pipe
  pipeF, // fluent pipe - not limited to 10
  record, // fp-ts Record
  semigroup, // fp-ts Semigroup
  set, // fp-ts Set
  show, // fp-ts Show
  tree // fp-ts Tree
} from "@matechs/prelude";

```

## Simple Effect

Let's start with a simple syncronious computation:

```typescript
import { T, pipe, Ex } from "@matechs/aio";
import * as assert from "assert";

const add = (x: number, y: number): T.Sync<number> => T.sync(() => x + y);
const mul = (x: number, y: number): T.Sync<number> => T.sync(() => x * y);

const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

const result: Ex.Exit<never, number> = T.runSync(addAndMul);

assert.deepStrictEqual(result, Ex.done(6));
```

The same computation can run in different ways:

```typescript
import { T, pipe, Ex, F } from "@matechs/aio";
import * as assert from "assert";

const add = (x: number, y: number): T.Sync<number> => T.sync(() => x + y);
const mul = (x: number, y: number): T.Sync<number> => T.sync(() => x * y);

const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

// run as non failable promise returning Exit
T.runToPromiseExit(addAndMul).then((result) => {
  assert.deepStrictEqual(result, Ex.done(6));
});

// run as failable promise returning result
T.runToPromise(addAndMul)
  .then((result) => {
    assert.deepStrictEqual(result, 6);
  })
  .catch((error) => {
    console.error(error);
  });

// invoking canceller cancel the computation (not in this case because all sync)
const canceller: F.Lazy<void> = T.run(addAndMul, (result) => {
    assert.deepStrictEqual(result, Ex.done(6));
})

// run as throwable
const result_n: number = T.runUnsafeSync(addAndMul)

assert.deepStrictEqual(result_n, 6);
```

Let's add some asynchronousity to the computation by adding a simple delay via liftDelay:

```typescript
import { T, pipe, Ex, F } from "@matechs/aio";
import * as assert from "assert";

const add = (x: number, y: number): T.Sync<number> => T.sync(() => x + y);
const mul = (x: number, y: number): T.Sync<number> => T.sync(() => x * y);

const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2)),
  T.liftDelay(100) // delay execution for 100ms
);

// run as non failable promise returning Exit
T.runToPromiseExit(addAndMul).then((result) => {
  assert.deepStrictEqual(result, Ex.done(6));
});

// run as failable promise returning result
T.runToPromise(addAndMul)
  .then((result) => {
    assert.deepStrictEqual(result, 6);
  })
  .catch((error) => {
    console.error(error);
  });

// invoking canceller cancel the computation (not in this case because all sync)
const canceller: F.Lazy<void> = T.run(addAndMul, (result) => {
    assert.deepStrictEqual(result, Ex.done(6));
})
```

If we now try to use runSync we will get a compile error:

```typescript
T.runSync(addAndMul) 
// Argument of type 'AsyncRE<unknown, never, number>' is not assignable
// to parameter of type 'SyncRE<{}, never, number>'. 
// Type 'unknown' is not assignable to type 'never'
```

This is the first time we see a very important principle in statically typed functional programming, encoding logic at the type level to make errors impossible. 

## Environmental Effect

We can create a module that wraps the console interaction and use the module from environment like:

```typescript
// create a unique symbol to identify the module in the environment
// you can use either a unique symbol or a string, 
const consoleUri: unique symbol = Symbol();

// create an interface that describe the module
// you want to put only core implementation functions here
interface ConsoleEnv {
  [consoleUri]: {
    log: (s: string) => T.Effect<unknown, never, void>;
  };
}

// create utility functions that access the environment
// and use the effect
// you want to create a DSL using those functions
function log(s: string) {
  return T.accessM((env: ConsoleEnv) => env[consoleUri].log(s));
}
```

Let's use our newly created `log` function:

```typescript
// as you can see we are now requiring a ConsoleEnv to run our effect
const helloWorld: T.Effect<ConsoleEnv, never, void> = log(
  "hello world"
);
```

Let's write an implementation:

```typescript
const consoleLive: ConsoleEnv = {
  [consoleUri]: {
    log: (s: string) => T.sync(() => console.log(s))
  }
};
```

## Running Live

In order to run the program you will need to provide the module before passing it into the `run`/`runToPromise`/`runToPromiseExit`:

```typescript
T.run(T.provideAll(consoleLive)(helloWorld))
```

The `provideAll` function takes a complete environment and provides it to the program, when called this will print:

```typescript
$ yarn ts-node src/hello.ts
hello world!
```

## Running Test

What if we want to test now?

```typescript
// in your preferred test framework  
async function helloWorldTest() {
  // hold messages
  const messages: string[] = [];

  const consoleTest: ConsoleEnv = {
    [consoleUri]: {
      log: (s: string) =>
        T.sync(() => {
          messages.push(s);
        })
    }
  };

  // the real "log" & "helloWorld" will be called, console will be
  // using test environment
  await T.runToPromiseExit(T.provideAll(consoleTest)(helloWorld));

  assert.deepEqual(messages, ["hello world"])
}
```

