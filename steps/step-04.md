# Step 04 - CRUD Tasks

## Objectifs

- Utiliser le scaffold pour générer un CRUD complet
- Comprendre le code généré (controller, vues, routes)
- Adapter les vues avec des selects pour les enums et des badges Bootstrap
- Afficher les erreurs de validation dans le formulaire
- Créer des helpers

## 4.1 Scaffold

Le modèle Task existe déjà. Utilise le scaffold avec `--skip` pour générer le controller, les vues et les routes sans toucher aux fichiers déjà existants:

```bash
bin/rails generate scaffold Task title status:integer priority:integer due_on:date --skip --skip-collision-check
```

Observe tous les fichiers générés. Lance le serveur et teste le CRUD dans le navigateur.

> Doc : [Rails Generators](https://guides.rubyonrails.org/command_line.html#bin-rails-generate)

## 4.2 Comprendre le code généré

Prends le temps de lire le controller généré. Repère :

- Les 7 actions CRUD et à quelle route chacune correspond (`bin/rails routes`)
- Le `before_action` et la méthode `set_task`
- Les **strong parameters** dans `task_params`
- Le `redirect_to` avec `notice` après create/update/destroy
- Le `render :new` / `render :edit` quand la validation échoue

> Doc : [Action Controller Overview](https://guides.rubyonrails.org/action_controller_overview.html)

## 4.3 Styliser le formulaire avec Bootstrap

Le scaffold génère un formulaire fonctionnel mais sans classes CSS. Ajoute les classes Bootstrap pour un rendu propre :

- `form-label` sur les labels
- `form-control` sur les inputs
- `mb-3` sur les divs qui wrappent chaque champ

> Doc : [Bootstrap Forms](https://getbootstrap.com/docs/5.3/forms/overview/)

## 4.4 Adapter les selects pour les enums

Le scaffold génère des `number_field` pour les colonnes integer (status, priority). Ce n'est pas très pratique pour des enums.

Remplace-les par des `select` qui affichent les valeurs lisibles. Les enums Rails exposent un hash des valeurs via une méthode de classe sur le modèle — cherche comment l'utiliser.

> Doc : [Form Helpers](https://guides.rubyonrails.org/form_helpers.html)

## 4.5 Erreurs de validation

Soumets un formulaire invalide (titre vide ou trop court). Que se passe-t-il ?

Le scaffold génère déjà l'affichage des erreurs dans `_form.html.erb`. Stylise-le avec Bootstrap (classe `alert alert-danger` par exemple) à la place du `style="color: red"` inline.

> Doc : [Displaying Validation Errors in Views](https://guides.rubyonrails.org/active_record_validations.html#displaying-validation-errors-in-views)

## 4.6 Helpers pour les badges

Crée deux helpers dans `ApplicationHelper` pour afficher le status et la priority sous forme de **badges Bootstrap** colorés :

- Status : `bg-secondary` pour todo, `bg-warning` pour in_progress, `bg-success` pour done
- Priority : `bg-info` pour low, `bg-warning` pour medium, `bg-danger` pour high

Utilise-les dans les vues index et show à la place du texte brut.

> Doc : [Helpers](https://guides.rubyonrails.org/action_view_helpers.html)
> Doc : [Bootstrap Badges](https://getbootstrap.com/docs/5.3/components/badge/)

## 4.7 Styliser la page index

Le scaffold génère une liste simple pour l'index. Remplace-la par un **tableau Bootstrap** (`table table-striped`) avec des colonnes : Titre, Status, Priority, Échéance, et des liens d'actions (voir, modifier, supprimer).

Utilise les helpers de badges créés à l'étape précédente pour le status et la priority.

> Doc : [Bootstrap Tables](https://getbootstrap.com/docs/5.3/content/tables/)

## 4.8 Formater les dates avec I18n

Les dates s'affichent au format brut (`2026-03-10`). Rails fournit `I18n.l` (ou son raccourci `l`) pour formater les dates selon la locale.

Configure la locale française dans `config/application.rb` (`config.i18n.default_locale = :fr`), puis utilise `l(task.due_on)` dans les vues pour afficher les dates en français.

Attention : il faut que le fichier de locale FR soit disponible. Cherche la gem `rails-i18n` ou ajoute un fichier `config/locales/fr.yml`.

> Doc : [Rails Internationalization (I18n)](https://guides.rubyonrails.org/i18n.html)

## 4.9 Finitions

- Mets à jour la navbar avec un lien vers les tâches
- Mets à jour la page d'accueil avec un lien vers les tâches
