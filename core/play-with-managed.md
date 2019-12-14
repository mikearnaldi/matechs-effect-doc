---
description: Managed represent safe acquisition / usage and release of resources
---

# Play with Managed

## Managed

```typescript
/**
 * Lift a pure value into a resource
 * @param value
 */
export function pure<R = T.NoEnv, E = T.NoErr, A = unknown>(
  value: A
): Managed<R, E, A>

/**
 * Create a Resource by wrapping an IO producing a value that does not need to be disposed
 *
 * @param res
 * @param f
 */
export function encaseEffect<R, E, A>(
  rio: T.Effect<R, E, A>
): Managed<R, E, A>

/**
 * Create a resource from an acquisition and release function
 * @param acquire
 * @param release
 */
export function bracket<R, E, A, R2, E2>(
  acquire: T.Effect<R, E, A>,
  release: FunctionN<[A], T.Effect<R2, E2, unknown>>
): Managed<R & R2, E | E2, A>

export function bracketExit<R, E, A, R2, E2>(
  acquire: T.Effect<R, E, A>,
  release: FunctionN<[A, Exit<E, unknown>], T.Effect<R2, E2, unknown>>
): Managed<R & R2, E | E2, A>

/**
 * Lift an IO of a Resource into a resource
 * @param suspended
 */
export function suspend<R, E, R2, E2, A>(
  suspended: T.Effect<R, E, Managed<R2, E2, A>>
): Managed<R & R2, E | E2, A> 

/**
 * Compose dependent resourcess.
 *
 * The scope of left will enclose the scope of the resource produced by bind
 * @param bind
 */
export function chain<R, E, L, A>(
  bind: FunctionN<[L], Managed<R, E, A>>
): <R2, E2>(ma: Managed<R2, E2, L>) => Managed<R & R2, E | E2, A>

/**
 * Map a resource
 * @param f
 */
export function map<L, A>(
  f: FunctionN<[L], A>
): <R, E>(res: Managed<R, E, L>) => Managed<R, E, A>

/**
 * Zip two resources together with the given function.
 *
 * The scope of resa will enclose the scope of resb
 * @param resa
 * @param resb
 * @param f
 */
export function zipWith<R, E, A, R2, E2, B, C>(
  resa: Managed<R, E, A>,
  resb: Managed<R2, E2, B>,
  f: FunctionN<[A, B], C>
): Managed<R & R2, E | E2, C>

/**
 * Zip two resources together as a tuple.
 *
 * The scope of resa will enclose the scope of resb
 * @param resa
 * @param resb
 */
export function zip<R, E, A, R2, E2, B>(
  resa: Managed<R, E, A>,
  resb: Managed<R2, E2, B>
): Managed<R & R2, E | E2, readonly [A, B]>

/**
 * Apply the function produced by resfab to the value produced by resa to produce a new resource.
 * @param resa
 * @param resfab
 */
export function ap<R, E, A, R2, E2, B>(
  resa: Managed<R, E, A>,
  resfab: Managed<R2, E2, FunctionN<[A], B>>
): Managed<R & R2, E | E2, B>

/**
 * Flipped version of ap
 * @param resfab
 * @param resa
 */
function ap_<R, E, A, B, R2, E2>(
  resfab: Managed<R, E, FunctionN<[A], B>>,
  resa: Managed<R2, E2, A>
): Managed<R & R2, E | E2, B> 

/**
 * Map a resource to a static value
 *
 * This creates a resource of the provided constant b where the produced A has the same lifetime internally
 * @param fa
 * @param b
 */
export function as<R, E, A, B>(fa: Managed<R, E, A>, b: B): Managed<R, E, B>

/**
 * Curried form of as
 * @param b
 */
export function to<B>(
  b: B
): <R, E, A>(fa: Managed<R, E, A>) => Managed<R, E, B>

/**
 * Construct a new 'hidden' resource using the produced A with a nested lifetime
 * Useful for performing initialization and cleanup that clients don't need to see
 * @param left
 * @param bind
 */
export function chainTap<R, E, A, R2, E2>(
  left: Managed<R, E, A>,
  bind: FunctionN<[A], Managed<R2, E2, unknown>>
): Managed<R & R2, E | E2, A>

/**
 * Curried form of chainTap
 * @param bind
 */
export function chainTapWith<R, E, A>(
  bind: FunctionN<[A], Managed<R, E, unknown>>
): FunctionN<[Managed<R, E, A>], Managed<R, E, A>>

/**
 * Curried data last form of use
 * @param f
 */
export function consume<R, E, A, B>(
  f: FunctionN<[A], T.Effect<R, E, B>>
): <R2, E2>(ma: Managed<R2, E2, A>) => T.Effect<R & R2, E | E2, B> 

/**
 * Create a Resource from the fiber of an IO.
 * The acquisition of this resource corresponds to forking rio into a fiber.
 * The destruction of the resource is interrupting said fiber.
 * @param rio
 */
export function fiber<R, E, A>(
  rio: T.Effect<R, E, A>
): Managed<R, never, T.Fiber<E, A>> 

/**
 * Use a resource to produce a program that can be run.s
 * @param res
 * @param f
 */
export function use<R, E, A, R2, E2, B>(
  res: Managed<R, E, A>,
  f: FunctionN<[A], T.Effect<R2, E2, B>>
): T.Effect<R & R2, E | E2, B> 

export interface Leak<R, E, A> {
  a: A;
  release: T.Effect<R, E, unknown>;
}

/**
 * Create an IO action that will produce the resource for this managed along with its finalizer
 * action seperately.
 *
 * If an error occurs during allocation then any allocated resources should be cleaned up, but once the
 * Leak object is produced it is the callers responsibility to ensure release is invoked.
 * @param res
 */
export function allocate<R, E, A>(
  res: Managed<R, E, A>
): T.Effect<R, E, Leak<R, E, A>>

/**
 * Use a resource to provide the environment to a WaveR
 * @param man
 * @param ma
 */
export function provideTo<R extends T.Env, E, R2 extends T.Env, A, E2>(
  man: Managed<R, E, R2>,
  ma: T.Effect<R2, E2, A>
): T.Effect<R, E | E2, A>

// Instances
export const managed: Monad3E<URI> = {
  URI,
  of: pure,
  map: map_,
  ap: ap_,
  chain: chain_
} as const;

// semigroup for result combination
export function getSemigroup<R, E, A>(
  Semigroup: Semigroup<A>
): Semigroup<Managed<R, E, A>> 

// get monoid to compose result
export function getMonoid<R, E, A>(
  Monoid: Monoid<A>
): Monoid<Managed<R, E, A>> 

// provide environment
export function provideAll<R>(
  r: R
): <E, A>(ma: Managed<R, E, A>) => Managed<T.NoEnv, E, A>
```

## Usage

```typescript
import { effect as T, managed as M } from "@matechs/effect";
import { Do } from "fp-ts-contrib/lib/Do";

const db: { kv: Record<string, string> } = { kv: {} }; // simulate a resource

const database = M.bracket(T.pure(db), ref =>
  T.sync(() => {
    ref.kv = {}; // cleanup resource on release
  })
);

const program = M.use(database, ref => // use the resource
  Do(T.effect)
    .do(
      T.sync(() => {
        ref.kv["foo"] = "bar"; // write data
      })
    )
    .do(
      T.sync(() => {
        ref.kv["bar"] = "gin"; // write data
      })
    )
    .do(T.raiseError("error!")) // raise an error
    .return(() => {})
);

T.run(program, () => {
  console.log(db); // logs => { kv: {} } (release invoked)
});
```

Note that `Managed` can represent any resource for example `database connections` and anything that need safe cleanup, additionally finalizers \(like release in this case\) are guaranteed to run even if effect is interrupted while the resource is in usage.

