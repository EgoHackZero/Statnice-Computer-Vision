# Data Normalization and Class Imbalance

## Normalizácia vstupných dát

Neurónovej sieti sa lepšie trénuje, keď vstupné hodnoty majú konzistentný rozsah. Nekalibrované vstupy (napr. pixely 0–255 vs. iné príznaky v rozsahu 0–1) spôsobujú asymetrické gradienty a pomalú konvergenciu.

### Min-Max normalizácia

$$x_{norm} = \frac{x - x_{min}}{x_{max} - x_{min}} \in [0, 1]$$

Vhodná ak je distribúcia rovnomerná a nie sú outliere.

### Z-score štandardizácia (zero-mean, unit variance)

$$x_{std} = \frac{x - \mu}{\sigma}$$

kde $\mu$ a $\sigma$ sú vypočítané z **tréningovej sady** — rovnaké hodnoty sa potom aplikujú na val/test sadu.

**Štandard pre obrázky CNN:**
```python
# ImageNet štatistiky (per-channel mean/std):
mean = [0.485, 0.456, 0.406]   # RGB
std  = [0.229, 0.224, 0.225]
```

### Čo normalizovať a čo nie

- Vždy normalizuj: pixelové hodnoty, spojité príznaky
- Nie: one-hot kódované triedy, binárne masky
- Štatistiky vždy počítaj z *tréningovej* sady — nikdy z celého datasetu (výnimka: exploratory analysis)

---

## Nevyváženosť tried (Class Imbalance)

Väčšina reálnych datasetov má nevyvážené triedy: nemoc vs. zdravie (1:10), detekcia defektov (1:100), anomálie v autonómnom riadení (1:1000+).

### Prečo je to problém

Model minimalizujúci cross-entropy ignoruje minoritnú triedu — dosahuje 99% accuracy tým, že predikuje vždy majoritnú triedu (ale s 0% recall pre dôležitú triedu).

### Riešenia na úrovni dát (data-level)

| Metóda | Popis | Kedy |
|---|---|---|
| **Oversampling minoritnej triedy** | Kopíruj menšinové vzorky | Keď je malá absolútna veľkosť |
| **Undersampling majority** | Vymaž náhodne vzorky z veľkej triedy | Keď máš obrovský dataset |
| **SMOTE** | Generuj syntetické vzorky interpoláciou | Tabulárne dáta |
| **Data augmentation** | Augmentuj len menšinové triedy | Obrazové datasety |

### Riešenia na úrovni stratovej funkcie

**Weighted cross-entropy:** každú triedu $c$ vážime inverzne k jej frekvencii:
$$w_c = \frac{N_{total}}{C \cdot N_c}$$

**Focal loss (Lin et al. 2017 — RetinaNet):**
$$FL(p_t) = -(1 - p_t)^\gamma \log(p_t)$$

Parameter $\gamma > 0$ znižuje stratu pre ľahko klasifikovateľné vzorky → sieť sa sústredí na ťažké (menšinové) prípady.

**Dice loss:** vhodná pre segmentáciu s nevyvážením plôch → [[06 Sémantická segmentácia a U-Net\|téma 6]].

### Metriky pre nevyvážené datasety

- **Accuracy** — zavádzajúca (ignoruje minority)
- **Precision / Recall / F1** — per-class hodnotenie
- **ROC AUC** — area under ROC curve, nezávislá od thresholdu
- **PR AUC** (Average Precision) — lepšia ako ROC AUC pri veľmi nevyvážených triedach

## Súvisiace pojmy

[[05 Konvolučné neurónové siete — tréning a regularizácia]] · [[train-validation-test split]] · [[batch normalization]]
