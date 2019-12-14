---
description: >-
  Describe service interactions using effects, call your server from the browser
  and forget that API even exists.
---

# The RPC Module

## Installation

```text
yarn add express @matechs/express @matechs/rpc @matechs/http-client
yarn add -D @types/express

// one http implementation like
yarn add @matechs/http-client-libcurl
```

## Module

```typescript
import { effect as T } from "@matechs/effect";
import { FunctionN } from "fp-ts/lib/function";
import * as H from "@matechs/http-client";
import * as E from "@matechs/express";
import { array } from "fp-ts/lib/Array";
import { right } from "fp-ts/lib/Either";
import { Exit } from "@matechs/effect/lib/original/exit";
import { Do } from "fp-ts-contrib/lib/Do";
import { isSome } from "fp-ts/lib/Option";

// environment entries
const clientConfigEnv: unique symbol = Symbol();
const serverConfigEnv: unique symbol = Symbol();

// describean environment that support RPC
export type Remote<T> = Record<
  keyof T,
  Record<string, FunctionN<any, T.Effect<any, any, any>>>
>;

// describe a client configuration
interface ClientConfig<M, K extends keyof M> {
  [clientConfigEnv]: {
    [k in K]: {
      baseUrl: string;
    };
  };
}

// create a client configuration
export function clientConfig<M, K extends keyof M>(
  _m: M,
  k: K
): (c: ClientConfig<M, K>[typeof clientConfigEnv][K]) => ClientConfig<M, K>

// describe a server configuration
interface ServerConfig<M, K extends keyof M> {
  [serverConfigEnv]: {
    [k in K]: {
      scope: string;
    };
  };
}

// create a server configuration
export function serverConfig<M, K extends keyof M>(
  _m: M,
  k: K
): (c: ServerConfig<M, K>[typeof serverConfigEnv][K]) => ServerConfig<M, K> 

// create a client for module M and entry K
export function client<M extends Remote<M>, K extends keyof M>(m: M, k: K): Client<M, K> 

// merge environment requirements for the whole module
export type Runtime<M> = M extends {
  [h: string]: (...args: any[]) => T.Effect<infer Q, any, any>;
}
  ? Q
  : never;

// request object
interface RPCRequest {
  args: unknown[];
}

// response object
interface RPCResponse {
  value: Exit<unknown, unknown>;
}

// bind module M and entry K to express
export function bind<M extends Remote<M>, K extends keyof M>(
  m: M,
  k: K
): T.Effect<
  E.ExpressEnv & Runtime<M[K]> & ServerConfig<M, K> & M,
  T.NoErr,
  void
>
```

## Usage

{% code title="shared.ts" %}
```typescript
import { effect as T } from "@matechs/effect";
import * as RPC from "@matechs/rpc";
import * as H from "@matechs/http-client";
import { Option } from "fp-ts/lib/Option";
import { pipe } from "fp-ts/lib/pipeable";

// environment entries
export const placeholderJsonEnv: unique symbol = Symbol();

// simple todo interface
export interface Todo {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}

// describe the service we want to expose
export interface PlaceholderJson extends RPC.Remote<PlaceholderJson> {
  [placeholderJsonEnv]: {
    getTodo: (n: number) => T.Effect<H.RequestEnv, string, Option<Todo>>;
  };
}

/*
 * Note this is the real implementation, but it doesn't depend
 * on any library directly only through Env
 * in this case this can be shared between server and client
 * if you want to move this to server code only
 * you can create a dumb implementation like below
 * that is considered enough for client generation
 */

// implement the service
export const placeholderJsonLive: PlaceholderJson = {
  [placeholderJsonEnv]: {
    getTodo: n =>
      pipe(
        H.get<unknown, Todo>(`https://jsonplaceholder.typicode.com/todos/${n}`),
        T.chainError(() => T.raiseError("error fetching todo")),
        T.map(({ body }) => body)
      )
  }
};

