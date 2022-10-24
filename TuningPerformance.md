# Tuning Performace

## Introduction

This document is intended to provide a guide to tuning the performance of the

## Tuning

### Tuning Function Memory

The memory used by a function is determined by the number of concurrent invocations of the function. The memory used by a function is the sum of the memory used by the function itself and the memory used by the function's containers. The memory used by a function's containers is the sum of the memory used by the function's containers and the memory used by the function's containers' containers.

install the aws -lambda-power-tuning tool

```zsh
npm install -g lumigo-cli
```

run the tool

```zsh
lumigo-cli powertune-lambda io-bound-example -r us-east-1
```

## Cold Start
- memory allocation does not effect initialization time during cold start
- it is faster to load dependencies fomr a lambda layer than from a package
- unused dependendices do not add to initialization time
- see https://github.com/lumigo/SAR-lambda-Janitor for a lambda layer that removes unused dependencies

### Analysis of cold starts

to get a table of all lambdas in an account and their cold start times
```zsh
lumigo-cli cold-starts -r us-east-1
```

to measure a single lambda's cold start time. notet that this only measures initialization time, not invocation time
```zsh
lumigo-cli measure-lambda-cold-starts -r us-east-1 -n aws-skd-example -i 100
```

to see invocation time, check X-Ray tracing, but understand that the tunable parameters are in initialization time
the time between initialization and invocation is AWS downloading our code and doing setup... not sometihng we have
any control over.

### Tuning Cold Starts

- use a lambda layer to reduce the size of the package
- should you package the sdk in your dependencies?
  - it does make the package smaller, therefore faster
  - it can make for a bad developer experience because what gets packaged may be an older version of the sdk
- import only what you need from the sdk
- the serverless framework has a plugin that can help with this
  - https://www.serverless.com/plugins/serverless-plugin-optimize
  - it does some tree shaking to remove unused dependencies


### provisioned Concurency

- should only be used as a last resort

#### When to use provisioned concurrency

- when you have a function that is invoked infrequently, but when it is invoked it is invoked a lot
- "I can't optimize the cold start time anymore"
- "Cold Starts are stacking up in the service call"

#### When not to use provisioned concurrency
- a better option is to use a http keep alive instead of provisioned concurrency