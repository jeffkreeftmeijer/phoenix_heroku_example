## Making our Project Heroku-ready

Every new Phoenix project ships with a config file `config/prod.secret.exs` which stores configuration that should not be commited along with our source code. By default Phoenix adds it to our `.gitignore` file.

This works great except Heroku uses [environment variables](https://devcenter.heroku.com/articles/config-vars) to pass sensitive informations to our application. It means we need to make some changes to our config before we can deploy.

First, let's make sure our secret key and port number are loaded from Heroku's environment variables instead of `config/prod.secret.exs` by adding a `secret_key_base` line and updating the port number from `80` to `System.get_env("PORT")` in `config/prod.exs`:

```elixir
config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: System.get_env("PORT")],
  url: [host: "example.com", port: 80],
  cache_static_manifest: "priv/static/manifest.json",
  secret_key_base: System.get_env("SECRET_KEY_BASE")
```

Then, we'll add the production database configuration to `config/prod.exs`:

```elixir
# Configure your database
config :hello_phoenix, HelloPhoenix.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: System.get_env("DATABASE_URL"),
  pool_size: 20
```

Now, let's tell Phoenix to use our Heroku URL and enforce we only use the SSL version of the website. Find the url line:

```elixir
url: [host: "example.com", port: 80],
```

... and replace it with this (don't forget to replace `mysterious-meadow-6277` with your application name):

```elixir
url: [scheme: "https", host: "mysterious-meadow-6277.heroku.com", port: 443],
force_ssl: [rewrite_on: [:x_forwarded_proto]],
```

Since our configuration is now handled using Heroku's environment variables, we don't need to import the `config/prod.secret.exs` file in `/config/prod.exs` any longer, so we can delete the following line:

```elixir
import_config "prod.secret.exs"
```

Our `config/prod.exs` now looks like this:

```elixir
use Mix.Config

...

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: System.get_env("PORT")],
  url: [scheme: "https", host: "mysterious-meadow-6277.heroku.com", port: 443],
  force_ssl: [rewrite_on: [:x_forwarded_proto]],
  cache_static_manifest: "priv/static/manifest.json",
  secret_key_base: System.get_env("SECRET_KEY_BASE")

# Do not print debug messages in production
config :logger, level: :info

# Configure your database
config :hello_phoenix, HelloPhoenix.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: System.get_env("DATABASE_URL"),
  pool_size: 20
``` 
