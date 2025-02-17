---
slug: /zenith/developer/go/191108/advanced-programming
displayed_sidebar: "zenith"
authors: ["Alex Suraci", "Erik Sipsma"]
date: "2023-11-03"
---

# Advanced Programming

{@include: ../../partials/_experimental.md}

## Introduction

Once you've understood the basics of writing your own Dagger modules, you're going to inevitably want to learn more and do more. That's where this guide comes in. It shows you some of the more advanced techniques, tips and tricks you will need to supercharge your Dagger module development.

## Requirements

This guide assumes that:

- You have a good understanding of the Dagger Go SDK. If not, refer to the [Go SDK reference](https://pkg.go.dev/dagger.io/dagger).
- You have the Dagger CLI installed. If not, [install Dagger](../../../current/cli/465058-install.md).
- You have a Dagger module. If not, create a module using the [Go](../../developer/go/525021-quickstart.md) quickstart.
- You have Docker installed and running on the host system. If not, [install Docker](https://docs.docker.com/engine/install/).

## Use modules in other modules

Modules can call each other. To add a dependency to your module, you can use `dagger mod install`, as in the following example:

```sh
dagger mod install github.com/shykes/daggerverse/helloWorld@26f8ed9f1748ec8c9345281add850fd392441990
```

This module will be added to your `dagger.json`:

```json
"dependencies": [
  "github.com/shykes/daggerverse/helloWorld@26f8ed9f1748ec8c9345281add850fd392441990"
]
```

You can also use local modules as dependencies. However, they must be stored in a sub-directory of your module. For example:

```sh
dagger mod install ./path/to/module
```

The dependent module will be added to your code-generation routines, so you can access it from your own module's code, as shown below:

```go
func (m *Potato) HelloWorld(ctx context.Context) (string, error) {
  return dag.HelloWorld().Message(ctx)
}
```

:::tip
Find modules on the [Daggerverse](https://daggerverse.dev).
:::

## Chain modules together

Module functions can return custom objects, which in turn can define new functions. This allows for "chaining" of functions in the same style as the [core Dagger API](https://docs.dagger.io/api/reference).

So long as your object can be JSON-serialized by your SDK, its state will be preserved and passed to the next function in the chain.

Here is an example module using the Go SDK:

```go file=./snippets/advanced-programming/chaining/main.go
```

And here is an example query for this module:

```graphql
{
  helloWorld {
    message
    withName(name: "Monde") {
      withGreeting(greeting: "Bonjour") {
        message
      }
    }
  }
}
```

The result will be:

```json
{
  "helloWorld": {
    "message": "Hello, World!",
    "withName": {
      "withGreeting": {
        "message": "Bonjour, Monde!"
      }
    }
  }
}
```

## Miscellanea

- The context and error return are optional in the module's function signature; remove them if you don't need them.
- A module's private fields will not be persisted.

## Conclusion

This guide showed you a few patterns and techniques you can use to improve your Dagger module programming skills.
