## 3ème étape: C'est bien tout ça... Mais quand est-ce qu'on parle ?

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

Attaquons désormais le javascript pour cette partie !

On retourne dans notre fichier js principal pour y ajouter la gestion des messages:

```
let messageInput = document.getElementById('NewMessage');
messageInput.addEventListener('keypress', e => {
  if (e.keyCode == 13 && messageInput.value != '') {
    room.push('message:new', messageInput.value);
    messageInput.value = '';
  }
});

```

Ici, nous allons gérer les entrées utilisateurs: quand il appuie sur ``Enter``, on push un nouveau message dans le channel ``message:new`` que l'on vient de créer !

```
let messageList = document.getElementById('MessageList');
let renderMessage = message => {
  let messageElement = document.createElement('li');
  messageElement.innerHTML = `
    <b>${message.user}</b>
    <i>${formatedTimestamp(message.timestamp)}</i>
    <p>${message.body}</p>
  `;
  messageList.appendChild(messageElement);
  messageList.scrollTop = messageList.scrollHeight;
};

room.on('message:new', message => renderMessage(message));
```

Nous venons ensuite gérer la récéption de messages grâce à ``room:on()``, qui à un message appliquera la fonction renderMessage pour créer du html dans notre liste !


Rechargez votre page et... Admirez votre boulot :tada:

Allez, c'est l'heure pour vous de prendre une petite pause avant d'attaquer [un petit CRUD avec actions en BDD](https://github.com/unip62/cara-elixir-phoenix/blob/master/CRUD1.md) !
