---
name: duckdb
description: >-
  Query, transform, and export data using DuckDB (SQL on CSV, Parquet, JSON,
  Excel, SQLite, remote URLs, S3). Use when the user mentions DuckDB, wants to
  inspect/query/convert data files, profile a dataset, or run analytics SQL
  without a database server. Triggers on "duckdb", "requete SQL", "parquet",
  "analyse ce fichier", "convertis en CSV".
license: MIT
user-invocable: true
allowed-tools:
  - bash
---

# DuckDB — SQL analytique sur fichiers

DuckDB lit les fichiers directement en SQL, sans base à monter ni serveur.
Vérifier d'abord qu'il est installé : `duckdb --version`
(sinon : `winget install DuckDB.cli`, ou l'extension n'est pas requise
pour les fichiers locaux).

## Patterns principaux

**Requête ponctuelle en mémoire** (le cas le plus courant) :

```bash
duckdb :memory: -c "SELECT * FROM 'data/file.csv' LIMIT 10;"
```

**Interroger un fichier sans copie** — tout format se lit par fonction ou
par nom direct :

```sql
SELECT * FROM 'file.parquet' USING SAMPLE 1000 ROWS;   -- parquet, csv, json...
SELECT * FROM read_csv_auto('file.csv');               -- détection auto
SELECT * FROM read_json('file.json');                  -- ndjson/jsonl aussi
SELECT * FROM read_parquet('s3://bucket/path/*.parquet');  -- httpfs requis
```

**Base persistante** :

```bash
duckdb data/mydb.duckdb -c "CREATE TABLE t AS SELECT * FROM 'in.csv';"
```

**Profilage d'un dataset** :

```sql
SUMMARIZE SELECT * FROM 'file.csv';        -- stats par colonne
DESCRIBE SELECT * FROM 'file.csv';         -- types détectés
```

## Friendly SQL utile

```sql
FROM 'file.csv' SELECT * EXCLUDE (col_a, col_b);   -- FROM d'abord
SELECT * REPLACE (trim(nom) AS nom) FROM 'file.csv';
GROUP BY ALL;                                      -- agrège toutes les non-agrégées
ORDER BY ALL;
SELECT unnest(list_column) FROM t;
```

Jointure de fichiers hétérogènes (fichier CSV + parquet) directement dans
le FROM ; `read_csv(..., union_by_name=true, ignore_errors=true)` pour
concaténer des JSON NDJSON de schémas variés ; `filename=true` pour garder
la trace du fichier d'origine.

## Extensions

```sql
INSTALL httpfs;  LOAD httpfs;     -- http(s) et S3/GCS/R2/MinIO
INSTALL spatial; LOAD spatial;   -- GeoJSON, Shapefile, GeoParquet
```

Elles se téléchargent une fois puis sont mises en cache. Si l'installation
échoue (hors ligne), s'en tenir aux fichiers locaux.

## Export / conversion

```sql
COPY (SELECT * FROM t) TO 'out.parquet' (FORMAT parquet, COMPRESSION zstd);
COPY (SELECT * FROM t) TO 'out.csv' (HEADER, DELIMITER ';');
COPY (SELECT * FROM t) TO 'out.jsonl' (FORMAT newline_delimited);
COPY (SELECT * FROM t) TO 'out.xlsx' (FORMAT xlsx);   -- extension excel requise
```

En ligne de commande : `duckdb :memory: -c "..." -csv` pour une sortie CSV
machine-lisible (utile pour réinjecter le résultat dans la conversation),
`.mode markdown` pour l'affichage, `.read script.sql` pour rejouer un
fichier SQL.

## Conventions

- Toujours un timeout sur les commandes bash ; les grosses requêtes
  peuvent être longues.
- Limiter les sorties (LIMIT, `left(col, 500)`, SAMPLE) avant d'afficher.
- Pour les données volumineuses, écrire dans un fichier puis l'inspecter
  plutôt que d'afficher des milliers de lignes.
- Les chemins Windows passent aussi en style `/c/...` (Git Bash) ou
  `C:/...` (DuckDB accepte les deux).
