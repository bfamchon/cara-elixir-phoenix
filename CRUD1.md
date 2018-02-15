
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

Essayons de récupérer l'ensemble de nos utilisateurs grâce à: ``Repo.all(User)``

Ben ouais BG, c'est un tableau vide, essaye d'insérer la structure User ci-dessous & rejoue la commande !

``Repo.insert(%User{email: "denis.brogniart@kohlant.ah", password: "cabane"})``

Maintenant, ce serait bien d'avoir un contrôleur pour servir d'intermédiaire entre les différentes briques, non ?

Allons y ! On créer le fichier ``lib/chien_web/controler/user_controller.ex`` :computer:

``
defmodule ChienWeb.UserController do
    use ChienrWeb, :controller
    alias Chien.User
    alias Chien.Repo
end
``

Puis celui correspondant à notre vue: 
// TODO trop compris à quoi il servait...

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

Actuellement si vous lancez un ``localhost:4000/users``, Phoenix vous dira qu'il manque un index.html pour l'UserView...

Allons créer le fichier ``lib/chien_web/templates/user/index.html.eex`` 

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

On vous donne en avance les fonctionnalités à implémenter par rapport aux balises ``<%= link %>``