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

We can create a module that wraps the add / mul operations in the environment as follows:

```typescript
import { T, pipe, Ex } from "@matechs/aio";
import * as assert from "assert";

// define a unique resource identifier
const CalculatorURI = "@matechs/examples/CalculatorURI";

// define the module description as an interface
interface Calculator {
  // scope it using the previously defined URI
  [CalculatorURI]: {
    add(x: number, y: number): T.Sync<number>;
    mul(x: number, y: number): T.Sync<number>;
  };
}

// access the module from environment and expose the add function
const add = (x: number, y: number): T.SyncR<Calculator, number> =>
  T.accessM(({ [CalculatorURI]: { add } }: Calculator) => add(x, y));

// access the module from environment and expose the mul function
const mul = (x: number, y: number): T.SyncR<Calculator, number> =>
  T.accessM(({ [CalculatorURI]: { mul } }: Calculator) => mul(x, y));

// our program is now independent from a concrete implementation
const addAndMul: T.SyncR<Calculator, number> = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

// define a provider for the specific Calculator module
const provideCalculator = T.provide<Calculator>({
  [CalculatorURI]: {
    add: (x, y) => T.sync(() => x + y),
    mul: (x, y) => T.sync(() => x * y)
  }
});

// run the program providing the concrete implementation
const result: Ex.Exit<never, number> = pipe(
  addAndMul,
  provideCalculator,
  T.runSync
);

assert.deepStrictEqual(result, Ex.done(6));

// define a second provider for the specific Calculator module
const provideCalculatorWithLog = (messages: Array<string>) =>
  T.provide<Calculator>({
    [CalculatorURI]: {
      add: (x, y) =>
        T.applySecond(
          T.sync(() => {
            messages.push(`called add with ${x}, ${y}`);
          }),
          T.sync(() => x + y)
        ),
      mul: (x, y) =>
        T.applySecond(
          T.sync(() => {
            messages.push(`called mul with ${x}, ${y}`);
          }),
          T.sync(() => x * y)
        )
    }
  });

// run the program providing the concrete implementation
const messages: Array<string> = [];
const resultWithLog: Ex.Exit<never, number> = pipe(
  addAndMul,
  provideCalculatorWithLog(messages),
  T.runSync
);

assert.deepStrictEqual(resultWithLog, Ex.done(6));
assert.deepStrictEqual(messages, [
  "called add with 1, 2",
  "called mul with 3, 2"
]);
```

## Multiple Environments

We can arbitrarily compose computations that require different environments as follows:

```typescript
import { T, pipe, Ex } from "@matechs/aio";
import * as assert from "assert";

// define a unique resource identifier
const AddURI = "@matechs/examples/AddURI";

// define the module description as an interface
interface Add {
  // scope it using the previously defined URI
  [AddURI]: {
    add(x: number, y: number): T.Sync<number>;
  };
}

// define a unique resource identifier
const MulURI = "@matechs/examples/MulURI";

// define the module description as an interface
interface Mul {
  // scope it using the previously defined URI
  [MulURI]: {
    mul(x: number, y: number): T.Sync<number>;
  };
}

// access the module from environment and expose the add function
const add = (x: number, y: number): T.SyncR<Add, number> =>
  T.accessM(({ [AddURI]: { add } }: Add) => add(x, y));

// access the module from environment and expose the mul function
const mul = (x: number, y: number): T.SyncR<Mul, number> =>
  T.accessM(({ [MulURI]: { mul } }: Mul) => mul(x, y));

// our program is now independent from a concrete implementation
const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

// define a provider for the specific Add module
const provideAdd = T.provide<Add>({
  [AddURI]: {
    add: (x, y) => T.sync(() => x + y)
  }
});

// define a provider for the specific Mul module
const provideMul = T.provide<Mul>({
  [MulURI]: {
    mul: (x, y) => T.sync(() => x * y)
  }
});

// run the program providing the concrete implementation
const result: Ex.Exit<never, number> = pipe(
  addAndMul, // T.SyncR<Mul & Add, number>
  provideAdd, // T.SyncR<Mul, number>
  provideMul, // T.Sync<number>
  T.runSync
);

assert.deepStrictEqual(result, Ex.done(6));
```

Note how we purposly omitted the type of addAndMul to show that all requirements are correctly inferred from usage, in fact if we forget to provide one dependency we will get a compilation error indicating that the dependency is missing as follows

```typescript
// run the program providing the concrete implementation
const result: Ex.Exit<never, number> = pipe(
  addAndMul,
  provideAdd,
  T.runSync // '[MulURI]' is missing in type '{}' but required in type 'Mul'
);
```

## Combining Providers

