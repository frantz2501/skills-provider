---
name: kanka-mcp
description: >-
  Use the Kanka MCP server to manage a Kanka.io tabletop RPG campaign through
  its tools — search entities, read and write characters, locations, notes,
  journals, events, creatures, abilities, tags, posts, attributes,
  relationships, inventory. Use when the user mentions Kanka, a campaign,
  a worldbuilding entity, or wants to create/update/delete Kanka content.
license: MIT
user-invocable: true
---

# Kanka MCP (kanka-mcp)

Serveur MCP pour l'API [Kanka.io](https://kanka.io) — gestionnaire de
campagnes de JDR. Ce skill suppose que le serveur `kanka-mcp` est connecté
comme MCP (stdio ou HTTP sur `http://127.0.0.1:3333`).
Source : https://github.com/frantz2501/kanka-mcp

## Outils

| Outil | Description |
|---|---|
| `kanka_search` | Recherche globale (paramètre `q`) |
| `kanka_get_entity` | Entité par `entity_id` + `entity_type` |
| `kanka_list_entities` | Liste/pagination d'un module |
| `kanka_create_entity` | Création — le texte est du markdown converti en HTML |
| `kanka_update_entity` | Mise à jour partielle |
| `kanka_delete_entity` | Suppression (`force` possible) |
| `kanka_list_posts` / `kanka_get_post` | Posts d'une entité (entrées de journal, etc.) |
| `kanka_create_post` / `kanka_update_post` / `kanka_delete_post` | CRUD posts |
| `kanka_list_sub` / `kanka_create_sub` / `kanka_update_sub` / `kanka_delete_sub` | Sous-ressources : attributes, relationships, inventory, ... |

Modules (entity types) : characters, locations, families, organisations,
items, notes, events, calendars, timelines, creatures, races, quests, maps,
journals, abilities, tags, conversations, dice_rolls.

## Règles d'usage

- **Toujours chercher avant de créer** : `kanka_search` ou
  `kanka_list_entities` pour éviter les doublons.
- **Écriture en markdown** : fournir le texte descriptif en markdown ; le
  serveur le convertit en HTML à la Kanka. Les mentions `[entity:123]`
  sont préservées — les utiliser pour lier des entités existantes.
- **Campagne** : `campaign_id` optionnel si une campagne par défaut est
  configurée côté serveur ; sinon la passer explicitement.
- **Rate limit Kanka** : 90 requêtes/min — éviter les boucles serrées,
  préférer un `list` paginé ciblé à des `get` en rafale.
- Ne pas supprimer sans confirmation explicite de l'utilisateur ;
  `kanka_delete_entity` est définitif (sauf corbeille Kanka).

## Workflow type

1. Explorer : `kanka_search` sur le nom clé, ou `kanka_list_entities`
   du module pertinent.
2. Lire : `kanka_get_entity` (avec `kanka_list_posts` pour le contenu).
3. Écrire : `kanka_create_entity` / `kanka_update_entity` en markdown ;
   récupérer l'ID créé pour les opérations suivantes (posts, sous-ressources).
4. Vérifier : relire l'entité modifiée avant d'annoncer la réussite.

## Si le serveur n'est pas connecté

Le serveur est zéro-dépendance (Node >= 18) :
`git clone https://github.com/frantz2501/kanka-mcp && node server.js`
(stdio) ou `node server.js --local` (app HTTP sur 127.0.0.1:3333).
Config dans `kanka-mcp.json` : `apiToken`, `defaultCampaign`, `apiBase`
(https://api.kanka.io/1.0). Token : profil > API sur kanka.io.
