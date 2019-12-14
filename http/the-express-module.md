---
description: Provides an environmental effect that wraps and express app
---

# The Express Module

## Installation

```text
yarn add express @matechs/express
yarn add -D @types/express
```

## Module

```typescript
import newExpress from "express";
import { effect as T } from "@matechs/effect";
import * as EX from "express";
import * as bodyParser from "body-parser";
import { Server } from "http";

// environment entries
export const expressAppEnv: unique symbol = Symbol();
export const expressEnv: unique symbol = Symbol();

// environment to hold the app instance
export interface HasExpress {
  [expressAppEnv]: {
    app: EX.Express;
  };
}

// http method
export type Method = "post" | "get" | "put" | "patch" | "delete";

// effect
export interface Express {
  [expressEnv]: {
    withApp<R, E, A>(op: T.Effect<R & HasExpress, E, A>): T.Effect<R, E, A>;
    route<R, E, RES>(
      method: Method,
      path: string,
      f: (req: EX.Request) => T.Effect<R, E, RES>
    ): T.Effect<R & HasExpress, T.NoErr, void>;
    bind(
      port: number,
      hostname?: string
    ): T.Effect<HasExpress, T.NoErr, Server>;
  };
}

// provides a new app into env of op
export function withApp<R, E, A>(
  op: T.Effect<R & HasExpress, E, A>
): T.Effect<Express & R, E, A>

// bind a new route to express
export function route<R, E, RES>(
  method: Method,
  path: string,
  f: (req: EX.Request) => T.Effect<R, E, RES>
): T.Effect<R & HasExpress & Express, T.NoErr, void> 

// listen on hostname:port
export function bind(
  port: number,
  hostname?: string // default 127.0.0.1
): T.Effect<HasExpress & Express, T.NoErr, Server>

// access express app
export function accessAppM<R, E, A>(
  f: (app: EX.Express) => T.Effect<R, E, A>
): T.Effect<HasExpress & Express & R, E, A> 

// access express app purely
export function accessApp<A>(
  f: (app: EX.Express) => A
): T.Effect<HasExpress & Express, T.NoErr, A> 

// environment for consumer usage
export type ExpressEnv = HasExpress & Express;

// implementation
export const express: Express
```

## Usage

```typescript
import { effect as T, exit as E } from "@matechs/effect";
import * as EX from "@matechs/express";
import { Do } from "fp-ts-contrib/lib/Do";
import { pipe } from "fp-ts/lib/pipeable";

// create a new express server
const program = EX.withApp(
  Do(T.effect)
    .do(
      // bind GET / to {"message":"OK"}
      EX.route("get", "/", () =>
        T.sync(() => ({
          message: "OK"
        }))
      )
    )
    .bind("server", EX.bind(8081)) // listen on port 8081
    .return(s => s.server) // return node server
);

// construct live environment
const envLive = pipe(T.noEnv, T.mergeEnv(EX.express));

// run express server
T.run(
  T.provideAll(envLive)(program),
  E.fold(
    server => {
      // listen for exit Ctrl+C
      process.on("SIGINT", () => {
        server.close(err => {
          process.exit(err ? 2 : 0);
        });
      });
      
      // listen for SIGTERM
      process.on("SIGTERM", () => {
        server.close(err => {
          process.exit(err ? 2 : 0);
        });
      });
    },
    e => console.error(e),
    e => console.error(e),
    () => console.error("interrupted")
  )
);
```

```typescript
$ curl http://127.0.0.1:8081/
{"message":"OK"}
```

## Notes

The module is a work in progress and api is expected to be changed \(not significantly\)

