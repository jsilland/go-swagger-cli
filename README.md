# Go Swagger CLI

This repository contains templates and configuration to generate a CLI client from a Swagger definition. It's an extension of [Go Swagger](https://github.com/go-swagger/go-swagger).

**Disclaimer**: this project is an entirely incomplete proof of concept. It includes basic support for a limited set of parameter types (strings, booleans, some numbers) and response formats. API responses are simply pretty-printed out to the standard output.

## Generating a client

First [install](https://goswagger.io/install.html) the Go Swagger tooling, then run the following:

```sh
$ swagger generate client \
    --spec <path to swagger.json spec> \
    --target= <the directory where to generate the client> \
    --template-dir <path to go-swagger-cli>/templates \
    --allow-template-override \
    --config-file <path to go-swagger-cli>/config.yml
```

## Generated code structure

The code for the CLI commands will be generated alongside the client files in the client package, in separate files suffixed with `_cmd.go`.

The generated CLI code depends on [Cobra](https://github.com/spf13/cobra), which therefore needs to be installed as a dependency. Cobra [sub-commands](https://godoc.org/github.com/spf13/cobra#Command.AddCommand) are used to structure the code and commands 

There are three levels of commands:

1. The root command, which will be named after the API
2. The root command has one sub-command per operation [group](https://swagger.io/docs/specification/grouping-operations-with-tags/)
3. Each operation group has one sub-command per operation. For each operation command, parameters will be declared as CLI flags, e.g. `--quxId`.

Given the following sample spec:

```json
{
  "info": {
    "title": "ACME",
  },
  "paths": {
    "/foo/bar": {
      "post": {"tags": ["Foo"], "operationId": "PostBar"}
    },
    "/foo/qux": {
      "get": {"tags": ["Foo"], "operationId": "GetQux"}
    },
    "/baz": {
      "get": {"tags": ["Baz"], "operationId": "GetBaz"}
    },
  }
}
```

The following code will be generated:

```
- client/acme_cmd.go
  - NewCommand(client *Acme) *cobra.Command
  - NewCommandWithDefaultClient() *cobra.Command // This will use the client.Default instance
- client/foo/foo_cmd.go
  - Command(context.Context, foo.ClientService) *cobra.Command
    - postBar(context.Context, foo.ClientService) *cobra.Command
    - getQux(context.Context, foo.ClientService) *cobra.Command
- client/baz/baz_cmd.go
  - Command(context.Context, baz.ClientService) *cobra.Command
    - getBaz(context.Context, baz.ClientService) *cobra.Command
```

## Usage

Once the code is properly generated, you can then use the root command in a main file:

```go
package main

import (
    "fmt"
    "os"
    
    // + import client package here
)

func main() {
    command := client.NewCommandWithDefaultClient()
    if _, err := command.ExecuteC(); err != nil {
		fmt.Errorf(err.Error())
		os.Exit(-1)
	}
}
```

Then compile and run the code:

```sh
$ go build -o acme main.go
$ ./acme foo getQux
```
