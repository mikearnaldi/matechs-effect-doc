---
description: 'And write better end to end software, so you can sleep at night!'
---

# How to use Effect!

## Getting Super Powers

You will aquire knowledge on how to build software in a purely functional manner, you will leverage a particular kind of algebraic effects called environmental effects in order to structure your code in a better way enabling you to test each individual component in an isolated way and making your modules reusable and composable.

We are not cheating, neither using magic tricks! By leveraging functional programming principles we have developed a set of libraries that enrich the [fp-ts](https://github.com/gcanti/fp-ts) ecosystem with the functionalities that you may find in a library like [ZIO](https://github.com/zio/zio) in the Scala ecosystem.

The latest development in the core TypeScript language allow us to take environment usage even further compared to what ZIO can achieve, namely TypeScript has a very good support for intersection types and this makes environment usage wonderful.

## Installation

So let's get the superpowers, bootstrap a new `npm` project and install basic dependencies:

```text
$ mkdir my-project
$ yarn init -y
$ yarn install fp-ts fp-ts-contrib @matechs/effect
$ yarn install -D typescript
```

## License

The library is released with an MIT license and the codebase is fully open-source at:   
[https://github.com/mikearnaldi/matechs-effect](https://github.com/mikearnaldi/matechs-effect)  
  
As with any good library there is a commercial project that support the development and maintainance, if you want to know more find us at [https://www.matechs.com/](https://www.matechs.com/) we are a digital accelerator looking for smart founders!

## Project Config

You will need to scaffold a classic `npm` package with TypeScript configured strictly, additionally we advice configuration of jest or another testing framework of your choice: you can find examples at [https://github.com/mikearnaldi/matechs-effect](https://github.com/mikearnaldi/matechs-effect) where you can "copy" both `tsconfig` & `jest` configs.

