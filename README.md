# skills-provider

Skills Mistral Vibe réutilisables. Chaque skill vit dans son propre
dossier `<name>/SKILL.md` (frontmatter `name` + `description`, corps en
Markdown).

## Skills fournis

| Skill | Usage |
|---|---|
| `duckdb` | SQL analytique sur fichiers (CSV, Parquet, JSON, S3/HTTPS), profilage, exports |
| `pdf-extraction` | Texte, tableaux, métadonnées et OCR de PDF (pdftotext, PyMuPDF, pdfplumber, Tesseract) |
| `kanka-mcp` | Campagnes Kanka.io via le serveur MCP `kanka-mcp` (entités, posts, sous-ressources) |
| `zotero-mcp` | Bibliographie Zotero via le serveur MCP `zotero-mcp` (items, collections, citations, exports) |
| `simplemind-mcp` | Cartes mentales SimpleMind `.smmx` via le serveur MCP `simplemind-mcp-server` |

## Installation (Mistral Vibe)

Cloner le repo puis ajouter son chemin aux `skill_paths` de
`~/.vibe/config.toml` :

```toml
skill_paths = [
    "C:/Users/tortu/git/skills-provider",
    # ... autres chemins
]
```

Le dossier du repo contient directement les dossiers de skills : le chemin
à enregistrer est la racine du repo. Les skills sont chargés au démarrage
de chaque session et se déclenchent sur leur `description`.

## Ajouter un skill

1. Créer `<name>/SKILL.md` avec le frontmatter :

```markdown
---
name: mon-skill
description: >-
  Décrire précisément quand le skill doit se déclencher — c'est ce texte
  qui sert au matching.
license: MIT
---
```

2. Corps : instructions opérationnelles (commandes, workflows, pièges).
3. Rester générique : pas de chemins machine spécifiques dans la mesure
   du possible.

## Licences

MIT.
