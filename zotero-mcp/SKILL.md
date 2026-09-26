---
name: zotero-mcp
description: >-
  Use the Zotero MCP server to search, read, create, update, export, and cite
  bibliographic references — items, collections, tags, notes, formatted
  bibliographies (APA, Chicago...), BibTeX/RIS/CSL exports. Use when the user
  mentions Zotero, a bibliography, a reference/citation, a DOI import, or a
  reading list.
license: MIT
user-invocable: true
---

# Zotero MCP (zotero-mcp)

Serveur MCP pour l'API Web Zotero v3 (et l'API locale du client desktop).
Ce skill suppose que le serveur `zotero-mcp` est connecté comme MCP
(stdio ou HTTP sur `http://127.0.0.1:3334`).
Source : https://github.com/frantz2501/zotero-mcp

## Outils

| Outil | Description |
|---|---|
| `zotero_health` | Connectivité + privilèges de la clé (userID, accès par bibliothèque) |
| `zotero_search_items` | Recherche rapide (`q`, `qmode`, `tag`, `itemType`, booléens) |
| `zotero_list_items` | Items (tous, top, corbeille, par collection), tri/pagination, `all_pages` |
| `zotero_get_item` | Un item par clé (+ `include` bib/citation) |
| `zotero_list_item_children` | Enfants d'un item (notes, pièces jointes) |
| `zotero_list_item_tags` | Tags d'un item |
| `zotero_list_collections` / `zotero_get_collection` | Collections |
| `zotero_list_tags` | Tags de la bibliothèque |
| `zotero_list_searches` | Recherches sauvegardées |
| `zotero_item_template` | Gabarit JSON éditable pour un type d'item |
| `zotero_bibliography` | Bibliographie XHTML (CSL style, locale) d'une collection ou de clés |
| `zotero_export_items` | Export BibTeX, RIS, CSL JSON, CSV, TEI... |
| `zotero_create_items` | Création de 1 à 50 items |
| `zotero_update_item` | PATCH d'un item (champs modifiés seulement) |
| `zotero_delete_items` | Suppression (1 à 50) |
| `zotero_create_collections` / `zotero_delete_collections` | Collections |
| `zotero_delete_tags` | Suppression de tags dans toute la bibliothèque |

Bibliothèques de groupe : passer `group_id` à n'importe quel outil.
Sans clé API : lecture des bibliothèques publiques uniquement
(passer `user_id`/`group_id`).

## Règles d'usage

- **Vérifier d'abord** : `zotero_health` (clé, userID, droits d'écriture).
- **Recherche avant création** : `zotero_search_items` par titre/auteur/DOI
  pour éviter les doublons.
- **Création en deux temps** : `zotero_item_template` pour obtenir le
  gabarit du type voulu (book, journalArticle, thesis...), remplir les
  champs, puis `zotero_create_items` avec ce JSON.
- **Mise à jour minimale** : `zotero_update_item` ne passer que les champs
  modifiés — le verrouillage optimiste (versions Zotero) est géré par le
  serveur, ne pas retoucher les métadonnées de version.
- **Citations** : `zotero_bibliography` avec `style` (apa, chicago,
  ...) et `locale` (fr-FR) plutôt que de formater à la main ;
  `zotero_export_items` en `bibtex` pour du LaTeX.
- Ne jamais supprimer (`zotero_delete_items`, `zotero_delete_collections`,
  `zotero_delete_tags`) sans confirmation explicite de l'utilisateur.

## Workflow type

1. `zotero_list_collections` ou `zotero_search_items` pour situer.
2. `zotero_get_item` (+ `zotero_list_item_children` pour les notes
   rattachées) pour lire.
3. Créer/mettre à jour via gabarit + create/update.
4. Générer la bibliographie ou l'export demandé.

## Si le serveur n'est pas connecté

Zéro-dépendance (Node >= 18) :
`git clone https://github.com/frantz2501/zotero-mcp && node server.js`.
Config `zotero-mcp.json` : `apiKey` (https://www.zotero.org/settings/keys),
`userId` (numérique), `apiBase` (`https://api.zotero.org` ou
`http://localhost:23119/api` pour l'API locale du client desktop Zotero).
