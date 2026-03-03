# Step 06 - Scopes et filtres

## Objectifs

- Ajouter des scopes au modèle Task
- Chaîner des scopes
- Implémenter un filtrage dans le controller via query params
- Créer un formulaire de filtrage dans la vue index

## 6.1 Scopes dans le modèle Task

Ajoute des scopes au modèle Task pour encapsuler des requêtes courantes :

- `urgent` — priority high et pas done
- `overdue` — due_on passée et pas done
- `by_status(status)` — filtre par status (scope avec argument)
- `by_priority(priority)` — filtre par priority

Teste-les en console : `Task.urgent`, `Task.overdue.count`, `Task.by_status(:todo)`, etc.

Essaie de chaîner des scopes : `Task.by_status(:todo).by_priority(:high)`.

> Doc : [Scopes](https://guides.rubyonrails.org/active_record_querying.html#scopes)

## 6.2 Filtrage dans le controller

Modifie l'action `index` du `TasksController` pour filtrer les tâches selon les query params `status` et `priority`.

L'idée : on part de `Task.all`, puis on chaîne les scopes si les params sont présents.

> Doc : [Query Interface](https://guides.rubyonrails.org/active_record_querying.html)

## 6.3 Formulaire de filtrage

Ajoute un formulaire en haut de la vue index avec deux selects (status et priority) et un bouton "Filtrer". Le formulaire doit envoyer une requête GET (pas POST) pour que les filtres apparaissent dans l'URL.

Teste les filtres individuels et les combinaisons.

> Doc : [Form Helpers](https://guides.rubyonrails.org/form_helpers.html)
