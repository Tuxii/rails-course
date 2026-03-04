# Step 10 - Turbo Drive

## Objectifs

- Comprendre ce que Turbo Drive fait par défaut
- Observer la navigation sans rechargement complet
- Découvrir le prefetch
- Utiliser `data-turbo-confirm` et `data-turbo-method`

## 10.1 Observer Turbo Drive

Turbo Drive est actif par défaut dans toute app Rails. Il intercepte les clics sur les liens et les soumissions de formulaires pour remplacer le contenu de la page sans rechargement complet.

Ouvre l'onglet Network de l'inspecteur (filtre sur "Doc" ou "Fetch/XHR"). Navigue entre les pages de l'app et observe :

- Pas de rechargement complet (le compteur de requêtes dans l'onglet ne se réinitialise pas)
- Les requêtes sont de type `fetch`, pas des navigations classiques
- Le `<body>` est remplacé, mais le `<head>` (CSS, JS) n'est pas rechargé

> Doc : [Turbo Drive](https://turbo.hotwired.dev/handbook/drive)

## 10.2 Désactiver Turbo Drive

Pour bien comprendre ce que Turbo Drive apporte, désactive-le temporairement.

Il y a deux façons de le faire :

- **Globalement en JS** : dans `application.js`, ajoute `Turbo.session.drive = false`. Ça désactive Turbo Drive pour toute l'application.
- **Localement en HTML** : ajoute `data-turbo="false"` sur un élément (lien, formulaire, ou conteneur) pour désactiver Turbo Drive uniquement pour cet élément et ses enfants.

Teste la désactivation globale via JS, navigue dans l'app et observe la différence dans l'onglet Network : les pages se rechargent complètement.

Retire la ligne quand tu as fini.

## 10.3 Prefetch

Turbo Drive peut précharger les pages au survol d'un lien, avant même que l'utilisateur clique. Ça rend la navigation quasi instantanée.

Observe dans l'onglet Network : au survol d'un lien, une requête fetch est envoyée avant le clic.

Tu peux contrôler ce comportement avec `data-turbo-prefetch` :

- `data-turbo-prefetch="false"` sur un lien pour le désactiver
- La config globale se fait via `<meta name="turbo-prefetch" content="false">` dans le `<head>`

Teste les deux en observant l'onglet Network.

> Doc : [Prefetch](https://turbo.hotwired.dev/handbook/drive#prefetching-links-on-hover)

## 10.4 data-turbo-confirm

Le `data-turbo-confirm` affiche une boîte de dialogue de confirmation avant d'exécuter une action.

Vérifie que le bouton "Destroy" sur les tâches et les projets utilise bien `data: { turbo_confirm: "..." }`. Si ce n'est pas le cas, ajoute-le.

## 10.5 data-turbo-method

`data-turbo-method` permet de transformer un lien `<a>` en requête PATCH, PUT ou DELETE sans avoir besoin d'un formulaire (`button_to`).

Dans la vue index des tâches, remplace un des `button_to` "Destroy" par un `link_to` avec `data: { turbo_method: :delete, turbo_confirm: "Supprimer cette tâche ?" }`.

Observe la différence dans le HTML généré (un `<a>` au lieu d'un `<form>`).

> Note : `button_to` reste la méthode recommandée. `data-turbo-method` est utile quand on veut un lien visuel plutôt qu'un bouton.
