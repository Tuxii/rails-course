# Step 09 - Tagging polymorphe (bonus)

## Objectifs

- Comprendre le polymorphisme dans les associations Rails
- Transformer la table Tagging pour qu'elle soit polymorphe
- Permettre aux Projects d'avoir des tags aussi

## 9.1 Pourquoi le polymorphisme ?

La table `taggings` a actuellement une colonne `task_id`. Si on veut aussi taguer les Projects, on pourrait ajouter une colonne `project_id`… mais ça ne scale pas.

Le polymorphisme remplace `task_id` par deux colonnes : `taggable_type` (le nom du modèle) et `taggable_id` (l'id de l'enregistrement).

## 9.2 Migration

Crée une migration pour transformer la table `taggings` :

- Supprime la colonne `task_id`
- Ajoute les colonnes `taggable_type` (string) et `taggable_id` (integer)
- Ajoute un index sur `[:taggable_type, :taggable_id]`

> Doc : [Polymorphic Associations](https://guides.rubyonrails.org/association_basics.html#polymorphic-associations)

## 9.3 Mettre à jour les associations

Modifie les modèles :

- `Tagging` : remplace `belongs_to :task` par `belongs_to :taggable, polymorphic: true`
- `Task` : utilise `has_many :taggings, as: :taggable`
- `Project` : ajoute les mêmes associations que Task pour les tags

## 9.4 Ajouter les tags aux formulaires Project

Comme pour Task, ajoute les checkboxes de tags dans le formulaire Project. Mets à jour les strong parameters.

Affiche les tags en badges dans les vues Project (index et show).

## 9.5 Re-seeder et tester

La migration a supprimé `task_id`, donc les taggings existants sont perdus. Mets à jour le `seeds.rb` et relance `bin/rails db:seed` pour recréer les données.

En console, vérifie qu'un même tag peut être associé à un Task et à un Project :

```bash
bin/rails console
tag = Tag.first
tag.taggings  # => Tagging avec taggable_type "Task" et "Project"
```

> Doc : [Has Many Through](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
