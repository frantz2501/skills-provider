---
name: pdf-extraction
description: >-
  Extract text, tables, and metadata from PDF files. Use when the user wants
  to read, extract, convert, or OCR a PDF (to text, markdown, CSV, or
  structured data). Triggers on "extrais ce PDF", "PDF vers markdown",
  "lignes de ce tableau PDF", "OCR ce scan".
license: MIT
user-invocable: true
allowed-tools:
  - bash
---

# Extraction PDF

## 0. Choisir la méthode

| Situation | Méthode |
|---|---|
| PDF natif (couche texte) | `pdftotext` ou PyMuPDF (rapide) |
| Tableaux précis | `pdfplumber` (ou `camelot` si lignes de quadrillage) |
| Scan / image (pas de couche texte) | OCR : PyMuPDF + Tesseract |
| Métadonnées seulement | `pypdf` |

Détecter la couche texte avant tout : si l'extraction rapide rend moins
d'environ 100 caractères par page, c'est un scan → passer à l'OCR.

## 1. Extraction de texte

**pdftotext (poppler)** — le plus rapide quand il est installé :

```bash
pdftotext -layout input.pdf out.txt        # conserve la mise en page
pdftotext -f 5 -l 10 input.pdf out.txt     # pages 5 à 10 seulement
```

**PyMuPDF (fitz)** — rapide, précis, page par page :

```python
import fitz  # pip install pymupdf
doc = fitz.open("input.pdf")
for i, page in enumerate(doc):
    text = page.get_text()
    (open(f"out/page-{i+1:04}.txt", "w", encoding="utf-8")
     .write(text))
```

**pdfplumber** — plus lent mais meilleur sur les mises en page complexes :

```python
import pdfplumber  # pip install pdfplumber
with pdfplumber.open("input.pdf") as pdf:
    for page in pdf.pages:
        print(page.extract_text() or "")
```

## 2. Tableaux

```python
import pdfplumber
with pdfplumber.open("input.pdf") as pdf:
    for page in pdf.pages:
        for table in page.extract_tables():
            for row in table:
                print(row)   # ou écrire en CSV
```

Pour des tableaux à quadrillage net : `camelot` (`pip install camelot-py[base]`,
mode `lattice`) ; en dernier recours `tabula`. Toujours vérifier le résultat
sur 2-3 pages avant de traiter tout le document.

## 3. OCR (scans)

```bash
# Vérifier tesseract : tesseract --version  (winget install UB-Mannheim.TesseractOCR)
pip install pymupdf pytesseract pillow
```

```python
import fitz, pytesseract
from PIL import Image
import io

doc = fitz.open("scan.pdf")
for i, page in enumerate(doc):
    pix = page.get_pixmap(dpi=300)
    img = Image.open(io.BytesIO(pix.tobytes("png")))
    text = pytesseract.image_to_string(img, lang="fra+eng")
    print(f"--- page {i+1} ---\n{text}")
```

Préciser `-fra`/`lang="fra"` pour le français. Pour les livres anciens à
caractères anciens, essayer un entraînement spécifique ou avertir l'utilisateur
des limites.

## 4. Métadonnées et structure

```python
from pypdf import PdfReader  # pip install pypdf
r = PdfReader("input.pdf")
print(len(r.pages), r.metadata)
```

## Conventions

- Sortir en markdown quand le but est un document lisible ; en CSV quand
  c'est un tableau. Conserver les sauts de page (`--- page N ---`) pour
  pouvoir citer la source.
- Ne jamais afficher plus de quelques centaines de lignes extraites :
  écrire dans un fichier et montrer un échantillon.
- Indiquer clairement à l'utilisateur si le texte extrait contient des
  artefacts OCR (liaisons brisées, chiffres douteux) et sur quelles pages.
- Vérifier les pages : `-f`/`-l` de pdftotext, ou `doc[de:à]` en Python,
  pour cibler l'extraction avant tout le document.
