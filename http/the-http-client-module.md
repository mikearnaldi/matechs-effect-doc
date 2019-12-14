---
description: >-
  This module exports a generic http client environmental effect with syntactic
  sugar to deal with http requests
---

# The HTTP Client Module

## Feature set

Multiple implementations are available with different tradeoffs:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Package</th>
      <th style="text-align:left">Features</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>@matechs/http-client-fetch</code>
      </td>
      <td style="text-align:left">
        <p>Content: <code>JSON</code>, <code>URLEncoded</code>, <code>Multipart</code>
          <br
          />Protocols: <code>HTTP 1 &amp; 2</code>  <code>(not in node js)</code>
        </p>
        <p>Target: <code>Browser &amp; Node (with any fetch polyfill)</code>
        </p>
        <p>Cancellable:<code> yes (but socket will complete due to fetch)</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@matechs/http-client-libcurl</code>
      </td>
      <td style="text-align:left">
        <p>Content: <code>JSON, URLEncoded (no support for multipart yet)</code>
          <br
          />Protocols: <code>HTTP 1 &amp; 2</code>
        </p>
        <p>Target: <code>Node</code>
        </p>
        <p>Cancellable: <code>yes</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>## Install

```text
yarn add @matechs/http-client
```

## Module

```typescript
import { effect as T } from "@matechs/effect";
import { Predicate } from "fp-ts/lib/function";
import { Option } from "fp-ts/lib/Option";

// various environment entries
export const middlewareStackEnv: unique symbol = Symbol();
export const httpEnv: unique symbol = Symbol();
export const httpHeadersEnv: unique symbol = Symbol();
export const httpDeserializerEnv: unique symbol = Symbol();

// http methods
export enum Method {
  GET,
  POST,
  PUT,
  DELETE,
  PATCH
}

// request content type use for posting data
export type RequestType = "JSON" | "DATA" | "FORM";

// represents an input compatible with type DATA
export interface DataInput {
  [k: string]: unknown;
}

// represents headers
export type Headers = Record<string, string>;

// represents an http response
export interface Response<Body> {
  body: Option<Body>;
  headers: Headers;
  status: number;
}

// represent the error when we have a response
export interface HttpResponseError<ErrorBody> {
  _tag: HttpErrorReason.Response;
  response: Response<ErrorBody>;
}

// represent cases where the request failed, 
// for example malformed or network down
export interface HttpRequestError {
  _tag: HttpErrorReason.Request;
  error: Error;
}

// index error cases
export enum HttpErrorReason {
  Request,
  Response
}

// describe http error
export type HttpError<ErrorBody> =
  | HttpRequestError
  | HttpResponseError<ErrorBody>;

// describe an effect used to deserialize http responses
export interface HttpDeserializer {
  [httpDeserializerEnv]: {
    response: <A>(a: string) => A | undefined;
    errorResponse: <E>(error: string) => E | undefined;
  };
}

// fold on an http error
export function foldHttpError<A, B, ErrorBody>(
  onError: (e: Error) => A,
  onResponseError: (e: Response<ErrorBody>) => B
): (err: HttpError<ErrorBody>) => A | B

// describe an environment used to provide headers
export interface HttpHeaders {
  [httpHeadersEnv]: Record<string, string>;
}

// main http effect
export interface Http {
  [httpEnv]: {
    request: <E, O>(
      method: Method,
      url: string,
      headers: Record<string, string>,
      requestType: RequestType,
      body?: unknown
    ) => T.Effect<HttpDeserializer, HttpError<E>, Response<O>>;
  };
}

// request function type exposed to allow middleware creation
export type RequestF = <R, E, O>(
  method: Method,
  url: string,
  requestType: RequestType,
  body?: unknown
) => T.Effect<RequestEnv & R, HttpError<E>, Response<O>>;

// describe a middleware that will be executed on every request
export type RequestMiddleware = (request: RequestF) => RequestF;

// describe an environment entry to hold the middlewares configured
export interface MiddlewareStack {
  [middlewareStackEnv]?: {
    stack: RequestMiddleware[];
  };
}

// construct an environment with provided middlewares
export const middlewareStack: (
  stack?: RequestMiddleware[]
) => MiddlewareStack

// represent environment to be used by consumer
export type RequestEnv = Http & HttpDeserializer & MiddlewareStack;

// JSON GET
export function get<E, O>(
  url: string
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// JSON POST
export function post<I, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// DATA GET
export function postData<I extends DataInput, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>> 

// FORM POST
export function postForm<E, O>(
  url: string,
  body: FormData
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// JSON PATCH
export function patch<I, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// DATA PATCH
export function patchData<I extends DataInput, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>> 

// FORM PATCH
export function patchForm<E, O>(
  url: string,
  body: FormData
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// JSON PUT
export function put<I, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// DATA PUT
export function putData<I extends DataInput, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>> 

// FORM PUT
export function putForm<E, O>(
  url: string,
  body: FormData
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

// JSON DELETE
export function del<I, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>>

export function delData<I extends DataInput, E, O>(
  url: string,
  body?: I
): T.Effect<RequestEnv, HttpError<E>, Response<O>>
 
// FORM DELETE
export function delForm<E, O>(
  url: string,
  body: FormData
): T.Effect<RequestEnv, HttpError<E>, Response<O>> 

// Provide headers in child environment
// replace = true will discard any header already in environment
// replace = false (default) will merge the two
export function withHeaders(
  headers: Record<string, string>,
  replace = false
): <R, E, A>(eff: T.Effect<R, E, A>) => T.Effect<R, E, A>

// Provide headers through a middleware
// Request path used to restrict to specific domains/urls
// Replace as per withHeaders
export function withPathHeaders(
  headers: Record<string, string>,
  path: Predicate<string>,
  replace = false
): RequestMiddleware 

// Default json deserializer implementation
export const jsonDeserializer: HttpDeserializer 

// Fold over the request type (useful in middleware dev)
export function foldRequestType<A, B, C>(
  requestType: RequestType,
  onJson: () => A,
  onData: () => B,
  onForm: () => C
): A | B | C 

// Get method as string (useful in middleware dev)
export function getMethodAsString(method: Method): string
```

