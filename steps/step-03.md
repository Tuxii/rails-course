# Step 03 - Routes, controller et layout

## Objectifs

- Comprendre le cycle requête/réponse dans Rails (route → controller → vue)
- Générer un controller
- Configurer les routes
- Personnaliser le layout avec Bootstrap

## 3.1 Un premier controller

Génère un controller `Pages` avec une action `home` :

```bash
bin/rails generate controller Pages home
```

Observe les fichiers générés. Configure la route `root` pour pointer vers cette action.

Lance le serveur et vérifie dans le navigateur.

> Doc : [Action Controller Overview](https://guides.rubyonrails.org/action_controller_overview.html)
> Doc : [Rails Routing](https://guides.rubyonrails.org/routing.html)

## 3.2 Comprendre le cycle MVC

Observe ce qui se passe quand tu visites `http://localhost:3000` :

1. La requête arrive → Rails consulte `config/routes.rb`
2. La route matche → Rails appelle le controller et l'action correspondante
3. Le controller exécute l'action → Rails rend la vue associée

Inspecte les logs du serveur pour voir ce cycle en action.

## 3.3 Personnaliser le layout

Le fichier `app/views/layouts/application.html.erb` est le layout principal. Toutes les vues sont rendues à l'intérieur via `<%= yield %>`.

Des partials sont fournies dans le dossier `ror/views/layouts/` (`_navbar`, `_flash`, `_footer`). Copie-les dans `app/views/layouts/` de ton projet.

Modifie le layout `application.html.erb` pour rendre ces partials au bon endroit dans le `<body>` avec `<%= render "layouts/navbar" %>`, etc.

Ajoute aussi un container Bootstrap autour du `<%= yield %>` pour que le contenu soit bien centré.

> Doc : [Layouts](https://guides.rubyonrails.org/layouts_and_rendering.html#structuring-layouts)
> Doc : [Partials](https://guides.rubyonrails.org/layouts_and_rendering.html#using-partials)

## 3.4 Ajouter du contenu à la page d'accueil

Dans la vue `pages/home.html.erb`, ajoute un titre et un peu de contenu pour que la page ne soit pas vide. On y ajoutera un lien vers les tâches à la prochaine étape.
