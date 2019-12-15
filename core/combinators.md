---
description: Core functions available to create and transform effects
---

# Play with Effect

## Module

```typescript
// pure value
export function pure<A>(a: A): Effect<NoEnv, NoErr, A>

// create an errored effect
export function raised<E, A = never>(e: Cause<E>): Effect<NoEnv, E, A>

// raise an error (like "throw")
export function raiseError<E, A = never>(e: E): Effect<NoEnv, E, A>

// raise an abort reason (unknown error)
export function raiseAbort(u: unknown): Effect<NoEnv, NoErr, never>

// raise an interruption signal in the execution fiber
export const raiseInterrupt: Effect<NoEnv, NoErr, never>

// create a completed effect with the same exit as the one provided
export function completed<E, A>(exit: Exit<E, A>): Effect<NoEnv, E, A>

// suspend execution of the thunk (give space for interrupt)
export function suspended<R, E, A>(
  thunk: Lazy<Effect<R, E, A>>
): Effect<R, E, A>

// encase a sync computation
export function sync<E = NoErr, A = unknown>(
  thunk: Lazy<A>
): Effect<NoEnv, E, A>

// try a sync computation
export function trySync<E = unknown, A = unknown>(
  thunk: Lazy<A>
): Effect<NoEnv, E, A>

// try a sync computation mapping the catched exception
export function trySyncMap<E = unknown>(
  onError: (e: unknown) => E
): <A = unknown>(thunk: Lazy<A>) => Effect<NoEnv, E, A> 

// wrap an async callback into an effect, op return Lazy is triggered
// on cancel
export function async<E, A>(
  op: FunctionN<[FunctionN<[Either<E, A>], void>], Lazy<void>>
): Effect<NoEnv, E, A>

// wrap an async callback into an effect that cannot fail
export function asyncTotal<A>(
  op: FunctionN<[FunctionN<[A], void>], Lazy<void>>
): Effect<NoEnv, NoErr, A>

// denote a region (interruptible or not)
export function interruptibleRegion<R, E, A>(
  inner: Effect<R, E, A>,
  flag: boolean
): Effect<R, E, A>

// encase an either into an effect
export function encaseEither<E, A>(e: Either<E, A>): Effect<NoEnv, E, A>

// encase an option into an effect
export function encaseOption<E, A>(
  o: Option<A>,
  onError: Lazy<E>
): Effect<NoEnv, E, A>

// encase nullable pure value
export function fromNullableM<R, E, A>(
  ma: Effect<R, E, A>
): Effect<R, E, Option<A>>

// encase promise
export function fromPromise<A>(
  thunk: Lazy<Promise<A>>
): Effect<NoEnv, unknown, A>

// encase promise mapping error
export function fromPromiseMap<E>(
  onError: (e: unknown) => E
): <A>(thunk: Lazy<Promise<A>>) => Effect<NoEnv, E, A>

// fold an effect handling success and failure
export function foldExit<E1, RF, E2, A1, E3, A2, RS>(
  failure: FunctionN<[Cause<E1>], Effect<RF, E2, A2>>,
  success: FunctionN<[A1], Effect<RS, E3, A2>>
): <R>(io: Effect<R, E1, A1>) => Effect<RF & RS & R, E2 | E3, A2>

// list a function into an effect
export function lift<A, B>(
  f: FunctionN<[A], B>
): <R, E>(io: Effect<R, E, A>) => Effect<R, E, B>

// map to constant
export function as<R, E, A, B>(io: Effect<R, E, A>, b: B): Effect<R, E, B>

export function to<B>(b: B): <R, E, A>(io: Effect<R, E, A>) => Effect<R, E, B>

// chain without changing outout
export function chainTap<R, E, A>(
  bind: FunctionN<[A], Effect<R, E, unknown>>
): <R2, E2>(inner: Effect<R2, E2, A>) => Effect<R & R2, E | E2, A>

// as unit
export function asUnit<R, E, A>(io: Effect<R, E, A>): Effect<R, E, void>

// unit
export const unit: Effect<NoEnv, NoErr, void>

// handle raised error
export function chainError<R, E1, E2, A>(
  f: FunctionN<[E1], Effect<R, E2, A>>
): <R2>(rio: Effect<R2, E1, A>) => Effect<R & R2, E2, A>

// map raised error
export function mapError<E1, E2>(
  f: FunctionN<[E1], E2>
): <R, A>(io: Effect<R, E1, A>) => Effect<R, E2, A>

// zip two effects with a function
export function zipWith<R, E, A, R2, E2, B, C>(
  first: Effect<R, E, A>,
  second: Effect<R2, E2, B>,
  f: FunctionN<[A, B], C>
): Effect<R & R2, E | E2, C>

// zip with identity
export function zip<R, E, A, R2, E2, B>(
  first: Effect<R, E, A>,
  second: Effect<R2, E2, B>
): Effect<R & R2, E | E2, readonly [A, B]>

// zip with first
export function applyFirst<R, E, A, R2, E2, B>(
  first: Effect<R, E, A>,
  second: Effect<R2, E2, B>
): Effect<R & R2, E | E2, A>

// zip with second
export function applySecond<R, E, A, R2, E2, B>(
  first: Effect<R, E, A>,
  second: Effect<R2, E2, B>
): Effect<R & R2, E | E2, B>

// zip with second lazy
export function applySecondL<R, E, A, R2, E2, B>(
  first: Effect<R, E, A>,
  second: Lazy<Effect<R2, E2, B>>
): Effect<R & R2, E | E2, B>

// flipped apply
export function ap__<R, E, A, R2, E2, B>(
  ioa: Effect<R, E, A>,
  iof: Effect<R2, E2, FunctionN<[A], B>>
): Effect<R & R2, E | E2, B>

// parallel zip equivalents
export function parZipWith<R, R2, E, E2, A, B, C>(
  ioa: Effect<R, E, A>,
  iob: Effect<R2, E2, B>,
  f: FunctionN<[A, B], C>
): Effect<R & R2, E | E2, C>

export function parZip<R, R2, E, A, B>(
  ioa: Effect<R, E, A>,
  iob: Effect<R2, E, B>
): Effect<R & R2, E, readonly [A, B]>

export function parApplyFirst<R, R2, E, A, B>(
  ioa: Effect<R, E, A>,
  iob: Effect<R2, E, B>
): Effect<R & R2, E, A>

export function parApplySecond<R, R2, E, A, B>(
  ioa: Effect<R, E, A>,
  iob: Effect<R2, E, B>
): Effect<R & R2, E, B>

export function parAp<R, R2, E, A, B>(
  ioa: Effect<R, E, A>,
  iof: Effect<R2, E, FunctionN<[A], B>>
): Effect<R & R2, E, B>

export function parAp_<R, R2, E, E2, A, B>(
  iof: Effect<R, E, FunctionN<[A], B>>,
  ioa: Effect<R2, E2, A>
): Effect<R & R2, E | E2, B>

// flip A and E
export function flip<R, E, A>(io: Effect<R, E, A>): Effect<R, A, E>

// execute forever or until failure
export function forever<R, E, A>(io: Effect<R, E, A>): Effect<R, E, A>

// get the result of executing io
export function result<R, E, A>(
  io: Effect<R, E, A>
): Effect<R, NoErr, Exit<E, A>>

// make io interruptible
export function interruptible<R, E, A>(io: Effect<R, E, A>): Effect<R, E, A>

// make io non interruptible
export function uninterruptible<R, E, A>(io: Effect<R, E, A>): Effect<R, E, A>

// bracket with full exit control
export function bracketExit<R, E, A, B, R2, E2, R3, E3>(
  acquire: Effect<R, E, A>,
  release: FunctionN<[A, Exit<E | E3, B>], Effect<R2, E2, unknown>>,
  use: FunctionN<[A], Effect<R3, E3, B>>
): Effect<R & R2 & R3, E | E2 | E3, B>

// bracket not dealing with use result
export function bracket<R, E, A, R2, E2, R3, E3, B>(
  acquire: Effect<R, E, A>,
  release: FunctionN<[A], Effect<R2, E2, unknown>>,
  use: FunctionN<[A], Effect<R3, E3, B>>
): Effect<R & R2 & R3, E | E2 | E3, B>

// run finalizer on complete
export function onComplete<R, E, A, R2, E2>(
  ioa: Effect<R, E, A>,
  finalizer: Effect<R2, E2, unknown>
): Effect<R & R2, E | E2, A>
```

