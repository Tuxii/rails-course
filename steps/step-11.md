# Step 11 - Turbo Frames

## Objectifs

- Comprendre le concept de Turbo Frame
- Éditer un projet inline sans quitter la page show
- Créer une nouvelle tâche inline depuis l'index
- Charger du contenu en lazy loading avec `loading: :lazy`
- Combiner Turbo Frames et Stimulus pour filtrer sans bouton

## Rappel

Un Turbo Frame est une portion de page délimitée par un `<turbo-frame id="...">`. Quand un lien ou formulaire à l'intérieur d'un frame est cliqué, seul le contenu du frame est remplacé — le reste de la page ne bouge pas.

La règle : la réponse du serveur doit contenir un `<turbo-frame>` avec le **même id**. Turbo extrait ce frame et remplace le contenu correspondant.

> Doc : [Turbo Frames](https://turbo.hotwired.dev/handbook/frames)

## 11.1 Édition inline d'un projet

On veut pouvoir cliquer sur "Modifier" sur la page show d'un projet et voir le formulaire apparaître **à la place des infos du projet**, sans quitter la page.

### Côté show

1. Dans `projects/show.html.erb`, entoure les informations du projet (le nom, le badge archivé, les tags) et le lien "Modifier" dans un `turbo_frame_tag` :

   ```erb
   <%= turbo_frame_tag @project do %>
     <h1>
       <%= @project.name %>
       ...
     </h1>
     ...
     <%= link_to "Modifier", edit_project_path(@project), class: "btn btn-outline-secondary" %>
   <% end %>
   ```

   Le `turbo_frame_tag @project` génère un `<turbo-frame id="project_1">` (via `dom_id`).

   Laisse le tableau des tâches et les autres boutons (Archiver, Retour) **en dehors** du frame.

### Côté edit

2. Dans `projects/edit.html.erb`, entoure le formulaire avec le même frame :

   ```erb
   <%= turbo_frame_tag @project do %>
     <h1>Editing project</h1>
     <%= render "form", project: @project %>
     <%= link_to "Annuler", @project, class: "btn btn-outline-secondary" %>
   <% end %>
   ```

3. Teste : va sur la page show d'un projet, clique sur "Modifier". Le formulaire remplace les infos du projet. Modifie le nom et soumets — les infos se mettent à jour sans recharger la page.

### Que se passe-t-il ?

- Le clic sur "Modifier" envoie une requête GET à `/projects/:id/edit`
- Turbo cherche dans la réponse un `<turbo-frame>` avec le même id (`project_1`)
- Il remplace le contenu du frame dans la page show par le formulaire
- À la soumission, le contrôleur redirige vers show — Turbo refait la même chose en sens inverse
- Le lien "Annuler" est dans le frame : il pointe vers show, donc il remet les infos du projet

### Le lien "Modifier" depuis l'index

Depuis l'index des projets, le lien "Edit" pointe aussi vers `/projects/:id/edit`. Clique dessus : que se passe-t-il ?

Le lien n'est pas dans un frame, donc Turbo Drive fait une navigation classique — on arrive sur la page edit complète. C'est le comportement par défaut quand un lien n'est pas dans un `<turbo-frame>` : Turbo Drive prend le relais.

## 11.2 Nouvelle tâche inline

On veut pouvoir créer une tâche sans quitter l'index, en affichant le formulaire en bas de page.

1. Dans `tasks/index.html.erb`, entoure le lien "New task" avec un turbo frame :

   ```erb
   <%= turbo_frame_tag "new_task" do %>
     <%= link_to "New task", new_task_path %>
   <% end %>
   ```

2. Dans `tasks/new.html.erb`, entoure le formulaire avec le même frame :

   ```erb
   <%= turbo_frame_tag "new_task" do %>
     <h2>New task</h2>
     <%= render "form", task: @task %>
     <%= link_to "Annuler", tasks_path %>
   <% end %>
   ```

3. Teste : clique sur "New task" — le formulaire remplace le lien. Soumets — si la validation passe, que se passe-t-il ?

### Problème : la redirection après création

Après `create`, le contrôleur redirige vers `show`. Mais la page show ne contient pas de frame `new_task`. Le frame se vide.

Pour l'instant, modifie la redirection du `create` pour revenir vers l'index :

```ruby
format.html { redirect_to tasks_path, notice: "Task was successfully created." }
```

Le formulaire disparaît et le lien "New task" réapparaît — mais la nouvelle tâche n'apparaît pas dans le tableau ! C'est normal : Turbo ne met à jour que le contenu du frame `new_task`, il ne touche pas au reste de la page.

C'est la limite des Turbo Frames : **un frame ne peut mettre à jour qu'un seul élément**. Pour ajouter la ligne dans le tableau ET mettre à jour le compteur ET remettre le lien, il faudra Turbo Streams (step suivante).

## 11.3 Lazy loading avec turbo frame

Un Turbo Frame avec un attribut `src` charge automatiquement son contenu depuis l'URL indiquée. Par défaut (`loading: :eager`), la requête est envoyée immédiatement au chargement de la page.

Avec `loading: :lazy`, la requête est **différée** : le contenu n'est chargé que lorsque le frame entre dans le viewport (la zone visible de la page). C'est parfait pour du contenu secondaire qu'on ne veut pas charger inutilement.

On va afficher la liste des tâches d'un projet en lazy loading sur la page show du projet.

1. Crée une route imbriquée (nested) pour les tâches d'un projet :

   ```ruby
   resources :projects do
     resources :tasks, only: [:index], controller: "projects/tasks"
   end
   ```

   (garde les routes `member` pour archive/unarchive)

2. Crée le contrôleur `app/controllers/projects/tasks_controller.rb` :

   ```ruby
   class Projects::TasksController < ApplicationController
     def index
       @project = Project.find(params[:project_id])
       @tasks = @project.tasks
     end
   end
   ```

3. Crée la vue `app/views/projects/tasks/index.html.erb` avec le tableau des tâches du projet, entouré d'un turbo frame :

   ```erb
   <%= turbo_frame_tag "project_tasks" do %>
     <table class="table table-striped">
       <!-- le même tableau que dans projects/show -->
     </table>
   <% end %>
   ```

4. Dans `projects/show.html.erb`, remplace le tableau des tâches par un frame lazy :

   ```erb
   <%= turbo_frame_tag "project_tasks", src: project_tasks_path(@project), loading: :lazy do %>
     <p class="text-muted">Chargement des tâches...</p>
   <% end %>
   ```

5. Teste en deux temps :

   **D'abord, vérifie que ça marche** : ouvre la page d'un projet. Le texte "Chargement des tâches..." apparaît brièvement, puis le tableau se charge. Observe la requête dans l'onglet Network.

   **Ensuite, observe le lazy loading** : pour bien voir que le chargement ne se fait qu'à l'entrée dans le viewport, ajoute temporairement une série de `<br>` avant le frame dans le show (assez pour pousser le frame sous la zone visible). Recharge la page et ouvre l'onglet Network : aucune requête vers `project_tasks` n'est envoyée. Scrolle vers le bas — la requête part au moment où le frame devient visible.

   Retire les `<br>` quand tu as fini.

## 11.4 Filtres sans rechargement (Turbo Frames + Stimulus)

On va faire en sorte que les filtres de l'index des tâches fonctionnent sans bouton, en soumettant automatiquement le formulaire quand un select change.

### Le Turbo Frame

1. Dans `tasks/index.html.erb`, entoure le tableau dans un turbo frame :

   ```erb
   <%= turbo_frame_tag "tasks" do %>
     <table class="table table-striped table-bordered">
       ...
     </table>
   <% end %>
   ```

2. Sur le formulaire de filtres, ajoute `data: { turbo_frame: "tasks" }` pour indiquer que la réponse doit mettre à jour le frame `tasks` :

   ```erb
   <%= form_tag tasks_path, method: :get, data: { turbo_frame: "tasks" } do %>
   ```

   Le formulaire reste en dehors du frame — seul le tableau est remplacé par la réponse.

3. Teste en cliquant sur "Filter" — seul le tableau est rechargé, le reste de la page ne bouge pas. Pour le vérifier, ajoute temporairement un `<%= Time.current %>` en dehors du frame (par exemple sous le `<h1>`) : l'heure ne change jamais quand tu filtres, ce qui prouve que seul le frame est mis à jour.

### Le Stimulus controller

On va créer un petit contrôleur Stimulus pour soumettre le formulaire automatiquement au changement d'un select.

4. Génère le contrôleur Stimulus avec le générateur Rails :

   ```bash
   bin/rails generate stimulus auto_submit
   ```

   Ça crée le fichier `app/javascript/controllers/auto_submit_controller.js` et met à jour l'index. Remplace le contenu par :

   ```javascript
   import { Controller } from "@hotwired/stimulus"

   export default class extends Controller {
     submit() {
       this.element.requestSubmit()
     }
   }
   ```

5. Connecte le contrôleur au formulaire et écoute l'événement `change` sur les selects :

   ```erb
   <%= form_tag tasks_path, method: :get, data: { turbo_frame: "tasks", controller: "auto-submit" } do %>
     <%= select_tag :status, ..., data: { action: "auto-submit#submit" } %>
     <%= select_tag :priority, ..., data: { action: "auto-submit#submit" } %>
   ```

6. Tu peux maintenant retirer le bouton "Filter" (le `submit_tag`) puisque le formulaire se soumet automatiquement.

7. Teste : change un filtre — la liste se met à jour instantanément.

> Doc : [Stimulus Handbook](https://stimulus.hotwired.dev/handbook/introduction)
