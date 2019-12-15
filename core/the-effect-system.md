---
description: Description of the core component of the system
---

# The Effect System

The main type you will be working with is:

```typescript
type Effect<R, E, A>
```

Mentally you can think of this in simplified terms as:

```typescript
type Effect<R, E, A> = (environment: R) => Async<E, A>
```

Where `R` represents an environment needed for the computation `Async<E, A>` to run and when started the computation may succeed with a result `A` or fail with an error `E`.

## Simple Effect

Let's start with a simple hello world:

```typescript
import { effect as T } from "@matechs/effect"

const helloWorld = T.sync(() => {
    console.log("hello world!")
})
```

We can already highlight the type of `helloWorld` and discover that:

```typescript
import { effect as T } from "@matechs/effect"

const helloWorld: T.Effect<unknown, never, void> = T.sync(() => {
    console.log("hello world!")
})
```

So we can say that `helloWorld` is an effect that requires no environment, can never fail and upon completion returns `void`.

## Running

We can run the effect in multiple ways:

```typescript
// failable promise
const asPromise: Promise<void> = T.runToPromise(helloWorld)

// non failable promise, Exit will be disected later but encapsulate all exit
// scenarios (success, raised, aborted, interrupted)
const asPromiseExit: Promise<Exit<never, void>> = T.runToPromiseExit(helloWorld)

// the best way to run the effect is not to use a promise, 
// both for performance reasons and because promise is not easily
// cancellable. 
const cancelFunction: Lazy<void> = T.run(helloWorld, (ex: Exit<never, void>) => {
    // executed
})

// cancel the running computation
cancelFunction();
```

What's wrong with this? Nothing in principle, but from `helloWorld` we are directly calling `console.log`. What if we want to test this?

## Environmental Effect

We can create a module that wraps the console interaction and use the module from environment like:

```typescript
// create a unique symbol to identify the module in the environment
// you can use either a unique symbol or a string, 
const consoleUri: unique symbol = Symbol();

// create an interface that describe the module
// you want to put only core implementation functions here
interface ConsoleEnv {
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

