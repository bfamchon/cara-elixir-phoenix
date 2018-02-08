:heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart:   
:heart::princess::princess::princess::princess::princess:......................:princess::princess::princess::princess::princess::heart:   
:heart::princess:..................................................:princess:.............................:heart:       
:heart::princess::princess::princess:....................................:princess:.............................:heart:            
:heart::princess:..................................................:princess:.............................:heart:  
:heart::princess:..................................................:princess:.............................:heart:  
:heart::princess:..................................................:princess::princess::princess::princess::princess::heart:   
:heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart::heart:    
# CARA-elixir-phoenix
Découverte du langage Elixir &amp; de son framework web Phoenix !

# Description du projet :

Et pour ce projet, nous allons mettre en place un tchat, pour les plus fainéants d'entre vous, vous pouvez clone [discord](https://github.com/discordapp). Pour les plus courageux, ca se passe plus bas ! :grinning:

Mais tout d'abord voici les quelques users Stories que nous voulons mettre en place
+ TODO
+ TODO
+ TODO
+ TODO

# Mise en place des outils :
Pour commencer le projet, il faudra mettre en place les outils nécéssaires aux développeurs.
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

# OTP, mettre en oeuvre notre Supervision Tree
Rappelez-vous le Supervision Tree permet d'obtenir une application distribué hautement tolérante aux pannes, pour celà nous avons à notre disposition 2 types de process : 
   ###### Les "Supervisor" en charge de gérer des processus fils à travers différentes stratégies.
   - Les "Application" peut être comparé à un main, il sera lancé au démarrage de l'application et initialisera le supervision tree
   - Les "Registry" ajouté récemment, permettent de référencer les différents process.
   - Les "Supervisor" ajouté récemment, permettent de référencer les différents process.
   
   ###### Les "GenServer" des machines permettant de stocker un état et recevoir différents messages.

En plus de ça nous aurons un processus très important, notre "Application" qui jouera le rôle de main.
Il faut savoir que d'autres entités existe [ici](https://hexdocs.pm/elixir/Agent.html#content) rubrique PROCESSES & APPLICATIONS.

Une application distribué est toujours découpé de la façon suivante :
Todo : schéma Application -> repository contenant registry -> supervisor(s) -> GenServer(s)

## Créer les process
N'hésitez pas à lancer votre application et regarder le supervision tree à l'observeur (onglet application):
```bash
#Démarre l'application compilé en mode interactif
iex -S mix 
#Démarre l'observeur permettant de monitorer l'application
:observer.start
```

### Un [genServer](https://hexdocs.pm/elixir/GenServer.html)
Ci-dessous le code nécéssaire à la création d'un genServer, la méthode `start_link` est nécéssaire, les méthodes `handle_call` permettent de recevoir des messages et faire des traitements en conséquence.
```elixir
defmodule MyApp.Worker do
  use GenServer
  
  def start_link(name, options \\ []) do #Méthode lancée au démarrage du processus
    GenServer.start_link(__MODULE__, %__MODULE__{name: name}, options)
  end
  
  #handle_cast -> Synchrone
  def handle_call({:MessageTraitementA, Param1}, _pid, state) do #Méthode lancée lors de la réception d'un message avec comme identifiant :MessageTraitementA
    #Code métier
  end
  #handle_cast -> Asynchrone
  def handle_cast({:MessageTraitementB, Param1 , Param2}, _pid, state) do 
    #Code métier
  end
```
Il est d'usage de faciliter l'utilisation de nos workers en y ajoutant des fonctions permettant de générer les messages attrapés par les méthodes `handle_call` ainsi :
```elixir
  def traitementA(pid, params) do
  #Appel synchrone
    GenServer.call(pid, {:MessageTraitementA, param1})
  end

  def traitementB(pid, task_id) do
  #Appel asynchrone
    GenServer.cast(pid, {:MessageTraitementB, param1,param2})
  end
```
### Un [supervisor](https://hexdocs.pm/elixir/Supervisor.html#content)

```elixir
defmodule MyApp.Supervisor do
  use Supervisor

  def start_child(pid, genServer_name, name_from_registry) do
    Supervisor.start_child(pid, [genServer_name, [name: name_from_registry]])
  end

  def init(_) do
    children = [
      worker(MyApp.Worker, [], restart: :transient) #Stratégie de redémarrage ici transient
    ]

    supervise(children, strategy: :one_for_one) #Stratégie globale
  end

  def start_link do
    Supervisor.start_link(__MODULE__, [])
  end
end
```
##### :princess: Le point culture : :princess:   
Stratégies globale :
+ :one_for_one -> Si 1 processus meurt, on le relance
+ :one_for_all -> Si 1 processus meurt, on relance tout les processus
+ :rest_for_one -> Si 1 processus meurt, on relance tout les processus lancées après celui qui est mort (inclus)

