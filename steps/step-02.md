# Step 02 - Enrichir le modèle Task

## Objectifs

- Ajouter des colonnes à un modèle existant via une migration
- Ajouter des validations
- Configurer des enums
- Écrire des méthodes de modèle
- Créer des seeds

## 2.1 Ajouter des colonnes

Le modèle Task est trop simple. On veut ajouter :
- `status` (integer) — pour stocker un enum (todo, in_progress, done)
- `priority` (integer) — pour stocker un enum (low, medium, high)
- `due_on` (date) — la date d'échéance

Génère une migration pour ajouter ces trois colonnes, puis exécute-la.

> Doc : [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html#creating-a-standalone-migration)

## 2.2 Ajouter des validations

Ajoute les validations suivantes au modèle Task :
- Le titre doit être **présent** et avoir une longueur entre **3 et 255 caractères**
- Le status doit être **présent**
- La priority doit être **présente**

Teste en console :
- Essaie de créer une tâche sans titre
- Essaie avec un titre de 2 caractères
- Inspecte les erreurs avec `valid?`, `errors`, `errors.full_messages`

> Doc : [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html)

## 2.3 Configurer les enums

Dans le modèle Task, configure deux enums :
- `status` avec les valeurs `todo`, `in_progress`, `done`
- `priority` avec les valeurs `low`, `medium`, `high`

Teste en console les méthodes générées automatiquement par les enums :
- Les prédicats (`task.high?`, `task.done?`)
- Les scopes (`Task.high`, `Task.todo`)
- Le changement de valeur (`task.high!`)

> Doc : [Active Record Enums](https://api.rubyonrails.org/classes/ActiveRecord/Enum.html)

## 2.4 Méthodes de modèle

Ajoute deux méthodes prédicats au modèle Task :
- `urgent?` — renvoie `true` si la priority est "high" **et** la tâche n'est pas done
- `overdue?` — renvoie `true` si `due_on` est passée **et** la tâche n'est pas done (attention au cas où `due_on` est nil)

Teste chaque méthode en console.

## 2.5 Seeds

Crée quelques tâches dans `db/seeds.rb` avec des statuts, priorités et dates d'échéance variés. Exécute les seeds :

```bash
bin/rails db:seed
```

Vérifie en console que les données sont bien là.

> Doc : [Seeds](https://guides.rubyonrails.org/active_record_migrations.html#migrations-and-seed-data)
