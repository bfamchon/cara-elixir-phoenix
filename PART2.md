## 2ème étape: Et si on gérait le multi-user ?

On va tout d'abord générer le [module Presence](https://hexdocs.pm/phoenix/Phoenix.Presence.html) de Phoenix, qui va nous permettre de générer un superviseur chargé d'observer & gérer les événements sur nos channels !
`` mix phx.gen.presence `` nous indique alors d'ajouter ce nouveaux membre au supervision-tree !
Allez, vous pouvez le faire, l'étape est indiquée dans le terminal !

On cherche maintenant à créer un [Channel](https://hexdocs.pm/phoenix/channels.html) sur notre application, afin de gérer le transport des actions utilisateur: envoi de message, connexion... ! Une connexion ouverte entre notre application et le client !

Et là vous dites: 

"Mais bien sûr, c'est ce qu'ils ont montré pendant le Live-Coding ! Allons donc décommenter la ligne de création du channel dans ``channels/user_socket.ex`` !"

Vous êtes vraiment beaucoup trop bons !

On va maintenant modifier notre fonction ``connect``, histoire qu'elle corresponde un peu plus à notre sujet !
```elixir
  def connect(%{"user" => user}, socket) do
    {:ok, assign(socket, :user, user)}
  end
```

On lui donne un premier paramètre la  socket, les deux autres paramètres seront liée pour former une map dont la clef est ``"user"``, et la valeur ``user`` !  (Vous remarquez l'utilisation d'un atom, ces constantes dont la valeur est égal à leurs noms)
Cette fonction assigne donc au socket un nouveau couple clef valeur, notre utilisateur !

Nous avons dé-commenté une ligne supposant l'existence d'un certain RoomChannel... Il faut donc créer le fichier ``room_channel.ex`` dans le même dossier, en voilà la structure:
```elixir
	defmodule ChienWeb.RoomChannel do
	    use ChienWeb, :channel
	    alias ChienWeb.Presence
		
	end
```

Rien de sorcier, on spécifie la création d'un channel grâce à l'atom, et on alias Presence pour l'utiliser dans le scope du module !

Alias ? Mais c'est quoi ça ? Eh bien c'est simple le mot clef alias permet de raccourcir les appels à un module donnée avec un nom.
En utilisant ``alias ChatterWeb.Presence, as:pre`` vous pouvez appeler le module ChatterWeb.Presence avec pre. Ici nous n'avons pas définit de "as" l'alias par défaut se fait sur la deuxième partie de l'élément donc Presence.

Ajoutez y la fonction suivante:
```elixir
    def join("room:lobby", _, socket) do
      send self(), :after_join
      {:ok, socket}
    end
```
Ici, on définit le comportement à l'arrivée d'un utilisateur dans le channel ! Dans notre cas, on exécute la callback correspondant à l'atom ``:after_join``.
```elixir
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

```javascript
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

```javascript
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

Ici, nous récupérons la liste d'id ``UserList`` pour la compléter avec notre tableau de présences ! Ce tableau de présences, on l'a formaté un peu pour afficher quelque chose de plus lisible qu'un Timestamp venant du serveur... 

Les deux événements sur la room permettent de gérer les changements dans notre liste de présence à notre arrivée dans le salon ( ``presence_state``), mais aussi en temps réel ( ``presence_diff``) !

Actualisez votre page et ouvrez plusieurs onglets... :tada:

<img src="https://preview.ibb.co/kQTwO7/Capture_du_2018_02_15_13_54_28.png" alt="Phoenix starter" border="0">


***C'est bien tout ça mais... [Quand est-ce qu'on parle ?](https://github.com/unip62/cara-elixir-phoenix/blob/master/PART3.md)***
