# Step 13 - Turbo Morphing

## Objectifs

- Comprendre le concept de morphing (page refresh)
- Simplifier le code en remplaçant les templates Turbo Stream par du morphing
- Découvrir `turbo_refreshes_with` et `Turbo.visit`

## Rappel

Avec Turbo Streams, on écrit des templates `.turbo_stream.erb` qui décrivent précisément quels éléments modifier. C'est puissant, mais ça demande de maintenir un template par action et de penser à chaque élément à mettre à jour (compteur, formulaire, ligne...).

Le **morphing** prend une approche différente : au lieu de cibler des éléments individuels, il demande au serveur de renvoyer la page entière, puis utilise un algorithme de diff (idiomorph) pour ne modifier que ce qui a changé dans le DOM. Le résultat est le même visuellement, mais le code est beaucoup plus simple côté serveur.

> Doc : [Page Refreshes](https://turbo.hotwired.dev/handbook/page_refreshes)

## 13.1 Activer le morphing

Le layout contient déjà `<%= yield :head %>` dans le `<head>`. Il suffit d'utiliser `content_for :head` dans les vues pour y injecter du contenu.

1. Dans `tasks/index.html.erb`, ajoute en haut du fichier :

   ```erb
   <% content_for :head do %>
     <%= turbo_refreshes_with method: :morph, scroll: :preserve %>
   <% end %>
   ```

   Cela ajoute deux balises `<meta>` dans le `<head>` :
   - `method: :morph` active le morphing au lieu du remplacement classique
   - `scroll: :preserve` conserve la position de scroll lors du refresh

## 13.2 Sortir des Turbo Frames

Le morphing ne fonctionne que pour une navigation **pleine page** (page refresh). Or, nos formulaires sont à l'intérieur de `<turbo-frame>` : le `button_to` Destroy est dans la frame `"tasks"`, et le formulaire de création est dans la frame `:new_task`. Quand un formulaire est soumis depuis une frame, Turbo ne fait qu'un **frame update** — il ne déclenche pas le morphing.

Pour que le morphing se déclenche, il faut dire à Turbo de traiter la réponse au niveau de la page entière avec `data-turbo-frame="_top"`.

1. Dans `_task_row.html.erb`, ajoute `data: { turbo_frame: "_top" }` sur le bouton Destroy :

   ```erb
   <%= button_to "Destroy", task, method: :delete,
       class: "btn btn-sm btn-danger",
       data: { turbo_confirm: "Supprimer cette tâche ?", turbo_frame: "_top" } %>
   ```

2. Dans `_form.html.erb`, ajoute `data: { turbo_frame: "_top" }` sur le `form_with` :

   ```erb
   <%= form_with(model: task, data: { turbo_frame: "_top" }) do |form| %>
   ```

3. Teste : crée et supprime une tâche. Le compteur se met à jour correctement car Turbo fait maintenant un page refresh (morphing) au lieu d'un simple frame update.

## 13.3 Refactorer le create

Au lieu de répondre avec un template `turbo_stream`, on va simplement rediriger et laisser le morphing faire le travail.

1. Modifie l'action `create` du contrôleur :

   ```ruby
   def create
     @task = Task.new(task_params)

     if @task.save
       redirect_to tasks_path, notice: "Task was successfully created."
     else
       render :new, status: :unprocessable_entity
     end
   end
   ```

2. Supprime le fichier `app/views/tasks/create.turbo_stream.erb` — on n'en a plus besoin.

3. Teste : crée une tâche. Le morphing compare la page actuelle avec la nouvelle version renvoyée par le serveur et applique uniquement les différences (la nouvelle ligne + le compteur mis à jour).

## 13.4 Refactorer le destroy

1. Modifie l'action `destroy` :

   ```ruby
   def destroy
     @task.destroy!
     redirect_to tasks_path, notice: "Task was successfully destroyed.", status: :see_other
   end
   ```

2. Supprime le fichier `app/views/tasks/destroy.turbo_stream.erb`.

3. Teste : supprime une tâche. La ligne disparaît et le compteur se met à jour, le tout via morphing.

## 13.5 Protéger un élément du morphing avec `data-turbo-permanent`

Parfois, on veut qu'un élément ne soit **pas touché** par le morphing. Par exemple : un menu déroulant ouvert, un `<details>` déplié, un lecteur audio/vidéo en cours de lecture, ou tout élément avec un état éphémère côté client.

L'attribut `data-turbo-permanent` dit à Turbo de ne **jamais** modifier cet élément lors d'un morph. L'élément doit avoir un `id` unique.

```html
<div id="player" data-turbo-permanent>
  <!-- Ce contenu ne sera pas morphé -->
</div>
```

C'est utile quand le morphing "casse" un état visuel que l'utilisateur est en train de manipuler. En pratique, on l'utilise rarement, mais c'est bon à connaître.

> Doc : [Attributs Turbo](https://turbo.hotwired.dev/reference/attributes)

## 13.6 Streams vs Morphing : quand utiliser quoi ?

| Aspect | Turbo Streams | Morphing |
|--------|--------------|----------|
| Complexité serveur | Un template `.turbo_stream.erb` par action | Simple redirection |
| Précision | Chirurgical : on cible exactement les éléments | Automatique : le diff est calculé |
| Performance | Plus léger (envoie uniquement les fragments) | Renvoie la page entière |
| Cas d'usage idéal | Mises à jour en temps réel (WebSocket/broadcasts), pages complexes avec beaucoup d'éléments | CRUD classique, simplification du code |

En pratique, le morphing est souvent suffisant pour les actions CRUD classiques. Les Turbo Streams restent utiles pour les broadcasts en temps réel (quand un autre utilisateur modifie des données) et pour les cas où on a besoin d'un contrôle fin sur les mises à jour.

> Pour aller plus loin : [Page Refreshes with Morphing Demo](https://dev.37signals.com/page-refreshes-with-morphing-demo/) — un article de 37signals qui montre concrètement la différence entre les deux approches.
