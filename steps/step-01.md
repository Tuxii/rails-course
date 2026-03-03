# Step 01 - Création de l'app et découverte d'Active Record

## Objectifs

- Créer une application Rails
- Générer un premier modèle
- Découvrir les méthodes Active Record en console

## 1.1 Générer l'application

```bash
rails new taskmanager --css=bootstrap --skip-test
cd taskmanager
```

Explore la structure des dossiers générée. Repère en particulier :
- `app/` — le code de l'application (models, views, controllers)
- `config/` — la configuration (routes, database, etc.)
- `db/` — les migrations et les seeds
- `Gemfile` — les dépendances

Lance le serveur et vérifie que http://localhost:3000 fonctionne :

```bash
bin/dev
```

## 1.2 Générer le modèle Task

```bash
bin/rails generate model Task title:string
```

Observe les fichiers générés. Ouvre la migration et lis son contenu avant de l'exécuter :

```bash
bin/rails db:migrate
```

> Doc : [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html)

## 1.3 Découvrir Active Record en console

Ouvre la console Rails :

```bash
bin/rails console
```

Explore les méthodes suivantes. Pour chacune, observe ce qui se passe en base (regarde les requêtes SQL affichées) :

**Créer des enregistrements :**
- Crée une tâche avec `Task.new` puis `save`
- Crée une tâche directement avec `Task.create`
- Quelle différence entre les deux approches ?

**Lire des enregistrements :**
- Récupère une tâche par son id avec `find`
- Récupère la première et la dernière avec `first` et `last`
- Liste toutes les tâches avec `all`
- Cherche des tâches par titre avec `where`

**Mettre à jour :**
- Modifie un attribut puis `save`
- Modifie directement avec `update`

**Supprimer :**
- Supprime une tâche avec `destroy`

**Compter et vérifier :**
- `Task.count`
- `task.persisted?`
- `task.new_record?`

> Doc : [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html)
> Doc : [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html)
