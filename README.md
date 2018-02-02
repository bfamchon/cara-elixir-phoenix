# CARA-elixir-phoenix
Découverte du langage Elixir &amp; de son framework web Phoenix !

# Description du projet

# Mise en place des outils :

## Mettre en place l'environnement de travail
### Intaller Elixir & Phoenix
Nous allons commencer par installer la [machine Erlang](http://erlang.org/doc/index.html) et le langage [Elixir](https://elixir-lang.org/), référez vous [ici](https://elixir-lang.org/install.html). Dès l'installation terminé, lancer la commande mix -v afin de vous assurer du bon fonctionnement de celle-ci. Vous devriez voir deux choses, la version de la machine Erlang et la version d'elixir (à l'heure où j'écris ces lignes 1.5.3).

Ensuite, il vous faudra installer [Phoenix](http://phoenixframework.org/), pour cela nous allons utiliser l'outils `mix` que nous venons juste d'installer. Jouez la commande `mix local.hex`. Pour plus d'informations sur mix rendez-vous sur la [doc](http://phoenixframework.org/)

### Mettre en place l'IDE
On cherche à travailler dans un minimum de confort, de la coloration syntaxique, de l'auto-complétion, une gestion d'erreur.
#### Si vous travaillez sur Atom :
   On vous conseille le plugin [ElixirSublime](https://github.com/vishnevskiy/ElixirSublime)
#### Si vous travaillez sur VS Code :
   On vous conseille le plugin [vscode-elixir](https://marketplace.visualstudio.com/items?itemName=mjmcloug.vscode-elixir)
#### Si vous travaillez sur Sublime Text :
   On vous conseille le plugin [atom-elixir](https://github.com/msaraiva/atom-elixir)
#### Sinon :
Bon courage beau goss :fu:

# Mise en place du projet :
Nous voilà dans la première étape du TP. Générer notre projet Elixir, nous cherchons à atteindre le résultat suivant :
<img src="https://image.ibb.co/nmovKR/Elixir_Project.png" alt="Elixir_Project" border="0">

Rien de plus simple, comme nous vous avons présenté à l'oral, nous avons à disposition les commandes :
+ mix new [AppName]
+ mix new [AppName] --umbrella
+ mix phx.new [AppName]
+ mix phx.new [AppName] --umbrella

Pour rappel, seule le projet web a besoin de phoenix !

A vous de jouer :kissing_heart:

