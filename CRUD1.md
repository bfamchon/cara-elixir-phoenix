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

Comme présenté dans le talk introductif, la commande ``iex -S mix phx.server`` va lancer un shell interractif en prenant compte vos sources compilées !

Commençons par définir quelques alias afin d'y accéder plus facilement dans nos commandes:
``alias Chien.Repo`` & ``alias Chien.User``

Essayons de récupérer l'ensemble de nos utilisateurs grâce à: ``Repo.all(User)``... Ben ouais BG, c'est un tableau vide, essaye d'insérer la structure User ci-dessous & rejoue la commande !

``Repo.insert(%User{email: "denis.brogniart@kohlant.ah", password: "cabane"})``

Si tu es jeune, beau, et intéréssé par Ecto, tu peux allez te renseigner sur les merveilleuses méthodes de ton repository préféré [ici](https://hexdocs.pm/ecto/Ecto.Repo.html)

***Maintenant, ce serait bien d'avoir un contrôleur pour servir d'intermédiaire entre les différentes briques, non ?***

Allons y ! On crée le fichier ``lib/chien_web/controler/user_controller.ex`` :computer:

```elixir
defmodule ChienWeb.UserController do
    use ChienWeb, :controller
    alias Chien.User
    alias Chien.Repo
end
```

Dans ce controller, ajoutons le point d'entrée principal qui listera tous les utilisateurs:

```elixir
    def index(conn, _params) do
      users = Repo.all(User)
      render(conn, "index.html", users: users)
    end
```

Nous n'avons pas encore routé ce chemin... Allons faire un tour dans le fichier ``lib/chien_web/router.ex`` pour ajouter notre ressource User :

```elixir
  scope "/", ChienWeb do
    ...
    resources "/users", UserController
    ...
  end
```

Nous avons été assez impréssioné par le fait qu'une simple ligne resources puisse implicitement déclarer une multitude d'endpoints pour notre router n'hésitez pas à utiliser ``mix phx.routes`` pour les visualiser !


Ajoutons maintenant une page permettant de gérer nos utilisateurs !

Avec Phoenix, chaque pages sont liées à un module permettant de la manager. On pourra y mettre des méthodes que l'on appelera dans la page afin d'effectuer différents traitements.
Mais nous, on utilise Javascript ! Nous devons tout de même lier notre page à ce module, mais nous le laisserons vide.
Rendez-vous dans ``chien_web/views`` et créez le module suivant (comme pour le controller, il faudra le déclarer en temps que view)
```elixir
defmodule ChienWeb.UserView do
    use ChienWeb, :view
end
```
Il faut maintenant lui associer une page ! Allons créer le fichier ``lib/chien_web/templates/user/index.html.eex`` !
Vous l'avez peut être déjà remarqué grâce aux pages générées, mais votre view sera liée à votre "Groupe de page" situé dans le dossier ``lib/chien_web/templates/user`` grâce à son nom !

À remplir avec:
```html
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

On vous donne en avance les fonctionnalités à implémenter par rapport aux balises ``<%= link %>``. Rien de sorcier là non plus, on boucle sur les utilisateurs de notre ``@conn`` & on met à dispo 4 liens qui vont diriger vers le ``user_path`` (/users), en y ajoutant les spécificités pour la navigation :
- :show (/users/:id),
- :edit (/users/:id/edit),
- :new (/users/new)
- :delete qui se fait ici en inline, on précise la méthode appelée au clic qui sera :delete, et on ouvre un alert de confirmation ``[confirm: "Are you sure?"]``


Allons y, retournons dans le ``user_controller`` pour ajouter la fonction show ! D'ailleurs j'suis sûr qu'avec tout ce que l'on vous a expliqué, vous pouvez y faire seul !

**Tips:**
- À l'image d'index, cette fonction prend 2 paramètres: la connexion & la map clef valeur id (vous en avez déjà fait dans le ``room_channel.ex``, quand il fallait gérer les messages!)
- Cette fonction va d'abord [récupérer un utilisateur d'id demandé](https://hexdocs.pm/ecto/Ecto.Repo.html#content), à l'aide d'un get par exemple ;-)
- Puis rendre une nouvelle page ``show.html`` en n'oubliant pas de lui faire passer l'utilisateur que l'on a récupéré !

Très bien, en bon expert Phoenix, vous avez remarqué que cette page n'existe pas encore... Vous vous souvenez du dossier ``user`` dans les templates ? Ajoutez y donc un nouveau fichier ``show.html.eex``

```html
<h2>Show user</h2>

<ul>
    <li>
        <strong>Email: </strong>
        ........................
    </li>
</ul>

<%= link "Let's go back !", to: user_path(@conn, :index) %>
```

On se contentera ici d'afficher son email par exemple ! À vous de jouer ! Récupérez le paramètres ! Facile, j'en ai déjà fait pour la toute première partie !

La dernière balise est un petit lien de retour vers notre index des utilisateurs, pas de dark-magic ici ! :crystal_ball:

On passe maintenant à l'édit, les procédures sont similaires ! On commence donc par implémenter les fonctionnalités du controller :runner:

```elixir
    def new(conn, _params) do
      changeset = User.changeset(%User{}, %{})
      render(conn, "new.html", changeset: changeset)
    end
```

Cette fonction new fait appel au changeset d'un User, mais qu'est ce que cela signifie ? La grosse différence par rapport aux autres fonctions, c'est que l'on récupère la structure type d'un User ! C'est plutôt sympatique pour faire des validations de champs par exemple... :microscope:
Et ce changeset récupéré, nous le poussons vers la page new.html !

D'ailleurs cette fameuse page html elixifié, allons la créer !

```html
  <h2>Create a new user</h2>

  <%= render "form.html", changeset: @changeset, action: user_path(@conn, :create) %>

  <%= link "Let's go back !", to: user_path(@conn, :index) %>
```

Alors, qu'est ce que l'on comprends de cette page ? :raising_hand:
Nous demandons un rendu de la page form.html et nous ajoutons dans le scope le changeset et l'action :create !

Mais une seconde, nous n'avons aucune page form.html !? Vous avez compris la prochaine étape (sauf peut être un ou deux, oui Mustapha, je parles de toi).

Créer la page suivante (vous devriez savoir ou la placer) :
```html
<%= form_for @changeset, @action, fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Ah !</p>
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
Vous devriez être capable de comprendre la page tout seul, pour toute question, appelez nous ! 

Nous souhaitons donc utiliser la méthode create ! Il faut bien sûre la créer dans notre User_controller !
Ajoutez donc :  
```elixir
    def create(conn, %{"user" => user_params}) do
      changeset = User.changeset(%User{}, user_params)

      case Repo.insert(changeset) do
        {:ok, _user} ->
          conn
          |> put_flash(:info, "User created successfully.")
          |> redirect(to: user_path(conn, :index))
        {:error, changeset} ->
          render(conn, "new.html", changeset: changeset)
      end
    end
```
Mais qu'est-ce que c'est que ce code ? Une explication s'impose ! Tout d'abord on récupère un changeSet à partir de l'utilisateur passé en paramètre, un changeSet est une structure représentant les modifications à effectuer dans la base de données. Ensuite, nous insérons le changeSet en base et traitons la réponse. Commençons par le cas simple :error nous redirigera vers new.html en alimentant changeset, si vous avez bien analysé le code vous observez qu'un "Ah !" apparait dans ce cas !   
Le cas :ok, vous pouvez voir la programmation fonctionnelle dans toute sa splendeur ! Nous associons dans le flash la clef :info et le message "User created successfully", le flash est simplement une zone mémoire "tampon" stockée en session qui quand le message sera consommé, sera supprimé. Pour finir, la conn retournée par la méthode put_flash est utilisé pour rediriger vers l'index.    
Félicitations, vous pouvez insérer votre utilisateur en base !    

Passons maintenant à l'update ! L'avantage avec un formulaire d'édition, c'est que c'est que c'est le même que celui de création avec une autre action. Je penses que vous êtes capable de créer la page edit.html.eex correspondant à notre souhait ainsi que la méthode edit dans le controlleur seul ! Dans la page que vous créerai, nous souhaitons accéder à l'action suivante :
```elixir
	def update(conn, %{"id" => id, "user" => user_params}) do
      user = Repo.get!(User, id)
      changeset = User.changeset(user, user_params)
      case Repo.update(changeset) do
        {:ok, user} ->
          conn
          |> put_flash(:info, "User updated successfully.")
          |> redirect(to: user_path(conn, :show, user))
        {:error, changeset} ->
          render(conn, "edit.html", user: user, changeset: changeset)
      end
  	end
```
Donc n'oubliez pas, il faudra avoir accès à l'id de l'utilisateur, le fournir à notre page d'edition serai une bonne idée !

Si vous n'y arrivez vraiment pas, allez voir plus bas !     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
Allez Ludo essai vraiment de le faire avant d'allez voir la solution...     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
|     
```elixir
 def edit(conn, %{"id" => id}) do
      user = Repo.get!(User, id)
      changeset = User.changeset(user)
      render(conn, "edit.html", user: user, changeset: changeset)
  	end   
```
```html
<h2>Edit user</h2>

<%= render "form.html", changeset: @changeset,
                        action: user_path(@conn, :update, @user) %>

<%= link "Back", to: user_path(@conn, :index) %>
```

Pour finir notre beau CRUD, il faut la méthode delete ! Et ce coup-ci on ne vous aidera pas ! Allez y et vous pourrez vous vanter d'avoir finis le TP !

N'oubliez pas de mettre le plus de points possible lorsque vous recevrez le mail :)

PS : Francis I <3 U
