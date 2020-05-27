![CI](https://github.com/brainn-co/xcribe/workflows/CI/badge.svg?branch=master)
![Hex.pm](https://img.shields.io/hexpm/v/xcribe?style=flat-square)
![Hex.pm](https://img.shields.io/hexpm/l/xcribe?style=flat-square)
![Hex.pm](https://img.shields.io/hexpm/dt/xcribe?style=flat-square)

# Xcribe

Xcribe is a doc generator for Rest APIs built with Phoenix.
The documentation is generated by the test specs.

Currently both Blueprint and Swagger (OpenAPI Spec 3.0) syntax are supported. In the future other formats will
be added.

## Installation

Add Xcribe to your application

mix.exs

```elixir
def deps do
  [
    {:xcribe, "~> 0.7.0"}
  ]
end
```

Update deps

```sh
mix deps.get
```

Xcribe requires that you create an "Information Module". This module defines
the custom information about your API documentation.

Create a module that uses `Xcribe.Information` as:

```elixir
defmodule YourAppWeb.Support.DocInformation do
  use Xcribe.Information

  xcribe_info do
    name "Your awesome API"
    description "The best API in the world"
    host "http://your-api.us"
  end
end
```

Add a new configuration with the created module

config/test.exs

```elixir
  config :xcribe,
    information_source: YourAppWeb.Support.DocInformation,
    format: :swagger,
    output: "app_doc.json"
```

Next, in your `test/test_helper.exs` you should configure ExUnit to use Xcribe
formatter. You would probably like to keep the default ExUnit.CLIFormatter as
well.

test/test_helper.exs

```elixir
 ExUnit.start(formatters: [ExUnit.CLIFormatter, Xcribe.Formatter])
```

## Usage

Xcribe generates documentation from `Plug.Conn` struct used on your tests. To
document a route you need pass the `conn` struct to macro `document`.

First you need import `Xcribe.Document` macro in your `ConnCase` file (usally located in `/test/support/conn_case.ex`).
Doing that, `document/2` macro will be available into your test specs.

/test/support/conn_case.ex

```elixir
defmodule YourAppWeb.ConnCase do
  use ExUnit.CaseTemplate

  using do
    quote do

     ...

      # import Xcribe `document/2` macro
      import Xcribe.Document

     ...

    end
  end
end
```

Now in your tests you should call the macro `document` with the conn struct

```elixir
test "get posts", %{conn: conn} do

  response =
    conn
    |> get("/posts")
    |> document()
    |> json_response(200)

  assert response == []
end
```

you can specify a custom request name by passing an argument `as: "description"`
to the macro

```elixir
test "get posts", %{conn: conn} do

  response =
    conn
    |> get("/posts")
    |> document(as: "index request")
    |> json_response(200)

  assert response == []
end
```

To generate the doc file run `mix test` with an env var `XCRIBE_ENV=true`

```sh
XCRIBE_ENV=true mix test
```

A file `api_doc.apib` will be created with the documentation if ApiBlueprint is configured. If Swagger format is chosen, then `openapi.json` file will be created.

### Rendering HTML

The output file is write in Blueprint sintax. To render to HTML you can use the
tools listed at [APIBLUEPRINT.ORG](https://apiblueprint.org/tools.html#renderers)

If Swagger format is configured, [Swagger UI](https://swagger.io/tools/swagger-ui/download/) can be used to display Swagger documentation.

## Configure

You can add this configurations to your `config/test.ex`

- information_source: the module with doc information
- output: a custom name to the output file
- format: ApiBlueprint or Swagger formats
- env_var: a custom name to the env to active Xcribe.Formatter
- json_library: The library to be used for json decode/encode. See `Xcribe.JSON`

Example

```elixir
config :xcribe, [
  information_source: YourAppWeb.Information,
  output: "API-DOCUMENTATION.apib",
  format: :api_blueprint # or :swagger,
  env_var: "DOC_API",
  json_library: Jason
]
```

## Contributing

[Contributing Guide](CONTRIBUTING.md)

## License

[Apache License, Version 2.0](LICENSE) © [brainn.co](https://github.com/brainn-co)
