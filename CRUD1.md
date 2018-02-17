
## Partons à la découverte du CRUD sur les utilisateurs !

Il faudrait peut-être commencer par créer notre premier schema:

```
  mix phx.gen.schema User users email:unique password:string
```

``User`` sera le nom du module, ``users`` pour le nom de notre table et nous précisons quelques champs: email & password...

Pour plus d'informations sur les possibilité de ``phx.gen.schema``, [cliquez ici](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Schema.html) !

Un petit tour dans le fichier ``priv/repo/migrations/*.exs`` généré pour comprendre qu'à la prochaine commande ``mix ecto.migrate``, les changements décrit seront repercuté en base ! Dans notre cas, il s'agit d'une creation avec deux champs, du timestamp et un index !

Côté ``lib/chien/user.ex``, on observe le schema User mais surtout la [fonction changeset](https://hexdocs.pm/ecto/Ecto.Changeset.html) qui définit ici vos contraintes ( unicité, max, min... )

Lancez la commande ecto pour migrer vos changements en base, et on part insérer notre premier utilisateur à l'aide du mix interactif !

Comme présenté dans le talk introductif, la commande ``iex -S mix phx.server`` va lancer un shell interractif !

Commençons par définir quelques alias afin d'y accéder plus facilement dans nos commandes:
``alias Chien.Repo`` & ``alias Chien.User``

Essayons de récupérer l'ensemble de nos utilisateurs grâce à: ``Repo.all(User)``... Ben ouais BG, c'est un tableau vide, essaye d'insérer la structure User ci-dessous & rejoue la commande !

``Repo.insert(%User{email: "denis.brogniart@kohlant.ah", password: "cabane"})``

***Maintenant, ce serait bien d'avoir un contrôleur pour servir d'intermédiaire entre les différentes briques, non ?***

Allons y ! On créer le fichier ``lib/chien_web/controler/user_controller.ex`` :computer:

``
defmodule ChienWeb.UserController do
    use ChienrWeb, :controller
    alias Chien.User
    alias Chien.Repo
end
``

Puis celui correspondant à notre vue: 
// TODO pas trop compris à quoi il servait...

``
defmodule ChienWeb.UserView do
    use ChienWeb, :view
end
``

Dans ce controller, ajoutons le point d'entrée principal qui listera tous les utilisateurs:

``
    def index(conn, _params) do
      users = Repo.all(User)
      render(conn, "index.html", users: users)
    end
``

Nous n'avons pas encore routé ce chemin... Allons faire un tour dans le fichier ``lib/chien_web/router.ex`` pour ajouter notre ressource supplémentaire:

``
  scope "/", ChienWeb do
    ...
    resources "/users", UserController
    ...
  end
``

Nous avons été assez impréssioné par le fait qu'une simple ligne resources puisse implicitement déclaré une multitude d'endpoints pour notre router: ``mix phx.routes`` pour y visualiser !

Actuellement si vous lancez un ``localhost:4000/users``, Phoenix vous dira qu'il manque un index.html pour l'UserView... Allons créer le fichier ``lib/chien_web/templates/user/index.html.eex`` !

À remplir avec:
```
<h2>List of users</h2>

<table class="table">
  <thead>
    <tr>
      <th>Email</th>

      <th></th>
    </tr>
  </thead>
  <tbody>
    <%= for user <- @users do %>
      <tr>
        <td><%= user.email %></td>
        <td class="text-right">
          <%= link "Profile", to: user_path(@conn, :show, user), class: "btn btn-default btn-xs" %>  
          <%= link "Edit User", to: user_path(@conn, :edit, user), class: "btn btn-default btn-xs" %>
          <%= link "Delete User", to: user_path(@conn, :delete, user), method: :delete, data: [confirm: "Are you sure?"], class: "btn btn-danger btn-xs" %>   
        </td>
      </tr>

    <% end %>
  </tbody>
</table>


<%= link "New user", to: user_path(@conn, :new) %>
```

On vous donne en avance les fonctionnalités à implémenter par rapport aux balises ``<%= link %>``. Rien de sorcier là non plus, on boucle sur les utilisateurs de notre ``@conn`` & on met à dispo 4 liens qui vont diriger vers le ``user_path`` (/users), en y ajoutant les spécificités pour la navigation:
- :show (/users/:id),
- :edit (/users/:id/edit),
- :new (/users/new)
- :delete qui se fait ici en inline, on précise la méthode appelée au clic qui sera :delete, et on ouvre un alert de confirmation ``[confirm: "Are you sure?"]``

Allons y, retournons dans le ``user_controller`` pour ajouter la fonction show ! D'ailleurs j'suis sûr qu'avez tout ce que l'on vous a expliqué, vous pouvez y faire seul !

**Tips:**
- À l'image d'index, cette fonction prend 2 paramètres: la connexion & la map clef valeur id (vous en avez déjà fait dans le ``room_channel.ex``, quand il fallait gérer les messages!)
- Cette fonction va d'abord [récupérer un utilisateur d'id demandé](https://hexdocs.pm/ecto/Ecto.Repo.html#content), à l'aide d'un get par exemple ;-)
- Puis rendre une nouvelle page ``show.html`` en n'oubliant pas de lui faire passer l'utilisateur que l'on a récupéré !

Très bien, en bon expert Phoenix, vous avez remarqué que cette page n'existe pas encore... Vous vous souvenez du dossier ``user`` dans les templates ? Ajoutez y donc un nouveau fichier ``show.html.eex``

```
<h2>Show user</h2>

<ul>
    <li>
        <strong>Email: </strong>
        ........................
    </li>
</ul>

<%= link "Let's go back !", to: user_path(@conn, :index) %>
```

On se contentera ici d'afficher son email par exemple ! À vous de jouer, de la récupération de paramètres dans la connexion ? Facile, j'en ai déjà fait pour la toute première partie !

La dernière balise est un petit lien de retour vers notre index des utilisateurs, pas de dark-magic ici ! :crystal_ball:

On passe maintenant à l'édit, les procédures sont similaires ! On commence donc par implémenter les fonctionnalités du controller :runner:

```
    def new(conn, _params) do
      changeset = User.changeset(%User{}, %{})
      render(conn, "new.html", changeset: changeset)
    end
```

Cette fonction new fait appel au changeset d'un User, mais qu'est ce que cela signifie ? La grosse différence par rapport aux autres fonctions, c'est que l'on récupère la structure type d'un User ! C'est plutôt sympatique pour faire des validations de champs par exemple... :microscope:
Et ce changeset récupéré, nous le poussons vers la page new.html !

D'ailleurs cette fameuse page html elixifié, allons la créer !

```
  <h2>Create a new user</h2>

  <%= render "form.html", changeset: @changeset, action: user_path(@conn, :create) %>

  <%= link "Let's go back !", to: user_path(@conn, :index) %>
```

Alors, qu'est ce que l'on comprends de cette page ? :raising_hand:

- Nous demandons un rendu d'un formulaire en lui passant le changeset et en précisant que l'action sera un appel à :create !
- Très bien Brigitte ! Nous verrons par la suite ce à quoi ``action`` fait référence !

Voici à quoi ressemble le fichier ``form.html.eex`` !
```
<%= form_for @changeset, @action, fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :email, class: "control-label" %>
    <%= email_input f, :email, class: "form-control" %>
    <%= error_tag f, :email %>
  </div>

  <div class="form-group">
    <%= label f, :password, class: "control-label" %>
    <%= password_input f, :password, class: "form-control" %>
    <%= error_tag f, :password %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
```

Le bout de code html le "plus compliqué" auquel nous allons avoir affaire, analysons le ensemble !

``form_for`` & ``@changeset`` font en fait référence à notre donnée, un utilisateur ici. Dans la page précédente, nous avons indiqué l'action lors du rendu de notre page... On le retrouve ici avec le tag``@action``
si changeset vient avec une erreur spécifique, alors on fait le traitement














