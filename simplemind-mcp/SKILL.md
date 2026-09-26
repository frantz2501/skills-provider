---
name: simplemind-mcp
description: >-
  Use the SimpleMind MCP server to create and edit SimpleMind mind maps
  (.smmx files) — list maps, read, add/update/move/delete topics with notes,
  dates, checkboxes, links, and layouts. Use when the user mentions SimpleMind,
  a mind map / carte mentale, .smmx files, or wants to brainstorm into a
  structured outline.
license: MIT
user-invocable: true
---

# SimpleMind MCP (simplemind-mcp)

Serveur MCP pour créer et modifier des cartes mentales SimpleMind. Ce skill
suppose que le serveur `simplemind-mcp-server` est connecté comme MCP (stdio).
Serveur de référence (TypeScript, SDK MCP officiel) :
https://github.com/kharabiner/simplemind-mcp-server — variante locale :
`simplemind-mcp-server` (dossier du même nom) avec un CLI `cli.mjs`
équivalent et un mode Docker si Node n'est pas installé.

## Format de fichier

Un `.smmx` est une archive ZIP contenant `document/mindmap.xml` : topics à
plat avec attribut `parent` (racine : `parent="-1"`). Ne jamais éditer le ZIP
à la main si le serveur est disponible.

## Outils

| Outil | Description |
|---|---|
| `list_mindmaps` | Liste les `.smmx` du dossier configuré |
| `create_mindmap` | Nouvelle carte (`title`, `layout` optionnel) → nom de fichier |
| `read_mindmap` | Arborescence JSON avec les IDs de topics |
| `add_topic` | Ajoute un topic (`text`, `parent_id` optionnel = racine, `note`, `date`, `checkbox`, `link`, `layout`) → **ID du topic** |
| `update_topic` | Modifie un topic par `topic_id` (champs optionnels) |
| `move_topic` | Déplace un topic et son sous-arbre (`topic_id`, `new_parent_id`) |
| `delete_topic` | Supprime un topic et son sous-arbre |

Ressources MCP : `simplemind://mindmaps/list`,
`simplemind://mindmap/<fichier>`.

## Règles d'usage

- **Capturer les IDs** : `add_topic` renvoie l'ID du topic créé — le
  conserver pour les ajouts enfants (`parent_id`), mises à jour,
  déplacements. Pour une construction en profondeur, procéder branche par
  branche.
- **Dates au format `DD-MM-YYYY`** uniquement (ex. `15-03-2026`).
- **Layouts** : `free-form`, `horizontal`, `vertical`, `list`, `top-down`,
  `linear-down`, `radial`, `matrix`.
- **Ne jamais supprimer le topic racine** (interdit par le serveur) et ne
  pas supprimer de topic sans confirmation si la carte préexiste.
- **Pas d'édition simultanée** : ne pas modifier une carte ouverte dans
  l'application SimpleMind en même temps (conflit de réécriture).
- Après une série d'opérations, relire la carte (`read_mindmap`) et
  montrer l'arborescence obtenue à l'utilisateur.

## Workflow type

1. `list_mindmaps` — voir ce qui existe déjà.
2. `create_mindmap` ou `read_mindmap` sur la carte cible.
3. `add_topic` en cascade (récupérer chaque ID), éventuellement
  `update_topic` / `move_topic` pour réorganiser.
4. `read_mindmap` final et restitution en arborescence lisible.

## Si le serveur n'est pas connecté

Le serveur de référence : `npm install && npm run build && node dist/index.js`
(stdio), dossier des cartes via `SIMPLEMIND_MINDMAPS_PATH` (macOS iCloud :
`SIMPLEMIND_ICLOUD_PATH`). Variante locale avec CLI sans client MCP :
`node cli.mjs list|create|read|outline|add|update|move|delete`.