```typescript
// run finalizer on interrupt
export function onInterrupted<R, E, A, R2, E2>(
  ioa: Effect<R, E, A>,
  finalizer: Effect<R2, E2, unknown>
): Effect<R & R2, E | E2, A>

// introduce a gap to allow other fibers to execute
export const shifted: Effect<NoEnv, NoErr, void>

// add shift before io
export function shiftBefore<E, A>(
  io: Effect<NoEnv, E, A>
): Effect<NoEnv, E, A>

// add shift after io
export function shiftAfter<E, A>(io: Effect<NoEnv, E, A>): Effect<NoEnv, E, A>

// shift and return control to js thread
export const shiftedAsync: Effect<NoEnv, NoErr, void>

export function shiftAsyncBefore<R, E, A>(
  io: Effect<R, E, A>
): Effect<R, E, A>

export function shiftAsyncAfter<R, E, A>(io: Effect<R, E, A>): Effect<R, E, A>

// never return
export const never: Effect<NoEnv, NoErr, never>

// add delay to effect
export function delay<R, E, A>(
  inner: Effect<R, E, A>,
  ms: number
): Effect<R, E, A>

export function liftDelay(
  ms: number
): <R, E, A>(io: Effect<R, E, A>) => Effect<R, E, A>

// fork into new fiber
export function fork<R, E, A>(
  io: Effect<R, E, A>,
  name?: string
): Effect<R, NoErr, Fiber<E, A>>

// race and fold
export function raceFold<R, R2, R3, R4, E1, E2, E3, A, B, C>(
  first: Effect<R, E1, A>,
  second: Effect<R2, E2, B>,
  onFirstWon: FunctionN<[Exit<E1, A>, Fiber<E2, B>], Effect<R3, E3, C>>,
  onSecondWon: FunctionN<[Exit<E2, B>, Fiber<E1, A>], Effect<R4, E3, C>>
): Effect<R & R2 & R3 & R4, E3, C>

// race with timeout
export function timeoutFold<R, E1, E2, A, B>(
  source: Effect<R, E1, A>,
  ms: number,
  onTimeout: FunctionN<[Fiber<E1, A>], Effect<NoEnv, E2, B>>,
  onCompleted: FunctionN<[Exit<E1, A>], Effect<NoEnv, E2, B>>
): Effect<R, E2, B>

// race and get first (that completes or error)
export function raceFirst<R, R2, E, A>(
  io1: Effect<R, E, A>,
  io2: Effect<R2, E, A>
): Effect<R & R2, E, A>

// race and get first success
export function race<R, R2, E, A>(
  io1: Effect<R, E, A>,
  io2: Effect<R2, E, A>
): Effect<R & R2, E, A>

// convert error to abort
export function orAbort<R, E, A>(io: Effect<R, E, A>): Effect<R, NoErr, A> 

// timeout returning option of result
export function timeoutOption<R, E, A>(
  source: Effect<R, E, A>,
  ms: number
): Effect<R, E, Option<A>>
```