We can combine arbitrary providers as follows:

```typescript
import { T, pipe, Ex, combineProviders } from "@matechs/aio";
import * as assert from "assert";

// define a unique resource identifier
const AddURI = "@matechs/examples/AddURI";

// define the module description as an interface
interface Add {
  // scope it using the previously defined URI
  [AddURI]: {
    add(x: number, y: number): T.Sync<number>;
  };
}

// define a unique resource identifier
const MulURI = "@matechs/examples/MulURI";

// define the module description as an interface
interface Mul {
  // scope it using the previously defined URI
  [MulURI]: {
    mul(x: number, y: number): T.Sync<number>;
  };
}

// access the module from environment and expose the add function
const add = (x: number, y: number): T.SyncR<Add, number> =>
  T.accessM(({ [AddURI]: { add } }: Add) => add(x, y));

// access the module from environment and expose the mul function
const mul = (x: number, y: number): T.SyncR<Mul, number> =>
  T.accessM(({ [MulURI]: { mul } }: Mul) => mul(x, y));

// our program is now independent from a concrete implementation
const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

// define a provider for the specific Add module
const provideAdd = T.provide<Add>({
  [AddURI]: {
    add: (x, y) => T.sync(() => x + y)
  }
});

// define a provider for the specific Mul module
const provideMul = T.provide<Mul>({
  [MulURI]: {
    mul: (x, y) => T.sync(() => x * y)
  }
});

// combine the 2 providers into a single one
// inferred as T.Provider<unknown, Add & Mul, never, never>
const provideLive = combineProviders().with(provideAdd).with(provideMul).done();

// run the program providing the concrete implementation
const result: Ex.Exit<never, number> = pipe(addAndMul, provideLive, T.runSync);

assert.deepStrictEqual(result, Ex.done(6));

```

## Higher Order Dependencies

Sometimes you may want to have your providers depending on other modules, you can do that as follows:

```typescript
import { T, pipe, Ex } from "@matechs/aio";
import * as assert from "assert";

// define a unique resource identifier
const AddURI = "@matechs/examples/AddURI";

// define the module description as an interface
interface Add {
  // scope it using the previously defined URI
  [AddURI]: {
    add(x: number, y: number): T.Sync<number>;
  };
}

// define a unique resource identifier
const MulURI = "@matechs/examples/MulURI";

// define the module description as an interface
interface Mul {
  // scope it using the previously defined URI
  [MulURI]: {
    mul(x: number, y: number): T.Sync<number>;
  };
}

// access the module from environment and expose the add function
const add = (x: number, y: number): T.SyncR<Add, number> =>
  T.accessM(({ [AddURI]: { add } }: Add) => add(x, y));

// access the module from environment and expose the mul function
const mul = (x: number, y: number): T.SyncR<Mul, number> =>
  T.accessM(({ [MulURI]: { mul } }: Mul) => mul(x, y));

// our program is now independent from a concrete implementation
const addAndMul = pipe(
  add(1, 2),
  T.chain((n) => mul(n, 2))
);

// define a unique resource identifier
const LoggerURI = "@matechs/examples/LoggerURI";

// define a unique resource identifier
interface Logger {
  // scope it using the previously defined URI
  [LoggerURI]: {
    log(message: string): T.Sync<void>;
  };
}

// access logger from environment
const accessLogger = T.access(({ [LoggerURI]: logger }: Logger) => logger);

// define a provider for the specific Add module depending on Logger
const provideAdd = pipe(
  accessLogger,
  T.map(
    (logger): Add => ({
      [AddURI]: {
        add: (x, y) =>
          pipe(
            T.sync(() => x + y),
            T.chainTap((n) => logger.log(`result: ${n}`))
          )
      }
    })
  ),
  T.provideM // provide monadically
);

// define a provider for the specific Mul module
const provideMul = T.provide<Mul>({
  [MulURI]: {
    mul: (x, y) => T.sync(() => x * y)
  }
});

// define a provider for the specific Log module
const provideLog = (messages: Array<string>) =>
  T.provide<Logger>({
    [LoggerURI]: {
      log: (message) =>
        T.sync(() => {
          messages.push(message);
        })
    }
  });

// run the program providing the concrete implementation
const messages: Array<string> = [];
const result: Ex.Exit<never, number> = pipe(
  addAndMul, // T.SyncR<Mul & Add, number>
  provideAdd, // T.SyncR<Logger & Add, number>
  provideMul, // T.SyncR<Logger, number>
  provideLog(messages), // T.Sync<number>
  T.runSync
);

assert.deepStrictEqual(result, Ex.done(6));
assert.deepStrictEqual(messages, ["result: 3"]);
```

