---
description: Potentially infinite effectful streams!
---

# Play with Streams

## Stream

```typescript
/**
 * Stream represents a potentially infinite stream of effectful computations
 * It is a more general structure in respect to the base effect
 * that allows running computations in a rective manner
 */
export interface Stream<R, E, A> {}

/**
 * Create a Stream from a source A action.
 *
 * The contract is that the acquisition of the resource should produce a Wave that may be repeatedly evaluated
 * during the scope of the Managed
 * If there is more data in the stream, the Wave should produce some(A) otherwise it should produce none.
 * Once it produces none, it will not be evaluated again.
 * @param r
 */
export function fromSource<R, E, A>(
  r: Managed<R, E, T.Effect<R, E, Option<A>>>
): Stream<R, E, A> 

/**
 * Create a stream from an Array
 * @param as
 */
export function fromArray<A>(as: readonly A[]): Stream<T.NoEnv, T.NoErr, A> 

/**
 * Create a stream from an iterator
 * @param iter
 */
export function fromIterator<A>(
  iter: Lazy<Iterator<A>>
): Stream<T.NoEnv, T.NoErr, A> 

/**
 * Create a stream that emits the elements in a range
 * @param start
 * @param interval
 * @param end
 */
export function fromRange(
  start: number,
  interval?: number,
  end?: number
): Stream<T.NoEnv, T.NoErr, number>

/**
 * Create a stream from an existing iterator
 * @param iter
 */
export function fromIteratorUnsafe<A>(
  iter: Iterator<A>
): Stream<T.NoEnv, T.NoErr, A>

/**
 * Create a stream that emits a single element
 * @param a
 */
export function once<A>(a: A): Stream<T.NoEnv, T.NoErr, A> 

/**
 * Create a stream that emits As as fast as possible
 *
 * Be cautious when using this. If your entire pipeline is full of synchronous actions you can block the main
 * thread until the stream runs to completion (or forever) using this
 * @param a
 */
export function repeatedly<A>(a: A): Stream<T.NoEnv, T.NoErr, A> 

export function periodically(ms: number): Stream<T.NoEnv, T.NoErr, number> 

/**
 * A stream that emits no elements an immediately terminates
 */
export const empty: Stream<
  T.NoEnv,
  T.NoErr,
  never
> 

/**
 * Create a stream that evalutes w to emit a single element
 * @param w
 */
export function encaseEffect<R, E, A>(w: T.Effect<R, E, A>): Stream<R, E, A> 

/**
 * Create a stream that immediately fails
 * @param e
 */
export function raised<E>(e: E): Stream<T.NoEnv, E, never> 

/**
 * Create a stream that immediately aborts
 * @param e
 */
export function aborted(e: unknown): Stream<T.NoEnv, T.NoErr, never> 

/**
 * Create a stream that immediately emits either 0 or 1 elements
 * @param opt
 */
export function fromOption<A>(opt: Option<A>): Stream<T.NoEnv, T.NoErr, A> 

/**
 * Zip all stream elements with their index ordinals
 * @param stream
 */
export function zipWithIndex<R, E, A>(
  stream: Stream<R, E, A>
): Stream<R, E, readonly [A, number]> 

/**
 * Create a stream that emits all the elements of stream1 followed by all the elements of stream2
 * @param stream1
 * @param stream2
 */
export function concatL<R, E, A, R2, E2>(
  stream1: Stream<R, E, A>,
  stream2: Lazy<Stream<R2, E2, A>>
): Stream<R & R2, E | E2, A> 

/**
 * Strict form of concatL
 * @param stream1
 * @param stream2
 */
export function concat<R, E, A, R2, E2>(
  stream1: Stream<R, E, A>,
  stream2: Stream<R2, E2, A>
): Stream<R & R2, E | E2, A>

/**
 * Creates a stream that repeatedly emits the elements of a stream forever.
 *
 * The elements are not cached, any effects required (i.e. opening files or sockets) are repeated for each cycle
 * @param stream
 */
export function repeat<R, E, A>(stream: Stream<R, E, A>): Stream<R, E, A> 

/**
 * Map the elements of a stream
 * @param stream
 * @param f
 */
export function map<R, A, B>(
  f: FunctionN<[A], B>
): <E>(stream: Stream<R, E, A>) => Stream<R, E, B> 

/**
 * Map every element emitted by stream to b
 * @param stream
 * @param b
 */
export function as<R, E, A, B>(stream: Stream<R, E, A>, b: B): Stream<R, E, B> 

/**
 * Filter the elements of a stream by a predicate
 * @param stream
 * @param f
 */
export function filter<R, E, A>(
  stream: Stream<R, E, A>,
  f: Predicate<A>
): Stream<R, E, A>

/**
 * Curried form of filter
 * @param f
 */
export function filterWith<A>(
  f: Predicate<A>
): <R, E>(stream: Stream<R, E, A>) => Stream<R, E, A> 

// filter and refine
export function filterRefineWith<A, B extends A>(
  f: Refinement<A, B>
): <R, E>(stream: Stream<R, E, A>) => Stream<R, E, B> 

/**
 * Filter the stream so that only items that are not equal to the previous item emitted are emitted
 * @param eq
 */
export function distinctAdjacent<A>(
  eq: Eq<A>
): <R, E>(stream: Stream<R, E, A>) => Stream<R, E, A> 

/**
 * Fold the elements of this stream together using an effect.
 *
 * The resulting stream will emit 1 element produced by the effectful fold
 * @param stream
 * @param f
 * @param seed
 */
export function foldM<R, E, A, R2, E2, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[B, A], T.Effect<R2, E2, B>>,
  seed: B
): Stream<R & R2, E | E2, B> 

/**
 * Fold the elements of a stream together purely
 * @param stream
 * @param f
 * @param seed
 */
export function fold<R, E, A, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[B, A], B>,
  seed: B
): Stream<R, E, B> 

/**
 * Scan across the elements the stream.
 *
 * This is like foldM but emits every intermediate seed value in the resulting stream.
 * @param stream
 * @param f
 * @param seed
 */
export function scanM<R, E, A, B, R2, E2>(
  stream: Stream<R, E, A>,
  f: FunctionN<[B, A], T.Effect<R2, E2, B>>,
  seed: B
): Stream<R & R2, E | E2, B> 

/**
 * Purely scan a stream
 * @param stream
 * @param f
 * @param seed
 */
export function scan<R, E, A, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[B, A], B>,
  seed: B
): Stream<R, E, B>

/**
 * Monadic chain on a stream
 * @param stream
 * @param f
 */
export function chain<A, R2, E2, B>(
  f: FunctionN<[A], Stream<R2, E2, B>>
): <R, E>(stream: Stream<R, E, A>) => Stream<R & R2, E | E2, B> 

/**
 * Flatten a stream of streams
 * @param stream
 */
export function flatten<R, E, R2, E2, A>(
  stream: Stream<R, E, Stream<R2, E2, A>>
): Stream<R & R2, E | E2, A> 

/**
 * Map each element of the stream effectfully
 * @param stream
 * @param f
 */
export function mapM<R, E, A, R2, E2, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[A], T.Effect<R2, E2, B>>
): Stream<R & R2, E | E2, B> 

export function mapMWith<A, R2, E2, B>(
  f: FunctionN<[A], T.Effect<R2, E2, B>>
): <R, E>(stream: Stream<R, E, A>) => Stream<R & R2, E | E2, B> 

/**
 * A stream that emits no elements but never terminates.
 */
export const never: Stream<T.NoEnv, T.NoErr, never> 

/**
 * Transduce a stream via a sink.
 *
 * This repeatedly run a sink to completion on the elements of the input stream and emits the result of each run
 * Leftovers from a previous run are fed to the next run
 *
 * @param stream
 * @param sink
 */
export function transduce<R, E, A, R2, E2, S, B>(
  stream: Stream<R, E, A>,
  sink: Sink<R2, E2, S, A, B>
): Stream<R & R2, E | E2, B> 

/**
 * Drop some number of elements from a stream
 *
 * Their effects to be produced still occur in the background
 * @param stream
 * @param n
 */
export function drop<R, E, A>(
  stream: Stream<R, E, A>,
  n: number
): Stream<R, E, A> 

/**
 * Curried form of drop
 * @param n
 */
export function dropWith(
  n: number
): <R, E, A>(stream: Stream<R, E, A>) => Stream<R, E, A> {
  return stream => drop(stream, n);
}

/**
 * Take some number of elements of a stream
 * @param stream
 * @param n
 */
export function take<R, E, A>(
  stream: Stream<R, E, A>,
  n: number
): Stream<R, E, A> 

/**
 * Take elements of a stream while a predicate holds
 * @param stream
 * @param pred
 */
export function takeWhile<R, E, A>(
  stream: Stream<R, E, A>,
  pred: Predicate<A>
): Stream<R, E, A> 

/**
 * Push a stream into a sink to produce the sink's result
 * @param stream
 * @param sink
 */
export function into<R, E, A, R2, E2, S, B>(
  stream: Stream<R, E, A>,
  sink: Sink<R, E2, S, A, B>
): T.Effect<R & R2, E | E2, B> 

/**
 * Push a stream into a sink to produce the sink's result
 * @param stream
 * @param managedSink
 */
export function intoManaged<R, E, A, S, B>(
  stream: Stream<R, E, A>,
  managedSink: Managed<R, E, Sink<R, E, S, A, B>>
): T.Effect<R, E, B> 

/**
 * Push a stream in a sink to produce the result and the leftover
 * @param stream
 * @param sink
 */
export function intoLeftover<R, E, A, S, B>(
  stream: Stream<R, E, A>,
  sink: Sink<R, E, S, A, B>
): T.Effect<R, E, readonly [B, readonly A[]]> 

/**
 * Zip two streams together termitating when either stream is exhausted
 * @param as
 * @param bs
 * @param f
 */
export function zipWith<R, E, A, R2, E2, B, C>(
  as: Stream<R, E, A>,
  bs: Stream<R2, E2, B>,
  f: FunctionN<[A, B], C>
): Stream<R & R2, E | E2, C> 

/**
 * zipWith to form tuples
 * @param as
 * @param bs
 */
export function zip<R, E, A, R2, E2, B>(
  as: Stream<R, E, A>,
  bs: Stream<R2, E2, B>
): Stream<R & R2, E | E2, readonly [A, B]> 

/**
 * Feed a stream into a sink to produce a value.
 *
 * Emits the value and a 'remainder' stream that includes the rest of the elements of the input stream.
 * @param stream
 * @param sink
 */
export function peel<R, E, A, S, B>(
  stream: Stream<R, E, A>,
  sink: Sink<R, E, S, A, B>
): Stream<R, E, readonly [B, Stream<R, E, A>]> 

export function peelManaged<R, E, A, S, B>(
  stream: Stream<R, E, A>,
  managedSink: Managed<R, E, Sink<R, E, S, A, B>>
): Stream<R, E, readonly [B, Stream<R, E, A>]>

/**
 * Create a stream that switches to emitting elements of the most recent input stream.
 * @param stream
 */
export function switchLatest<R, E, A>(
  stream: Stream<R, E, Stream<R, E, A>>
): Stream<R, E, A> 

/**
 * Create a stream that switches to emitting the elements of the most recent stream produced by applying f to the
 * element most recently emitted
 * @param stream
 * @param f
 */
export function chainSwitchLatest<R, E, A, R2, E2, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[A], Stream<R2, E2, B>>
): Stream<R & R2, E | E2, B> 

/**
 * Merge a stream of streams into a single stream.
 *
 * This stream will run up to maxActive streams concurrently to produce values into the output stream.
 * @param stream the input stream
 * @param maxActive the maximum number of streams to hold active at any given time
 * this controls how much active streams are able to collectively produce in the face of a slow downstream consumer
 */
export function merge<R, E, A>(
  stream: Stream<R, E, Stream<R, E, A>>,
  maxActive: number
): Stream<R, E, A> 

export function chainMerge<R, E, A, B>(
  stream: Stream<R, E, A>,
  f: FunctionN<[A], Stream<R, E, B>>,
  maxActive: number
): Stream<R, E, B> 

export function mergeAll<R, E, A>(
  streams: Array<Stream<R, E, A>>
): Stream<R, E, A> 

/**
 * Drop elements of the stream while a predicate holds
 * @param stream
 * @param pred
 */
export function dropWhile<R, E, A>(
  stream: Stream<R, E, A>,
  pred: Predicate<A>
): Stream<R, E, A> 

/**
 * Collect all the elements emitted by a stream into an array.
 * @param stream
 */
export function collectArray<R, E, A>(
  stream: Stream<R, E, A>
): T.Effect<R, E, A[]> 

/**
 * Evaluate a stream for its effects
 * @param stream
 */
export function drain<R, E, A>(stream: Stream<R, E, A>): T.Effect<R, E, void> 

// instances
export const stream: Monad3E<URI> = {
  URI,
  map: map_,
  of: <R, E, A>(a: A): Stream<R, E, A> => (once(a) as any) as Stream<R, E, A>,
  ap: <R, R2, E, E2, A, B>(
    sfab: Stream<R, E, FunctionN<[A], B>>,
    sa: Stream<R2, E2, A>
  ) => zipWith(sfab, sa, (f, a) => f(a)),
  chain: chain_
} as const;

// encase a node js stream object readable into a stream
export function fromObjectReadStream<A>(stream: Readable) {
  return fromSource(getSourceFromObjectReadStream<A>(stream));
}

// encase a node js stream object readable into a stream (batched)
export function fromObjectReadStreamB<A>(
  stream: ReadStream,
  batch: number,
  every: number
) {
  return fromSource(getSourceFromObjectReadStreamB<A>(stream, batch, every));
}
```

