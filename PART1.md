
# La recette !
## 1ère étape: Hello benjam' !

La majorité du projet web phoenix va se trouver dans chien_web.
On va alors modifier le fichier `templates/page/index.html.eex` de manière à préparer notre interface !
```html
	<div class="row">
	  <div class="col-md-12 alert alert-info">
	    Hello, <span id="User">
	    <%#= HERE %>
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

Vous voyez cette balise ``<%= # HERE %>`` ? ``<%= %>`` vous permettra d'ajouter du code elixir qui sera interprété dans votre page HTML tandis que le ``#`` est un commentaire ! Ce serait sympa de pouvoir afficher le nom de notre utilisateur non ?

L'avantage dans Phoenix, c'est qu'il est plutôt simple de manipuler les paramètres http ! Let's go, on veut pouvoir taper ``localhost:4000?user=Benjam'``

On va ajouter ``<%= @conn.params["user"] %>``, qui se chargera, dans la requête HTTP, de récupérer le paramètre  ``user`` & de l'afficher ! 


***Et maintenant... [Si on gérait le multi-user ?](https://github.com/unip62/cara-elixir-phoenix/blob/master/PART2.md)***
