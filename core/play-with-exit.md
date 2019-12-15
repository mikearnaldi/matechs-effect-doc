---
description: Exit represents the result of an effect execution
---

# Play with Exit

```typescript
import { exit } from "@matechs/effect"
```

Following the exposed module

```typescript
export const isDone = <E, A>(e: Exit<E, A>): e is Done<A> 
export const isRaise = <E, A>(e: Exit<E, A>): e is Raise<E> 
export const isAbort = <E, A>(e: Exit<E, A>): e is Abort
export const isInterrupt = <E, A>(e: Exit<E, A>): e is Interrupt

export function fold<E, A, R>(
  onDone: (v: A) => R,
  onRaise: (v: E) => R,
  onAbort: (v: unknown) => R,
  onInterrupt: () => R
): (e: Exit<E, A>) => R
```

The `Exit` ADT located at `@matechs/effect/lib/original/exit`:

```typescript
export type Exit<E, A> = Done<A> | Cause<E>;
export type ExitTag = Exit<unknown, unknown>["_tag"];

export interface Done<A> {
  readonly _tag: "Done";
  readonly value: A;
}

export function done<A>(v: A): Done<A> {
  return {
    _tag: "Done",
    value: v
  };
}

export type Cause<E> = Raise<E> | Abort | Interrupt;

export interface Raise<E> {
  readonly _tag: "Raise";
  readonly error: E;
}

export function raise<E>(e: E): Raise<E> {
  return {
    _tag: "Raise",
    error: e
  };
}

export interface Abort {
  readonly _tag: "Abort";
  readonly abortedWith: unknown;
}

export function abort(a: unknown): Abort {
  return {
    _tag: "Abort",
    abortedWith: a
  };
}

export interface Interrupt {
  readonly _tag: "Interrupt";
}

export const interrupt: Interrupt = {
  _tag: "Interrupt"
};
```

