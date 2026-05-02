# Train-Validation-Test Split

Rozdelenie datasetu na tri disjunktné podmnožiny je nevyhnutné pre správne vyhodnotenie a výber modelov v strojovom učení.

## Tri podmnožiny

| Podmnožina | Typický podiel | Účel |
|---|---|---|
| **Train** (tréningová) | 60–80 % | Tréning parametrov modelu (váhy) |
| **Validation** (validačná) | 10–20 % | Výber hyperparametrov, early stopping, porovnanie architektúr |
| **Test** (testovacia) | 10–20 % | Finálne, **jednorazové** vyhodnotenie — simuluje nasadenie |

## Prečo tri, nie dve?

Ak by sme ladili hyperparametre na testovacej sade, *information leakage* by spôsobil optimisticky skreslené výsledky — model by bol implicitne naučený na testovacej sade. Validačná sada slúži ako proxy testu počas vývoja; testovacia sada je vyhradená pre konečné hodnotenie.

**Pravidlo:** testovacia sada sa použije **raz**, po uzavretí všetkých rozhodnutí o architektúre a hyperparametroch.

## Stratified Splitting

Pre nevyvážené datasety (napr. 90 % normálne / 10 % choré snímky) musí každá podmnožina zachovať **rovnakú distribúciu tried** ako celý dataset:
```python
from sklearn.model_selection import train_test_split
X_train, X_temp, y_train, y_temp = train_test_split(
    X, y, test_size=0.3, stratify=y, random_state=42
)
X_val, X_test, y_val, y_test = train_test_split(
    X_temp, y_temp, test_size=0.5, stratify=y_temp
)
```

## Cross-Validation (k-fold)

Keď je dataset malý, k-fold CV umožňuje lepší odhad generalizácie:
1. Rozdeľ dáta na $k$ záhybov (folds), typicky $k = 5$ alebo $10$
2. Pre každý fold $i$: trénuj na ostatných $k-1$ foldoch, validuj na folde $i$
3. Finálna metrika = priemer cez všetky $k$ validácií

**Nevýhoda:** $k$-násobná tréningová záťaž. Pre veľké datasety stačí jednoduchý split.

## Temporálne dáta

Pre časové série (video, medical monitoring) **nemiešaj** záznamy z rôznych časových období — trénuj na starších dátach, testuj na novších:
```
[--- train ---][-- val --][test]  ← správne
[--- train ---][test][--- val ---]  ← nesprávne (budúcnosť v tréning)
```

## Súvisiace pojmy

[[05 Konvolučné neurónové siete — tréning a regularizácia]] · [[L1 and L2 regularization]] · [[data normalization and class imbalance]]