## Sink

```typescript
export interface Sink<R, E, S, A, B> {
  readonly initial: T.Effect<R, E, SinkStep<A, S>>;
  step: (state: S, next: A) => T.Effect<R, E, SinkStep<A, S>>;
  extract: (step: S) => T.Effect<R, E, B>;
}

export interface SinkPure<S, A, B> {
  readonly initial: SinkStep<A, S>;
  step: (state: S, next: A) => SinkStep<A, S>;
  extract: (state: S) => B;
}

/**
 * Step a sink repeatedly.
 * If the sink completes before consuming all of the input, then the done state will include the ops leftovers
 * and anything left in the array
 * @param sink
 * @param s
 * @param multi
 */
export function stepMany<R, E, S, A, B>(
  sink: Sink<R, E, S, A, B>,
  s: S,
  multi: readonly A[]
): T.Effect<R, E, SinkStep<A, S>> 

export function liftPureSink<S, A, B>(
  sink: SinkPure<S, A, B>
): Sink<T.NoEnv, T.NoErr, S, A, B> 

export function collectArraySink<R, E, A>(): Sink<R, E, A[], A, A[]> 

export function drainSink<R, E, A>(): Sink<R, E, void, A, void> 

/**
 * A sink that consumes no input to produce a constant b
 * @param b
 */
export function constSink<R, E, A, B>(b: B): Sink<R, E, void, A, B> 

/**
 * A sink that produces the head element of a stream (if any elements are emitted)
 */
export function headSink<R, E, A>(): Sink<R, E, Option<A>, A, Option<A>> 

/**
 * A sink that produces the last element of a stream (if any elements are emitted)
 */
export function lastSink<R, E, A>(): Sink<R, E, Option<A>, A, Option<A>> 

/**
 * A sink that evalutes an action for every element of a sink and produces no value
 * @param f
 */
export function evalSink<R, E, A>(
  f: FunctionN<[A], T.Effect<R, E, unknown>>
): Sink<R, E, void, A, void> 

/**
 * A sink that consumes elements for which a predicate does not hold.
 *
 * Returns the first element for which the predicate did hold if such an element is found.
 * @param f
 */
export function drainWhileSink<R, E, A>(
  f: Predicate<A>
): Sink<R, E, Option<A>, A, Option<A>> 

/**
 * A sink that offers elements into a concurrent queue
 *
 * @param queue
 */
export function queueSink<R, E, A>(
  queue: ConcurrentQueue<A>
): Sink<R, E, void, A, void> 

/**
 * A sink that offers elements into a queue after wrapping them in an option.
 *
 * The sink will offer one final none into the queue when the stream terminates
 * @param queue
 */
export function queueOptionSink<R, E, A>(
  queue: ConcurrentQueue<Option<A>>
): Sink<R, E, void, A, void> 

/**
 * Map the output value of a sink
 * @param sink
 * @param f
 */
export function map<R, E, S, A, B, C>(
  sink: Sink<R, E, S, A, B>,
  f: FunctionN<[B], C>
): Sink<R, E, S, A, C> 
```

