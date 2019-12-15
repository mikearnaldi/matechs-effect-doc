---
description: There are many ways you can access and provide environment
---

# Play with Environment

## Environment

Following the list of environment related combinators:

```typescript
// access all environment
export function accessEnvironment<R>(): Effect<R, NoErr, R>

// access environment and use it effectfully
export function accessM<R, R2, E, A>(
  f: FunctionN<[R], Effect<R2, E, A>>
): Effect<R & R2, E, A>

// access environment and use it purely
export function access<R, A, E = NoErr>(
  f: FunctionN<[R], A>
): Effect<R, E, A>

// merge environments (use deep-merge, use only at the top level)
export function mergeEnv<A>(a: A): <B>(b: B) => A & B 

// provide part of the environment (use deep-merge, use only at the top level)
export const provide = <R>(r: R) => <R2, E, A>(
  ma: Effect<R2 & R, E, A>
): Effect<R2, E, A> => accessM((r2: R2) => provideAll(mergeEnv(r2)(r))(ma));

// provide part of the environment with a transformation function
// equivalent to provide but efficient (no use of deep merging)
export const provideR = <R2, R>(f: (r2: R2) => R) => <E, A>
  (ma: Effect<R, E, A>): Effect<R2, E, A>

// provide all the environment
export const provideAll = <R>(r: R) => <E, A>(
  ma: Effect<R, E, A>
): Effect<NoEnv, E, A>

// provide environment effectfuly (only top-level)
export const provideM = <R2, R, E2>(
  f: Effect<R2, E2, R>
) => <E, A>(ma: Effect<R, E, A>): Effect<R2, E | E2, A> 

// provide part of environment effectfuly (only top-level)
export const provideSomeM = <R2, R, E2>(
  f: Effect<R2, E2, R>
) => <E, A, R3>(ma: Effect<R & R3, E, A>): Effect<R2 & R3, E | E2, A>
```

## Usage

```typescript
import { effect as T, exit as E } from "@matechs/effect";
import { pipe } from "fp-ts/lib/pipeable";

// utility to handle exit
const foldExit = <A>(onDone: (a: A) => void) =>
  E.fold(
    onDone,
    e => console.error("error:", e),
    e => console.error("abort:", e),
    () => console.error("interrupt")
  );

// unique symbol to hold an environment for app configuration
const appConfigEnv: unique symbol = Symbol();

// describe the AppConfig environment entry
interface AppConfig {
  [appConfigEnv]: {
    appName: string;
  };
}

// implement the live version of the config
const appConfigLive: AppConfig = {
  [appConfigEnv]: {
    appName: "My First App"
  }
};

// utility to access the config
function appName(): T.Effect<AppConfig, never, string> {
  return T.access(({ [appConfigEnv]: { appName } }: AppConfig) => appName);
}

// unique symbol to hold an environment for console effect
const consoleEnv: unique symbol = Symbol();

// describe the console environmental effect
interface Console {
  [consoleEnv]: {
    log: (s: string) => T.Effect<T.NoEnv, T.NoErr, void>;
  };
}

// live implementation of the console effect
const consoleLive: Console = {
  [consoleEnv]: {
    log: s =>
      T.sync(() => {
        console.log(s);
      })
  }
};

// utility to access the console log through the environment
function log(s: string): T.Effect<Console, never, void> {
  return T.accessM(({ [consoleEnv]: { log } }: Console) => log(s));
}

// log the app name, combine the environment requirements
const program: T.Effect<Console & AppConfig, never, void> = pipe(
  appName(),
  T.chain(log)
);

// construct the live environment
const live = pipe(
  T.noEnv, 
  T.mergeEnv(appConfigLive), 
  T.mergeEnv(consoleLive)
);

T.run(
  T.provideAll(live)(program), // print: My First App
  foldExit(() => {
    // no output
  })
);
```

## Notes

Type signatures of chain and all the main combinators have been refined to merge environment requirements so you can keep environment requirements on each function as small as possible as the required environment of a function is the environment you will have to provide for test purposes.

When you use environmental effects for everything you easily get to the point of having very long chains of combined environments, it is useful not to restrict the return type of your functions from the beginning and progressively type as you go. Type aliases can significantly reduce this "problem" and make consumer usage more friendly.

An example of the above can be seen in the [express](https://github.com/mikearnaldi/matechs-effect/blob/master/packages/express/src/index.ts#L182) package where environment aliases have been exposed in a combined way for the consumer while internally kept more granular \(granularity can help further with progressive providing of environment see for example the [route](https://github.com/mikearnaldi/matechs-effect/blob/master/packages/express/src/index.ts#L131) combinator in the express package\)

