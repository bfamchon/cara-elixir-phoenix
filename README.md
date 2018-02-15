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
`phx` est [votre super générateur](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Server.html#content) ! Super utile quand on débute pour se rendre compte de la puissance de l'outil... Mais déconseillé pour les utilisations avancées !
```
	Fetch and install dependencies? [Yn]
```
Répondez Yes* !
**Oui*

Laissez vous guider par le terminal !

``mix ecto.create`` va vous permettre de créer la base, par défaut Postgres [mais configurable pour d'autres langages](https://github.com/elixir-ecto/ecto), avec les paramètres que vous avez renseigné dans le ``config/*.exs``.

```
	The database for Chien.Repo has been created
```
Félicitation, vous pouvez allez en pause ! :tada:
*(Non je déconne, restez encore un peu...)*

Lancez désormais ``mix phx.server``, et rendez-vous à ``localhost:4000`` !

<img src="https://preview.ibb.co/k9cu87/Capture_du_2018_02_13_22_34_24.png" alt="Phoenix starter" border="0">

***Ça y est, tout fonctionne ! On passe aux choses sérieuses !***

# La recette !
## 1ère étape: Hello benjam' !

La majorité du projet web phoenix va se trouver dans chien_web.
On va alors modifier le fichier `templates/page/index.html.eex` de manière à préparer notre interface !
```
	<div class="row">
	  <div class="col-md-12 alert alert-info">
	    Hello, <span id="User">
	    <%# HERE %>
	    </span>!
	  </div>
	  <div class="col-md-8">
	    <h2>Messages:</h2>
	    <ul id="MessageList" class="list-unstyled" style="height: 400px; border: 1px solid black; overflow-y: auto; padding: 10px;" ></ul>
	    <input type="text" id="NewMessage" class="form-control">
	  </div>
	  <div  class="col-md-4">
	    <h2>Who’s Online</h2>
	    <ul id="UserList" class="list-unstyled">
	      <li>Loading online users...</li>
	    </ul>
	  </div>
	</div>
```
Une fois le serveur lancé, vous n'avez plus qu'à rafraichir la page pour que les modifications soient prises en compte !

Vous voyez cette balise ``<%# HERE %>`` ? Le ``#`` est un commentaire ! Ce serait sympa de pouvoir afficher le nom de notre utilisateur non ?

L'avantage dans Phoenix, c'est qu'il est plutôt simple de manipuler les paramètres http ! Let's go, on veut pouvoir taper ``localhost:4000?user=Benjam'``

On va donc ajouter ``<% @conn.params["user"] %>``, qui se chargera, dans la requête HTTP, de récupérer le paramètre  ``user`` & de l'afficher ! 

## 2ème étape: Et si on gérait de multiples utilisateurs ?

On va tout d'abord générer le [module Presence](https://hexdocs.pm/phoenix/Phoenix.Presence.html) de Phoenix, qui va nous permettre de générer un superviseur chargé d'observer & gérer les événements sur nos channels !
`` mix phx.gen.presence `` nous indique alors d'ajouter ce nouveaux membre au supervision-tree !
Allez, vous pouvez le faire, l'étape est indiquée dans le terminal !

On cherche maintenant à créer un [Channel](https://hexdocs.pm/phoenix/channels.html) sur notre application, afin de gérer le transport des actions utilisateur: envoi de message, connexion... ! Une connexion ouverte entre notre application et le client !

Et là vous dites: 

"Mais bien sûr, c'est ce qu'ils ont montré pendant le Live-Coding ! Allons donc décommenter la ligne de création du channel dans ``channels/user_socket.ex`` !"

Vous êtes vraiment beaucoup trop bons !

On va maintenant modifier notre fonction ``connect``, histoire qu'elle corresponde un peu plus à notre sujet !
```
  def connect(%{"user" => user}, socket) do
    {:ok, assign(socket, :user, user)}
  end
```

On lui donne en paramètre un socket ainsi qu'une map, dont la clef est ``"user"``, et la valeur ``user``  !
Cette fonction assigne donc au socket un nouveau couple clef valeur, notre utilisateur !

Nous avons dé-commenté une ligne supposant l'existence d'un certain RoomChannel... Il faut donc créer le fichier ``room_channel.ex`` dans le même dossier, en voilà la structure:
```
	defmodule ChatterWeb.RoomChannel do
	    use ChatterWeb, :channel
	    alias ChatterWeb.Presence
		
	end
```

Rien de sorcier, on spécifie la création d'un channel grâce à l'atom, et on alias Presence pour l'utiliser dans le scope du module !

Ajoutez y la fonction suivante:
```
    def join("room:lobby", _, socket) do
      send self(), :after_join
      {:ok, socket}
    end
```
Ici, on définit le comportement à l'arrivée d'un utilisateur dans le channel ! Dans notre cas, on exécute la callback correspondant à l'atom ``:after_join``.
```
    def handle_info(:after_join, socket) do
      Presence.track(socket, socket.assigns.user, %{
        online_at: :os.system_time(:milli_seconds)
      })
      push socket, "presence_state", Presence.list(socket)
      {:noreply, socket}
    end
```

Voilà cette fameuse callback qui ne fait rien de bien sorcier !

On ajoute simplement, pour nouvel utilisateur de ``socket.assigns.user``, une map contenant un timestamp ``online_at``.

On push tout ça dans la socket, et on met à jour notre état des présences !

Ajoutons maintenant un peu de traitement JS histoire de gérer & formater les informations de notre socket !

Remplacez le contenu du fichier ``/assets/js/app.js`` par 

```
import 'phoenix_html';
import { Presence, Socket } from 'phoenix';

let user = document.getElementById('User').innerText;
let socket = new Socket('/socket', { params: { user: user } });
socket.connect();

let presences = {};
```

À la connexion, on récupère notre utilisateur dans l'élément d'id ``User``, et on l'envoie dans les paramètres de ``/socket``. 
Elixir met automatiquement les sockets sur ``/socket``, vous pouvez observer ce comportement dans ``lib/chien_web/endpoint.ex``.

Rajoutez au fichier ces quelques lignes:

```
let formatedTimestamp = Ts => {
  let date = new Date(Ts);
  return date.toLocaleString();
};

let listBy = (user, { metas: metas }) => {
  return {
    user: user,
    onlineAt: formatedTimestamp(metas[0].online_at),
  };
};

let userList = document.getElementById('UserList');
let render = presences => {
  userList.innerHTML = Presence.list(presences, listBy)
    .map(
      presence => `
            <li>
                ${presence.user}
                <br>
                <small>online since ${presence.onlineAt}</small>
            </li>
        `
    )
    .join('');
};

let room = socket.channel('room:lobby');
// Handle two events from our channel,
// the first is about the server which send informations
// on users who are dis/connected @ the first time we come
room.on('presence_state', state => {
  presences = Presence.syncState(presences, state);
  render(presences);
});

// the second is when we're on the application and new users are dis/connecting, we sync the diff
room.on('presence_diff', diff => {
  presences = Presence.syncDiff(presences, diff);
  render(presences);
});

room.join();
```

Ici, nous récupérons la liste d'id ``UserList`` pour la compléter avec notre tableau de présences ! Ce tableau de présences, on l'a formaté un peu pour afficher quelque chose de plus lisible q'un Timestamp venant du serveur... 

Les deux événements sur la room permettent de gérer les changements dans notre liste de présence à notre arrivée dans le salon ( ``presence_state``), mais aussi en temps réel ( ``presence_diff``) !

Actualisez votre page et ouvrez plusieurs onglets... :tada:

<img src="https://preview.ibb.co/kQTwO7/Capture_du_2018_02_15_13_54_28.png" alt="Phoenix starter" border="0">

## 3ème étape: C'est bien beau tout ça... Mais on parle quand ?

Minute on y vient ! Mais d'abord, il va falloir ajouter quelques fonctionnalités à notre ``room_channel.ex``... Comme la réception de nouveaux messages !

Ajoutez y la nouvelle fonction:
```
	def handle_in("message:new", message, socket) do
      broadcast! socket, "message:new", %{
        user: socket.assigns.user,
        body: message,
        timestamp: :os.system_time(:milli_seconds)
      }
      {:noreply, socket}
    end
```
À la réception d'un nouveau message, on va transmettre l'événement à tous nos utilisateurs en précisant l'expéditeur, le message et l'heure d'émission !
On précise à la fin que la réponse des récepteurs ne nous intéresse pas !
