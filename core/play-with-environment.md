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
import { effect as T, exit as ex } from "@matechs/effect"

it("provideR", async () => {
  const http_symbol: unique symbol = Symbol();

  interface HttpEnv {
    [http_symbol]: number;
  }

  assert.deepEqual(
    await T.runToPromiseExit(
      T.provideR(() => ({ [http_symbol]: 10 }))(
        T.access(({ [http_symbol]: n }: HttpEnv) => n)
      )
    ),
    ex.done(10)
  );
});

it("provide", async () => {
  const http_symbol: unique symbol = Symbol();

  interface HttpEnv {
    [http_symbol]: number;
  }

  assert.deepEqual(
    await T.runToPromiseExit(
      pipe(
        T.access(({ [http_symbol]: n }: HttpEnv) => n),
        T.provide({ [http_symbol]: 10 }),
        T.provide(env2),
        T.provide(env3)
      )
    ),
    ex.done(10)
  );
});

it("mergeEnv", async () => {
  const http_symbol: unique symbol = Symbol();

  interface HttpEnv {
    [http_symbol]: number;
  }

  const env = pipe(
      T.noEnv,
      T.mergeEnv({ [http_symbol]: 10 }),
      T.mergeEnv(env2),
      T.mergeEnv(env3)
  )

  assert.deepEqual(
    await T.runToPromiseExit(
      pipe(
        T.access(({ [http_symbol]: n }: HttpEnv) => n),
        T.provideAll(env)
      )
    ),
    ex.done(10)
  );
});

```

You should play with each of the above combinators and try them in the various scenarios that you can think of, in each package you can find the same pattern applied over and over at: [https://github.com/mikearnaldi/matechs-effect/tree/master/packages](https://github.com/mikearnaldi/matechs-effect/tree/master/packages) \(look at test packages first to understand usage\)