## Usage

Let's add an implementation, we are gonna use libcurl for this purpose

```typescript
yarn add @matechs/http-client-libcurl
```

We are going to start with a simple get request

```typescript
import { effect as T, exit as E } from "@matechs/effect";
import * as HTTP from "@matechs/http-client";
import * as L from "@matechs/http-client-libcurl";
import { pipe } from "fp-ts/lib/pipeable";

// live environment with libcurl and json deserializer
const envLive = pipe(
  T.noEnv,
  T.mergeEnv(L.libcurl()),
  T.mergeEnv(HTTP.jsonDeserializer)
);

// simple http get
const program: T.Effect<
  HTTP.RequestEnv,
  HTTP.HttpError<unknown>,
  HTTP.Response<unknown>
> = HTTP.get("https://jsonplaceholder.typicode.com/todos/1");

// run the program
T.run(
  T.provideAll(envLive)(program),
  E.fold(
    res => {
      console.log(res);
    },
    err => {
      console.log(err);
    },
    e => console.error(e),
    () => console.error("interrupted")
  )
);
```

Will output something like:

```typescript
{
  status: 200,
  body: {
    _tag: 'Some',
    value: { userId: 1, id: 1, title: 'delectus aut autem', completed: false }
  },
  headers: {
    result: { version: 'HTTP/2', code: 200, reason: '' },
    date: 'Sat, 14 Dec 2019 14:06:53 GMT',
    'content-type': 'application/json; charset=utf-8',
    'content-length': '83',
    'Set-Cookie': [
      '__cfduid=dd36c55568f2aa040768612d145c13f1b1576332413; expires=Mon, 13-Jan-20 14:06:53 GMT; path=/; domain=.typicode.com; HttpOnly'
    ],
    'x-powered-by': 'Express',
    vary: 'Origin, Accept-Encoding',
    'access-control-allow-credentials': 'true',
    'cache-control': 'max-age=14400',
    pragma: 'no-cache',
    expires: '-1',
    'x-content-type-options': 'nosniff',
    etag: 'W/"53-hfEnumeNh6YirfjyjaujcOPPT+s"',
    via: '1.1 vegur',
    'cf-cache-status': 'HIT',
    age: '2833',
    'accept-ranges': 'bytes',
    'expect-ct': 'max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"',
    server: 'cloudflare',
    'cf-ray': '5450bdaff97fe638-LHR'
  }
}
```