## Usage

```typescript
import { effect as T, stream as S } from "@matechs/effect";
import { pipe } from "fp-ts/lib/pipeable";

const program = pipe(
  S.periodically(100),
  S.chain(n =>
    S.encaseEffect(
      T.sync(() => {
        console.log(`got: ${n}`);

        return n + 1;
      })
    )
  ),
  S.filterWith(n => n > 5)
);

T.runToPromise(S.collectArray(S.take(program, 10))).then(r => {
  console.log(`elements: ${r.length}`);
});

const program2 = pipe(S.repeatedly(1), s =>
  S.foldM(S.take(s, 100000), (acc, n) => T.sync(() => acc + n), 0)
);

T.runToPromise(S.collectArray(program2)).then(r => {
  console.log(`fold: ${r[0]}`);
});

// prints
// fold: 100000
// got: 0
// got: 1
// got: 2
// got: 3
// got: 4
// got: 5
// got: 6
// got: 7
// got: 8
// got: 9
// got: 10
// got: 11
// got: 12
// got: 13
// got: 14
// elements: 10
```

Note that errors are final if not handled in place, if you want a non final implementation you may use a `Stream<R, never, Either<E,A>>` where you can manage how error is propagated.

An implementation of `StreamEither` will be provided in work in progress and will be added to the core. 

### takeWhile

```ts
import { log, provideConsole } from "@matechs/console";

const sleep = (ms: number) => pipe(T.delay(log("I woke up!"), ms));

pipe(
  S.periodically(1000),
  takeUntil(sleep(5000)),
  S.chain((n) => S.encaseEffect(log(n))),
  S.drain,
  provideConsole,
  T.run
);

// prints
// 0
// 1
// 2
// 3
// I woke up!
```

