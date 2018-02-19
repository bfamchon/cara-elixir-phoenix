# In the program !

After this wonderful presentation, it's time for you to get your hands dirty!

* But how to cook our project? *
- Take a small pinch of Erlang,
- A few tablespoons of Elixir,
- Mix everything with the plug
- Then season your Phoenix ...
- And presto, 2 hours in the oven!
- A real Texas recipe worthy of a Cowboy.

# Arrange the work plan

## Erlang & Elixir

** If you are working on the college PC, we have already installed what you need, go directly to the Phoenix stage **

- Install the [Erlang machine] (http://erlang.org/doc/index.html)
- Install the [Elixir language] (https://elixir-lang.org/) by following the [procedure for your environment] (https://elixir-lang.org/install.html)
- Run `mix -v` to check the correct installation of the language! (1.5.3 currently)

## Phoenix

- Install [the Phoenix framework] (http://phoenixframework.org/), using the `mix local.hex` command.
- [For more information on mix] (http://phoenixframework.org/) click here!

## We're not here to stick stickers ... But if it's nice, it's ok too!

For the comfort of the eyes:

** If you work on Atom: ** We recommend the [ElixirSublime] plugin (https://github.com/vishnevskiy/ElixirSublime)

** If you work on VS Code: ** We recommend the [vscode-elixir] plugin (https://marketplace.visualstudio.com/items?itemName=mjmcloug.vscode-elixir)

** If you are working on Sublime Text: ** We recommend the [atom-elixir] plugin (https://github.com/msaraiva/atom-elixir)

** Otherwise .... ** Good luck BG! : Fu:

# Prepare the ingredients

We want to generate a skeleton thanks to Phoenix: `mix phx.new chien` (because I keaf less cat)
`phx` is [your super generator] (https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Server.html#content)! Super useful when you start to realize the power of the tool ...
`` `
Fetch and install dependencies? [YT]
`` `
Answer Yes *!
**Yes*

`` mix ecto.create`` will allow you to create the base, default Postgres [but configurable] (https://github.com/elixir-ecto/ecto), with the parameters you entered in `` config / *. exs``.

Precisely, we will use the connector sqlite3, more adapted for our use. In the `` mix.exs`` file, add the following dependency:
`` `
  defps deps do
    [
      ...
      {: sqlite_ecto2, "~> 2.1"}
    ]
  end
`` `
And run a `` mix depts.get`` to get the sqlite connector! Last little config to change, in the `` config / dev.exs`` file, replace the default configuration:

`` `
# Configure your database
config: dog, dog.Repo,
  adapt: ​​Sqlite.Ecto2,
  database: "dog_dev"
`` `

You can go there, launch the creation with ecto!


`` `
The database for Chien.Repo has been created
`` `
Congratulations, you can go on break! : Tada:
* (No I'm kidding, stay a little longer ...) *

Now run `` mix phx.server``, and go to `` localhost: 4000``!

<img src = "https://preview.ibb.co/k9cu87/Capture_of_2018_02_13_22_34_24.png" alt = "Phoenix starter" border = "0">

*** That's it, everything works! [We'll get down to business!] (Https://github.com/unip62/cara-elixir-phoenix/blob/master/PART1.md) ***
