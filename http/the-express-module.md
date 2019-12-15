---
description: Provides an environmental effect that wraps and Express application
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
export const requestContextEnv: unique symbol = Symbol();

// environment to hold the app instance
export interface HasExpress {
    app: EX.Express;
  };
}

// http method
export type Method = "post" | "get" | "put" | "patch" | "delete";

// effect
export interface Express {
    withApp<R, E, A>(op: T.Effect<R & HasExpress, E, A>): T.Effect<R, E, A>;
    route<R, E, A>(
      method: Method,
      path: string,
      f: (req: EX.Request) => T.Effect<R, RouteError<E>, RouteResponse<A>>
    ): T.Effect<R & HasExpress, T.NoErr, void>;
    bind(
      port: number,
      hostname?: string
    ): T.Effect<HasExpress, T.NoErr, Server>;
  };
}

// describe an errored response
export interface RouteError<E> {
  status: number;
  body: E;
}

// create error response
export function routeError<E>(status: number, body: E): RouteError<E> {
  return {
    status,
    body
  };
}

// describe successful response
export interface RouteResponse<A> {
  status: number;
  body: A;
}

// create successful response
export function routeResponse<A>(status: number, body: A): RouteResponse<A> {
  return {
    status,
    body
  };
}

// provides a new app into env of op
export function withApp<R, E, A>(
  op: T.Effect<R & HasExpress, E, A>
): T.Effect<Express & R, E, A>

// describe the environment used to carry current request
export interface RequestContext {
  [requestContextEnv]: {
    request: EX.Request;
  };
}

// bind a new route to express
export function route<R, E, A>(
  method: Method,
  path: string,
  handler: T.Effect<R & RequestContext, RouteError<E>, RouteResponse<A>>
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

// access request
export function accessReqM<R, E, A>(
  f: (req: EX.Request) => T.Effect<R, E, A>
): T.Effect<RequestContext & Express & R, E, A>

// access request purely
export function accessReq<A>(
  f: (req: EX.Request) => A
): T.Effect<RequestContext & Express, never, A>

// environment for consumer usage
export type ExpressEnv = HasExpress & Express;

// environment for consumer usage in request
export type ChildEnv = ExpressEnv & RequestContext;

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
      EX.route(
        "get",
        "/",
        T.pure(
          EX.routeResponse(200, {
            message: "OK"
          })
        )
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

The module is a work in progress and API is expected to be changed \(not significantly\).