```typescript
// run effect calling callback on completion
export function run<E, A>(
  io: Effect<{}, E, A>,
  callback?: FunctionN<[Exit<E, A>], void>
): Lazy<void>

// run effect as failable promise
export function runToPromise<E, A>(io: Effect<{}, E, A>): Promise<A>

// run effect as non failable promise of exit result
export function runToPromiseExit<E, A>(
  io: Effect<{}, E, A>
): Promise<Exit<E, A>>

// alternatively run fy if fx fails
export function alt<R2, E2, A>(
  fy: () => Effect<R2, E2, A>
): <R, E>(fx: Effect<R, E, A>) => Effect<R & R2, E | E2, A>

// instances
export const effect: EffectMonad = {
  URI,
  map: map_,
  of: pure,
  ap: ap_,
  chain: chain_,
  bimap: bimap_,
  mapLeft: mapLeft_,
  mapError: mapLeft_,
  throwError: raiseError,
  chainError: chainError_,
  foldExit: foldExit_,
  chainTap: chainTap_,
  alt: alt_
};

export const parEffect: Monad3E<URI> & Bifunctor3<URI> & MonadThrow3E<URI> = {
  URI,
  map: map_,
  of: pure,
  ap: parAp_,
  chain: chain_,
  bimap: bimap_,
  mapLeft: mapLeft_,
  throwError: raiseError
};

// pipeable combinators
const {
  ap,
  apFirst,
  apSecond,
  bimap,
  chain,
  chainFirst,
  filterOrElse,
  flatten,
  fromEither,
  fromOption,
  fromPredicate,
  map,
  mapLeft
} = pipeable(effect);

// get a semigroup to combine results
export function getSemigroup<R, E, A>(
  s: Semigroup<A>
): Semigroup<Effect<R, E, A>>

// get a monoid for result
export function getMonoid<R, E, A>(m: Monoid<A>): Monoid<Effect<R, E, A>>

/* conditionals */
export function when(
  predicate: boolean
): <R, E, A>(ma: Effect<R, E, A>) => Effect<R, E, Op.Option<A>>

export function or_(
  predicate: boolean
): <R, E, A>(
  ma: Effect<R, E, A>
) => <R2, E2, B>(
  mb: Effect<R2, E2, B>
) => Effect<R & R2, E | E2, Ei.Either<A, B>>

export function or<R, E, A>(
  ma: Effect<R, E, A>
): <R2, E2, B>(
  mb: Effect<R2, E2, B>
) => (predicate: boolean) => Effect<R & R2, E | E2, Ei.Either<A, B>>

export function or<R, E, A>(
  ma: Effect<R, E, A>
): <R2, E2, B>(
  mb: Effect<R2, E2, B>
) => (predicate: boolean) => Effect<R & R2, E | E2, Ei.Either<A, B>>

export function cond<R, E, A>(
  ma: Effect<R, E, A>
): <R2, E2, B>(
  mb: Effect<R2, E2, B>
) => (predicate: boolean) => Effect<R & R2, E | E2, A | B>

// parallel sequence
export function sequenceP(
  n: number
): <R, E, A>(ops: Array<Effect<R, E, A>>) => Effect<R, E, Array<A>>

// given semigroup for raised error get a semigroup of cause
export function getCauseSemigroup<E>(S: Semigroup<E>): Semigroup<Cause<E>>

// get validation monad given semigroup of error
export function getValidationM<E>(S: Semigroup<E>)

// get validation monad given semigroup of couse
export function getCauseValidationM<E>(
  S: Semigroup<Cause<E>>
): Monad3EC<URI, E> & MonadThrow3EC<URI, E> & Alt3EC<URI, E>

// wrap node standard (inspired by fp-ts taskify)
export function effectify<L, R>(
  f: (cb: (e: L | null | undefined, r?: R) => void) => void
): () => Effect<NoEnv, L, R>;
export function effectify<A, L, R>(
  f: (a: A, cb: (e: L | null | undefined, r?: R) => void) => void
): (a: A) => Effect<NoEnv, L, R>;
export function effectify<A, B, L, R>(
  f: (a: A, b: B, cb: (e: L | null | undefined, r?: R) => void) => void
): (a: A, b: B) => Effect<NoEnv, L, R>;
export function effectify<A, B, C, L, R>(
  f: (a: A, b: B, c: C, cb: (e: L | null | undefined, r?: R) => void) => void
): (a: A, b: B, c: C) => Effect<NoEnv, L, R>;
export function effectify<A, B, C, D, L, R>(
  f: (
    a: A,
    b: B,
    c: C,
    d: D,
    cb: (e: L | null | undefined, r?: R) => void
  ) => void
): (a: A, b: B, c: C, d: D) => Effect<NoEnv, L, R>;
export function effectify<A, B, C, D, E, L, R>(
  f: (
    a: A,
    b: B,
    c: C,
    d: D,
    e: E,
    cb: (e: L | null | undefined, r?: R) => void
  ) => void
): (a: A, b: B, c: C, d: D, e: E) => Effect<NoEnv, L, R>;
export function effectify<L, R>(f: Function): () => Effect<NoEnv, L, R>
```

## Usage

Basic usage, detailed examples will be presented in the following sections

```typescript
import { effect as T, exit as E } from "@matechs/effect";
import { right } from "fp-ts/lib/Either";
import { semigroupSum, fold } from "fp-ts/lib/Semigroup";

const pureValues = T.pure(1);

const syncValues = T.sync(() => 1);

const syncTry = T.trySync<Error>(() => {
  // tslint:disable-next-line: no-string-throw
  throw "error";
});

const asyncValue = T.async<never, number>(r => {
  const timer = setTimeout(() => {
    r(right(10));
  }, 100);

  return () => {
    clearTimeout(timer);
  };
});

const sumOf = fold(T.getSemigroup(semigroupSum))(T.pure(0), [
  pureValues,
  syncValues,
  asyncValue
]);

const foldExit = E.fold(
  a => console.log(a),
  e => console.error("error:", e),
  e => console.error("abort", e),
  () => console.error("interrupt")
);

T.run(pureValues, foldExit); // print 1
T.run(syncValues, foldExit); // print 1
T.run(syncTry, foldExit); // print error: error
T.run(asyncValue, foldExit); // print 10
T.run(sumOf, foldExit); // print 12
```

