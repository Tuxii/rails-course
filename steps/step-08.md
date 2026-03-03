# Step 08 - Callbacks et Concerns

## Objectifs

- Découvrir les callbacks Rails
- Créer un concern réutilisable (Archivable)
- Ajouter des routes member custom
- Utiliser `button_to` pour déclencher des actions

## 8.1 Callback : valeur par défaut

Dans le modèle Task, ajoute un callback `before_validation` pour définir le status à `todo` par défaut quand il n'est pas renseigné.

Teste en console :

```
task = Task.new(title: "Test")
task.valid?
task.status  # => "todo"
```

> Doc : [Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html)

## 8.2 Ajouter la colonne archived

Génère une migration pour ajouter une colonne `archived` (boolean, default: false) au modèle Project.

## 8.3 Créer le concern Archivable

Crée un module `Archivable` dans `app/models/concerns/` qui fournit :

- Des scopes `archived` et `active`
- Des méthodes `archive` / `archive!` et `unarchive` / `unarchive!`

La version sans bang modifie l'attribut en mémoire sans sauvegarder (comme un setter). La version avec bang sauvegarde en base avec `update!`. C'est cohérent avec la convention Ruby : `sort` retourne une copie, `sort!` modifie en place.

Le prédicat `archived?` est déjà fourni par Rails grâce à la colonne boolean, pas besoin de le redéfinir.

Inclus ce concern dans Project.

Rappel : un concern Rails utilise `extend ActiveSupport::Concern` et le block `included do ... end` pour les scopes.

> Doc : [Concerns](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html)

## 8.4 Routes member et actions controller

Les 7 actions REST ne couvrent pas tout. Pour archiver/désarchiver, on a besoin de routes custom sur un projet spécifique.

Ajoute des routes `member` pour `archive` et `unarchive` dans le fichier de routes. Vérifie avec `bin/rails routes -g project` que les nouvelles routes apparaissent.

Ajoute les actions correspondantes dans `ProjectsController`. Elles doivent appeler les méthodes du concern et rediriger vers le projet avec un message flash.

> Doc : [Adding More RESTful Actions](https://guides.rubyonrails.org/routing.html#adding-more-restful-actions)

## 8.5 Mettre à jour les vues

Dans la vue show du projet :
- Ajoute un bouton "Archiver" ou "Désarchiver" selon l'état du projet (utilise `button_to` avec `method: :patch`)
- Affiche un badge si le projet est archivé

Dans la vue index :
- Par défaut, n'affiche que les projets actifs
- Ajoute un lien pour voir les projets archivés (filtre via query param)
