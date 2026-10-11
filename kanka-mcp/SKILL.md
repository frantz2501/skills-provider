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

## Déploiement en production (VPS cyberchinois.eu)

Le serveur MCP `kanka-mcp` est déployé en permanence sur le VPS
`51.91.109.128` (`cyberchinois.eu`), utilisateur SSH `heimdal`
(authentification par clé via Pageant : utiliser
`"C:\Program Files\PuTTY\plink" -batch -agent heimdal@51.91.109.128`).
`ssh` OpenSSH ne voit pas Pageant sous Windows : passer par plink.

| Élément | Emplacement (serveur) |
|---|---|
| Code (v1.4.0) | `~/kanka-mcp/server.js` |
| Env (chmod 600) | `~/kanka-mcp/kanka-mcp.env` |
| Unité systemd | `~/.config/systemd/user/kanka-mcp.service` |

Variables d'env : `KANKA_API_TOKEN` (token API kanka.io),
`KANKA_DEFAULT_CAMPAIGN` (vide — passer `campaign_id` explicitement),
`KANKA_API_BASE=https://api.kanka.io/1.0`, `KANKA_MCP_BIND=127.0.0.1`,
`KANKA_MCP_PORT=3333`, `KANKA_MCP_HTTP_TOKEN` (bearer HTTP ; le lire sur
le serveur avec `grep KANKA_MCP_HTTP_TOKEN ~/kanka-mcp/kanka-mcp.env`).

Le service est un **unité systemd utilisateur** (pas de sudo nécessaire ;
linger activé, donc il survit aux déconnexions et reboots) :

```bash
systemctl --user status kanka-mcp        # état
systemctl --user restart kanka-mcp       # après édition de l'env
journalctl --user -u kanka-mcp -n 50     # logs
curl http://127.0.0.1:3333/               # health check (sur le serveur)
```

### Accès depuis le PC (tunnel SSH, option A du DEPLOY.md)

Le serveur bind sur 127.0.0.1 uniquement. Depuis le PC (Pageant chargé) :

```
"C:\Program Files\PuTTY\plink" -batch -agent -N -L 3333:127.0.0.1:3333 heimdal@51.91.109.128
```

Le client MCP est déjà configuré dans `~/.vibe/config.toml`
(`[[mcp_servers]]` kanka, transport http, `http://127.0.0.1:3333` +
bearer `Authorization`). Si un appel MCP échoue avec « connection
refused », le tunnel est tombé : le relancer. Vérification rapide :

```bash
curl -H "Authorization: Bearer <token-http>" -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' http://127.0.0.1:3333/
```

### Campagnes disponibles (ids, octobre 2026)

Qin - les chroniques du clan du Panda céleste (8498), Shade (31309),
Pavillon noir (34813), Anneau unique (38283), Shadowrun (38889),
Eclipse phase (39208), Qin Shaoling & Wudang (41777), Northmen (51901),
Mars : Prémices aux guerres stellaires (37119), Joan of Arc (66971),
The Expanse (83436), Nephilim (97596), Codex martialis (164060),
Imperator (178925), Shadowrun - NMA (185932). Utiliser
`kanka_list_campaigns` pour une liste à jour.

### État de campagne synchronisé en local

La campagne **Shade (31309)** a son relevé intégré dans l'atelier JDR
`D:\git\shade\kb\campagne\` (chronique, PJ, acteurs, scénarios, règles
maison), indexé par le RAG de l'atelier (catégorie `campagne`). Pour toute
production de contenu Shade, lire d'abord ces synthèses plutôt que de
re-parcourir Kanka ; ne resynchroniser Kanka que si elles datent
(procédure dans `D:\git\shade\kb\index.md`).

## Si le serveur n'est pas connecté

Fallback stdio local (zéro-dépendance, Node >= 18) :
`git clone https://github.com/frantz2501/kanka-mcp && node server.js`
(stdio) ou `node server.js --http` (HTTP sur 127.0.0.1:3333).
Config dans `kanka-mcp.json` : `apiToken`, `defaultCampaign`, `apiBase`
(https://api.kanka.io/1.0). Token : profil > API sur kanka.io.
Le déploiement complet est documenté dans `DEPLOY.md` du repo
(`/opt` + service système via `sudo bash install.sh`, ou variante
utilisateur systemd décrite ci-dessus).
