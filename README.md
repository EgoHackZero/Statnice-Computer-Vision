# Počítačové videnie — Statnice: Knowledge Graph

Interaktívna báza znalostí na prípravu na štátnu skúšku z predmetu **Pokročilé metódy počítačového videnia**.

Autor: **Nepokrytyi Yehor**

---

## O projekte

Tento knowledge graph pokrýva všetkých 10 okruhov štátnicovej otázky z počítačového videnia. Každá téma obsahuje:

- Komplexnú odpoveď pokrývajúcu každý čiastkový bod otázky
- Matematické odvodenia a vzorce (KaTeX rendering)
- Porovnania alternatív a diskusiu kompromisov
- Konkrétne príklady nasadenia v priemysle
- Sekciu "Spýtaj sa sám seba" na sebahodnotenie

Graf znalostí prepája všetky pojmy cez wikilinks, čo umožňuje navigáciu od konceptu ku konceptu.

## Témy

| # | Téma |
|---|---|
| 01 | SIFT — scale space a detekcia kľúčových bodov |
| 02 | SIFT — deskriptor a Generalized Hough Transform |
| 03 | Viola-Jones framework (Haar, AdaBoost, cascade) |
| 04 | SURF — Speeded-Up Robust Features |
| 05 | CNN — tréning a regularizácia |
| 06 | Sémantická segmentácia a U-Net |
| 07 | 2.5D, 3D a volumetrické reprezentácie |
| 08 | Kalibrácia stereo systému a rektifikácia |
| 09 | Korešpondenčný problém, disparita a hĺbka |
| 10 | Structure-from-Motion a Multi-View Stereo |
| + | Aplikácie v praxi (prierezová stránka) |

## Technológie

- **[Quartz v4](https://quartz.jzhao.xyz/)** — statický generátor stránok pre digitálne záhrady
- **Node.js ≥ 22**, npm ≥ 10
- Markdown + KaTeX matematika + Obsidian-style wikilinks

## Lokálne spustenie

```bash
npm install
npx quartz build --serve
# Otvor http://localhost:8080
```

## Štruktúra obsahu

```
content/
├── index.md                          # Domovská stránka
├── 01 SIFT — scale space ...md       # Témy (01–10)
├── ...
├── Aplikácie v praxi.md              # Prierezové aplikácie
└── concepts/                         # Konceptuálne sub-noty (~27 pojmov)
    ├── bundle adjustment.md
    ├── epipolar geometry.md
    └── ...
```

## Licencia

Obsah (markdown súbory v `content/`) © 2026 Nepokrytyi Yehor.  
Quartz framework je distribuovaný pod [MIT licenciou](LICENSE.txt).
