# Step 07 - Tags (has_many through)

## Objectifs

- Créer une association many-to-many avec une table de jointure
- Comprendre `has_many :through`
- Utiliser `collection_check_boxes` dans un formulaire
- Gérer les strong parameters avec un tableau

## 7.1 Générer les modèles

Crée deux modèles :

- `Tag` avec un attribut `name` (string, unique)
- `Tagging` comme table de jointure entre Tag et Task (avec `tag:references` et `task:references`)

> Doc : [Has Many Through](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)

## 7.2 Configurer les associations

Configure les associations dans les trois modèles :

- `Tag` a plusieurs taggings (et des tasks à travers les taggings)
- `Tagging` appartient à un tag et à une task
- `Task` a plusieurs taggings et des tags à travers les taggings

Ajoute une validation de présence et d'unicité sur `Tag#name`.

Teste en console : crée des tags, associe-les à des tâches.

## 7.3 Ajouter les tags au formulaire Task

Utilise `collection_check_boxes` pour afficher des checkboxes avec tous les tags disponibles dans le formulaire Task.

Mets à jour les strong parameters pour autoriser `tag_ids: []` (un tableau d'ids).

> Doc : [collection_check_boxes](https://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-collection_check_boxes)

## 7.4 Afficher les tags

Affiche les tags associés à chaque tâche sous forme de badges Bootstrap dans les vues index et show.

## 7.5 Seeds

Ajoute quelques tags dans les seeds et associe-les aux tâches existantes.