Note the request was performed using HTTP/2 and you have control over the full response, let's now do something more interesting:

```typescript
import { effect as T, exit as E } from "@matechs/effect";
import * as HTTP from "@matechs/http-client";
import * as L from "@matechs/http-client-libcurl";
import { pipe } from "fp-ts/lib/pipeable";
import * as A from "fp-ts/lib/Array";

// live environment with libcurl and json deserializer
const envLive = pipe(
  T.noEnv,
  T.mergeEnv(L.libcurl()),
  T.mergeEnv(HTTP.jsonDeserializer)
);

// fetch 10 todos in parallel and map the response
const program: T.Effect<
  HTTP.RequestEnv,
  HTTP.HttpError<unknown> | Error,
  unknown[]
> = pipe(
  // fetch a list of 10 todos in parallel
  A.array.sequence(T.parEffect)(
    pipe(
      A.range(1, 10),
      A.map(n => HTTP.get(`https://jsonplaceholder.typicode.com/todos/${n}`))
    )
  ),
  // extract body from each request, fail if empty
  T.chain(arr =>
    A.array.traverse(T.effect)(arr, r =>
      T.fromOption(() => new Error("empty response"))(r.body)
    )
  )
);

T.run(
  T.provideAll(envLive)(program),
  E.fold(
    res => {
      console.log(res);
    },
    err => {
      console.log(err);
    },
    e => console.error(e),
    () => console.error("interrupted")
  )
);
```

This will print:

```typescript
[
  { userId: 1, id: 1, title: 'delectus aut autem', completed: false },
  {
    userId: 1,
    id: 2,
    title: 'quis ut nam facilis et officia qui',
    completed: false
  },
  { userId: 1, id: 3, title: 'fugiat veniam minus', completed: false },
  { userId: 1, id: 4, title: 'et porro tempora', completed: true },
  {
    userId: 1,
    id: 5,
    title: 'laboriosam mollitia et enim quasi adipisci quia provident illum',
    completed: false
  },
  {
    userId: 1,
    id: 6,
    title: 'qui ullam ratione quibusdam voluptatem quia omnis',
    completed: false
  },
  {
    userId: 1,
    id: 7,
    title: 'illo expedita consequatur quia in',
    completed: false
  },
  {
    userId: 1,
    id: 8,
    title: 'quo adipisci enim quam ut ab',
    completed: true
  },
  {
    userId: 1,
    id: 9,
    title: 'molestiae perspiciatis ipsa',
    completed: false
  },
  {
    userId: 1,
    id: 10,
    title: 'illo est ratione doloremque quia maiores aut',
    completed: true
  }
]
```

Let's suppose [https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com) is a domain where we want to send specific headers, for example an auth token we can easily wire in the middleware enviromnet with only a small addition:

```typescript
// live environment with libcurl and json deserializer and header middleware
const envLive = pipe(
  T.noEnv,
  T.mergeEnv(L.libcurl()),
  T.mergeEnv(HTTP.jsonDeserializer),
  T.mergeEnv(
    HTTP.withPathHeaders({ token: "demo" }, path =>
      path.startsWith("https://jsonplaceholder.typicode.com")
    )
  )
);

```

The rest of the code doesn't change and you are not sending the header!

## Notes

We strongly recommend using [io-ts](https://github.com/gcanti/io-ts) to refine the responses to proper runtime safe types. If you don't need full deserialization you can specify the types on the request function in order to have the response typed.

```typescript
interface Todo {
    userId: number,
    id: number,
    title: string,
    completed: boolean
}

interface TodoError {
    message: string
}

const program: T.Effect<
  HTTP.RequestEnv,
  HTTP.HttpError<TodoError> | Error,
  Todo[]
> = pipe(
  // fetch a list of 10 todos in parallel
  A.array.sequence(T.parEffect)(
    pipe(
      A.range(1, 10),
      A.map(n => HTTP.get<TodoError, Todo>(`https://jsonplaceholder.typicode.com/todos/${n}`))
    )
  ),
  // extract body from each request, fail if empty
  T.chain(arr =>
    A.array.traverse(T.effect)(arr, r =>
      T.fromOption(() => new Error("empty response"))(r.body)
    )
  )
);
```

