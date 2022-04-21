# Serving generated doc

You can use `Xcribe` to serve your API documentation. Currently we support serve
`Swagger` format. To render documentation we use [Swagger UI](https://swagger.io/tools/swagger-ui/).

## Configuration

For serving with `Xcribe` you must configure doc format as `:swagger` the output path
must be `priv/static` and you must enable `serve` config.

```
      config :xcribe,
        information_source: YourApp.YouModuleInformation,
        format: :swagger,
        output: "priv/static/my_doc.json",
        serve: true

```

## Routing

Add a doc scope to your router, and forward all requests to `Xcribe.Web.Plug`

```
      scope "doc/swagger" do
        forward "/", Xcribe.Web.Plug
      end

```

### Running behind a reverse proxy

Since the static asset urls are based on the request's `port`, `host` and `scheme` the generated urls
when behind a reverse proxy may not be accurate. To get around this, we need to rewrite these values.
Use [Plug.RewriteOn](https://hexdocs.pm/plug/Plug.RewriteOn.html) to rewrite the values based on the 
`X-Forwarded-*` headers. This requires having your proxy set these headers appropriately.
E.g. for Nginx, within the location block:

```
  location {
    proxy_pass http://localhost:4000;
    proxy_set_header Host $host; # to set the host to whatever it's being served as externally
    proxy_set_header X-Forwarded-Proto $scheme; # when serving via https
  }
```

Then within your Phoenix router, add a pipeline that rewrites the values and use the pipleine within
your scope.

```
pipeline :api_doc do
   plug Plug.RewriteOn, [:x_forwarded_host, :x_forwarded_port, :x_forwarded_proto]
end
```

...

``` 
scope "/docs" do
  pipe_through [:api_doc]
end
```
