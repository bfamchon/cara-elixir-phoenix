## 3ème étape: C'est bien beau tout ça... Mais quand est-ce qu'on parle ?

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
