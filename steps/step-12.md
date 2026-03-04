# Step 12 - Turbo Streams

## Objectifs

- Comprendre la différence entre Turbo Frames et Turbo Streams
- Utiliser les actions `append`, `remove`, `update` et `replace`
- Mettre à jour plusieurs éléments de la page en une seule réponse
- Extraire un partial de ligne pour le réutiliser

## Rappel

Un **Turbo Frame** remplace un seul élément (le frame). Un **Turbo Stream** peut modifier **plusieurs éléments** de la page en une seule réponse, avec des actions comme `append`, `prepend`, `replace`, `update`, `remove`.

Le format est `turbo_stream` : le contrôleur répond avec un template `.turbo_stream.erb` au lieu de rediriger.

> Doc : [Turbo Streams](https://turbo.hotwired.dev/handbook/streams)

## 12.1 Préparer le terrain

Pour que Turbo Streams puisse cibler les éléments de la page, on a besoin d'identifiants HTML précis.

1. **Extraire un partial pour la ligne de tâche** : Crée un fichier `app/views/tasks/_task_row.html.erb` avec le contenu d'un `<tr>` de l'index. Utilise `dom_id(task)` pour donner un id unique à chaque ligne :

   ```erb
   <tr id="<%= dom_id(task) %>">
     <td>...</td>
     <!-- les mêmes colonnes que dans l'index -->
   </tr>
   ```

2. Dans `tasks/index.html.erb`, remplace la boucle `each` par un render de collection :

   ```erb
   <tbody id="task_rows">
     <%= render partial: "task_row", collection: @tasks, as: :task %>
   </tbody>
   ```

   Note l'`id="task_rows"` sur le `<tbody>` — on en aura besoin pour le `append`.

3. Vérifie que l'index fonctionne toujours comme avant.

## 12.2 Ajouter une tâche avec append

On va remplacer le comportement du formulaire "New task" : au lieu de recharger la page, on va ajouter la nouvelle ligne dans le tableau et mettre à jour le compteur.

1. Dans le contrôleur, modifie l'action `create` pour répondre en `turbo_stream` :

   ```ruby
   def create
     @task = Task.new(task_params)

     if @task.save
       respond_to do |format|
         format.turbo_stream
         format.html { redirect_to tasks_path, notice: "Task was successfully created." }
       end
     else
       render :new, status: :unprocessable_entity
     end
   end
   ```

2. Crée le template `app/views/tasks/create.turbo_stream.erb` :

   ```erb
   <%= turbo_stream.append "task_rows" do %>
     <%= render "task_row", task: @task %>
   <% end %>

   <%= turbo_stream.update "task_count" do %>
     <%= "#{Task.count} #{"tâche".pluralize(Task.count)}" %>
   <% end %>

   <%= turbo_stream.update "new_task" do %>
     <%= link_to "New task", new_task_path %>
   <% end %>
   ```

   Ce template fait trois choses en une seule réponse :
   - **append** la nouvelle ligne dans le `<tbody id="task_rows">`
   - **update** le compteur `<p id="task_count">`
   - **update** le frame `new_task` pour remettre le lien à la place du formulaire

3. Teste : clique sur "New task", remplis le formulaire. La tâche apparaît dans le tableau, le compteur se met à jour, et le lien "New task" réapparaît. Pas de rechargement de page.

## 12.3 Supprimer une tâche avec remove

1. Modifie l'action `destroy` pour répondre en `turbo_stream` :

   ```ruby
   def destroy
     @task.destroy!

     respond_to do |format|
       format.turbo_stream
       format.html { redirect_to tasks_path, notice: "Task was successfully destroyed.", status: :see_other }
     end
   end
   ```

2. Crée le template `app/views/tasks/destroy.turbo_stream.erb` :

   ```erb
   <%= turbo_stream.remove @task %>

   <%= turbo_stream.update "task_count" do %>
     <%= "#{Task.count} #{"tâche".pluralize(Task.count)}" %>
   <% end %>
   ```

   `turbo_stream.remove @task` cherche l'élément avec l'id `dom_id(@task)` (ex: `task_42`) et le supprime du DOM.

3. Teste : clique sur "Destroy" sur une tâche. La ligne disparaît et le compteur se met à jour.

## Récapitulatif des actions Turbo Stream

| Action    | Ce qu'elle fait                                         |
|-----------|---------------------------------------------------------|
| `append`  | Ajoute du contenu **à la fin** d'un élément cible      |
| `prepend` | Ajoute du contenu **au début** d'un élément cible      |
| `replace` | Remplace l'élément cible **entièrement** (tag compris)  |
| `update`  | Remplace le **contenu** de l'élément (garde le tag)     |
| `remove`  | Supprime l'élément cible du DOM                         |
