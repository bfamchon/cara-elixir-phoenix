# Au programme !

Après cette magnifique présentation, il est temps pour vous de mettre la main à la pâte !

*Mais comment cuisiner notre projet ?*
- Prenez une petite pincée d'Erlang,
- Quelques cuillères à soupe d'Elixir,
- Mélangez le tout à l'aide du Plug
- Puis assaisonnez votre Phœnix...
- Et hop, 2 heures au four !
- Une vraie recette texane digne d'un Cowboy.

# Arranger le plan de travail

## Erlang & Elixir 

**Si vous travaillez sur le PC de la fac, on vous a déjà installé ce qu'il faut, passez directement à l'étape Phoenix**

- Installer la [machine Erlang](http://erlang.org/doc/index.html)
- Installer le [langage Elixir](https://elixir-lang.org/) en suivant la [procédure pour votre environnement](https://elixir-lang.org/install.html)
- Lancer `mix -v` pour vérifier la bonne installation du langage ! ( 1.5.3 actuellement )

## Phoenix

- Installer [le framework Phoenix](http://phoenixframework.org/), grâce à  la commande `mix local.hex`. 
- [Pour plus d'informations sur mix](http://phoenixframework.org/) cliquez ici !

## On est pas ici pour coller des gommettes... Mais si c'est joli, ça nous va aussi ! 

Pour le confort des yeux:

**Si vous travaillez sur Atom :** On vous conseille le plugin [ElixirSublime](https://github.com/vishnevskiy/ElixirSublime)

**Si vous travaillez sur VS Code :** On vous conseille le plugin [vscode-elixir](https://marketplace.visualstudio.com/items?itemName=mjmcloug.vscode-elixir)

**Si vous travaillez sur Sublime Text :** On vous conseille le plugin [atom-elixir](https://github.com/msaraiva/atom-elixir)

**Sinon....** Bon courage BG ! :fu:

# Préparer les ingrédients

On veut générer un squelette grâce à Phoenix: `mix phx.new chien` ( parce-que j'keaf moins les chat ) 
`phx` est [votre super générateur](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Server.html#content) ! Super utile quand on débute pour se rendre compte de la puissance de l'outil... 
```
	Fetch and install dependencies? [Yn]
```
Répondez Yes* !
**Oui*

``mix ecto.create`` va vous permettre de créer la base, par défaut Postgres [mais configurable](https://github.com/elixir-ecto/ecto), avec les paramètres que vous avez renseigné dans le ``config/*.exs``.

Justement, nous allons utiliser le connecteur sqlite3, plus adapté pour notre usage. Dans le fichier ``mix.exs``, ajoutez la dépendance suivante:
```
  defp deps do
    [
      ...
      {:sqlite_ecto2, "~> 2.1"}
    ]
  end
```
Et lancez un ``mix depts.get`` pour récupérer le connecteur sqlite ! Dernière petite config à changer, dans le fichier ``config/dev.exs``, remplacez la configuration par défaut:

```
# Configure your database
config :chien, Chien.Repo,
  adapter: Sqlite.Ecto2,
  database: "chien_dev"
```

Vous pouvez y allez, lancez la création avec ecto !


```
	The database for Chien.Repo has been created
```
Félicitation, vous pouvez allez en pause ! :tada:
*(Non je déconne, restez encore un peu...)*

Lancez désormais ``mix phx.server``, et rendez-vous à ``localhost:4000`` !

<img src="https://preview.ibb.co/k9cu87/Capture_du_2018_02_13_22_34_24.png" alt="Phoenix starter" border="0">

***Ça y est, tout fonctionne ! [On passe aux choses sérieuses !](https://github.com/unip62/cara-elixir-phoenix/blob/master/PART1.md)***