// spec for client generation
// used if placeholderJsonLive -> server.ts
export const placeholderJsonSpec: PlaceholderJson = {
  [placeholderJsonEnv]: {
    getTodo: {} as any
  }
};
```
{% endcode %}

{% code title="server.ts" %}
```typescript
import { effect as T, exit as E } from "@matechs/effect";
import * as RPC from "@matechs/rpc";
import * as H from "@matechs/http-client";
import * as EX from "@matechs/express";
import * as L from "@matechs/http-client-libcurl";
import { pipe } from "fp-ts/lib/pipeable";
import { Do } from "fp-ts-contrib/lib/Do";
import { placeholderJsonLive, placeholderJsonEnv } from "./shared";

// create a new express server
const program = EX.withApp(
  Do(T.effect)
    // wire module to express
    .do(RPC.bind(placeholderJsonLive, placeholderJsonEnv))
    // listen on port 8081
    .bind("server", EX.bind(8081))
    // return node server
    .return(s => s.server)
);

// construct live environment
const envLive = pipe(
  T.noEnv,
  T.mergeEnv(EX.express),
  T.mergeEnv(L.libcurl()),
  T.mergeEnv(H.jsonDeserializer),
  T.mergeEnv(placeholderJsonLive),
  // configure RPC server for module <placeholderJsonLive, placeholderJsonEnv>
  T.mergeEnv(
    RPC.serverConfig(
      placeholderJsonLive,
      placeholderJsonEnv
    )({
      scope: "/placeholderJson" // exposed at /placeholderJson
    })
  )
);

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
{% endcode %}

{% code title="client.ts" %}
```typescript
import { effect as T, exit as E } from "@matechs/effect";
import * as RPC from "@matechs/rpc";
import * as H from "@matechs/http-client";
import * as L from "@matechs/http-client-libcurl";
import { pipe } from "fp-ts/lib/pipeable";
import { placeholderJsonLive, placeholderJsonEnv } from "./shared";
import * as A from "fp-ts/lib/Array";

const { getTodo } = RPC.client(placeholderJsonLive, placeholderJsonEnv);

const program = A.array.traverse(T.parEffect)(A.range(1, 10), getTodo);

const envLive = pipe(
  T.noEnv,
  T.mergeEnv(L.libcurl()),
  T.mergeEnv(H.jsonDeserializer),
  T.mergeEnv(
    RPC.clientConfig(
      placeholderJsonLive,
      placeholderJsonEnv
    )({
      baseUrl: "http://127.0.0.1:8081/placeholderJson"
    })
  )
);

// run express server
T.run(
  T.provideAll(envLive)(program),
  E.fold(
    todos => {
      console.log(todos);
    },
    e => console.error(e),
    e => console.error(e),
    () => console.error("interrupted")
  )
);
```
{% endcode %}

```typescript
$ yarn ts-node src/server.ts
...
```

```typescript
$ yarn ts-node src/client.ts
[
  {
    _tag: 'Some',
    value: { userId: 1, id: 1, title: 'delectus aut autem', completed: false }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 2,
      title: 'quis ut nam facilis et officia qui',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 3,
      title: 'fugiat veniam minus',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: { userId: 1, id: 4, title: 'et porro tempora', completed: true }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 5,
      title: 'laboriosam mollitia et enim quasi adipisci quia provident illum',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 6,
      title: 'qui ullam ratione quibusdam voluptatem quia omnis',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 7,
      title: 'illo expedita consequatur quia in',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 8,
      title: 'quo adipisci enim quam ut ab',
      completed: true
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 9,
      title: 'molestiae perspiciatis ipsa',
      completed: false
    }
  },
  {
    _tag: 'Some',
    value: {
      userId: 1,
      id: 10,
      title: 'illo est ratione doloremque quia maiores aut',
      completed: true
    }
  }
]
```

## Note

You can wire as many modules as you need! Keep in mind both arguments and return types \(error/success\) must be serializable in order for RPC to do its magic.

Full example with complete separation of server/client available at [https://github.com/mikearnaldi/matechs-effect/tree/master/packages/rpc/demo](https://github.com/mikearnaldi/matechs-effect/tree/master/packages/rpc/demo) 