Stratégies restart : 
+ :permanent (default) -> On relance tout le temps le processus
+ :temporary -> On ne relance jamais le processus
+ :transient -> On relance uniquement si le processus meurt en erreur.

Stratégies shutdown :
+ :brutal_kill -> On kill le processus.
+ X(integer) > 0 -> On kill le processus au bout de X seconds.
+ :infinity -> On ne kill pas le processus, il se chargera de se kill.

+ La première variable représente le message reçu par le genServer, à l'aide du pattern-matching il associera l'atom reçu à la méthode souhaitée.
+ La variable _pid représente simplement le pid du processus ayant envoyé le message.
+ La variable State représente l'etat de l'objet qui est stocké dans le genServer (n'oubliez pas, le genServeur est une machine à état pouvant recevoir des messages)

### Une [application](https://hexdocs.pm/elixir/Application.html#content)
Première étape, ajouter l'application permettant de lancer le serveur.

```elixir
defmodule MyApp.Application do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      supervisor(MyApp.Supervisor, [])
    ]

    opts = [strategy: :one_for_one]
    Supervisor.start_link(children, opts)
  end
end
```

Dernière étape (et oui c'était rapide) définir l'application à lancer au démarrage dans le fichier mix.exs correspondant, pour celà remplacer l'application existant par :
```elixir
  def application do
    [
      extra_applications: [:logger],
      #On définit ici l'application à lancer
      mod: {MyApp.Application, []} 
    ]
  end
```
### Ajouter un [registry](https://hexdocs.pm/elixir/Registry.html#content) pour pouvoir définir plus simplement le nom de vos process.
Les registry sont des supervisors spéciaux entièrement gérés par elixir, il est très simple d'en initialiser un. Votre registry sera l'enfant de l'un de vos supervisor, une bonne pratique et de le laisser s'occuper des process fils de son père, lorsque c'est utile.

Vous pouvez l'interposer entre votre Application et votre supervisor (sinon :hear_no_evil:) :
```elixir
defmodule MyApp.Registry_supervisor do
  use Supervisor
  @registry :my_registry    #Ici on déclare le nom du registry

  def init(_) do
    children = [
      supervisor(Registry, [[keys: :unique, name: @registry]]), # notre supervisor sera de type Registry, la clef qu'il associera sera unique et il s'appelera "my_registry"
      supervisor(MyApp.Another_supervisor, [], id: :supervisor) # d'autres fils
    ]
    supervise(children, strategy: :one_for_all) #On définit la stratégie one_for_all car si un tombe, le référencement sera inexact !
  end
  ```

C'est beau de faire un registry, mais il faut pouvoir l'utiliser ! Alors comment fait-on pour savoir si un processus existe ? Et si il n'existe pas, il faut le créer non ?
Pour celà nous allons ajouter les méthodes suivante à notre superviseur

```elixir
  def find_or_create(name) do
    case Registry.lookup(@registry, name) do
      [] ->       #Dans le cas ou le processus n'existe pas
        __MODULE__ #TODO Verif ici avec baptiste
        |> find_supervisor_pid #On cherche le supervisor qui s'occupera du nouveau genServer
        |> MyApp.Another_supervisor.start_child(children_name, {:via, Registry, {@registry, children_name}}) #On demande à notre superviseur fils de démarrer un nouveau genServer et on lui passe le registry où le nouveau processus devra s'enregistrer
      [{pid, _}] ->
        {:ok, pid} #Sinon on renvoi le PID (l'identité) du processus trouvé
    end
  end

  defp find_supervisor_pid(repository_pid) do
    Supervisor.which_children(repository_pid)
    |> Enum.find_value(fn
      {:supervisor, supervisor_pid, _, _} -> supervisor_pid
      _ -> nil
    end)
  end
  ```
  Félicitation, vous devriez avoir un résultat semblable à celui-ci :
  TODO
  Si ce n'est pas le cas appelez-nous :speech_balloon:
  
  # Mon premier "Tchat"

  # Mise en place de l'api
  
  # Et enfin l'interface
  
# Félicitation
:pray: Il n'y a plus qu'à mettre un très bon avis dans le mail que vous allez recevoir :pray:  
On espère que ça vous a plu :heart:  

Baptiste Famchon et Laurent Canis
