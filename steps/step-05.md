# Step 05 - Project et associations

## Objectifs

- Générer un nouveau modèle avec scaffold
- Configurer les associations `has_many` / `belongs_to`
- Comprendre `dependent: :destroy`
- Ajouter un select d'association dans un formulaire
- Mettre à jour les seeds

## 5.1 Scaffold Project

Génère le modèle Project avec un scaffold :

```bash
bin/rails generate scaffold Project name:string
```

Exécute la migration et teste le CRUD dans le navigateur.

Ajoute une validation de présence sur le `name` du projet.

## 5.2 Associer Task à Project

Une tâche peut appartenir à un projet (optionnellement). Un projet a plusieurs tâches.

Génère une migration pour ajouter la référence `project` à la table `tasks`.

Ensuite, reflète cette association dans les classes de modèles (`has_many`, `belongs_to`). Une tâche peut exister sans projet — cherche comment rendre l'association optionnelle. Ajoute `dependent: :destroy` pour que les tâches soient supprimées quand un projet est supprimé.

Teste en console : crée un projet, ajoute-lui des tâches via l'association (`project.tasks.create!(...)`).

> Doc : [Associations](https://guides.rubyonrails.org/association_basics.html)

## 5.3 Formulaire Task : sélectionner un projet

Ajoute un select dans le formulaire Task pour choisir un projet. Utilise `collection_select` ou `select` avec les projets disponibles. N'oublie pas d'autoriser le paramètre `project_id` dans les strong parameters.

Ajoute une option vide pour les tâches sans projet.

> Doc : [Form Helpers - Select](https://guides.rubyonrails.org/form_helpers.html#making-select-boxes-with-ease)

## 5.4 Afficher le projet dans les vues Task

Dans l'index et le show des tâches, affiche le nom du projet associé (s'il existe). Ajoute un lien vers le projet.

Dans le show d'un projet, affiche la liste de ses tâches.

## 5.5 Styliser les vues Project

Comme pour Task, adapte les vues générées par le scaffold avec Bootstrap (formulaire, index en tableau ou en cards).

Mets à jour la navbar pour ajouter un lien vers les projets.

## 5.6 Seeds

Mets à jour `db/seeds.rb` pour créer des projets et associer des tâches. Utilise `find_or_create_by` pour rendre les seeds idempotentes (rejouables sans dupliquer les données).

> Doc : [Seeds](https://guides.rubyonrails.org/active_record_migrations.html#migrations-and-seed-data)
