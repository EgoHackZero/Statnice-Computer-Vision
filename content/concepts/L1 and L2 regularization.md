# L1 and L2 Regularization

*Regularizácia* je technika, ktorá zabraňuje *overfitting* (prílišnému prispôsobeniu tréningovým dátam) tým, že pridáva penalizačný člen k stratovej funkcii.

## L2 Regularizácia (Weight Decay, Ridge)

Pridáva súčet štvorcov všetkých váh:

$$\mathcal{L}_{L2} = \mathcal{L}_{data} + \frac{\lambda}{2} \sum_i w_i^2$$

**Efekt:** Gradient update: $w_i \leftarrow w_i(1 - \lambda \eta) - \eta \nabla_{w_i}\mathcal{L}_{data}$

Váhy sú penalizované exponenciálne → veľké váhy sú silne potlačené, malé len jemne. Výsledok: váhy sú malé, ale zriedka presne nula — *hladké** riešenie.

## L1 Regularizácia (Lasso)

Pridáva súčet absolútnych hodnôt váh:

$$\mathcal{L}_{L1} = \mathcal{L}_{data} + \lambda \sum_i |w_i|$$

**Efekt:** Gradient obsahuje $\lambda \cdot \text{sign}(w_i)$ → konštantná sila tlačí váhy k nule. Výsledok: mnohé váhy sú presne nula → *riedka (sparse) reprezentácia*.

## Porovnanie

| | L1 | L2 |
|---|---|---|
| Penalizácia | $\lambda \|w\|_1$ | $\frac{\lambda}{2}\|w\|_2^2$ |
| Riešenie | Riedke (sparse) | Husté, malé |
| Výber príznakov | Implicitný (nulové váhy = odstránené) | Nie |
| Konvexnosť | Áno, ale nie diferencovateľné v 0 | Áno, hladké |
| Výpočet gradientu | `sign(w)` | `w` |
| Typické nasadenie | Feature selection, LASSO regresiu | Deep learning, Ridge regression |
| Alias | LASSO | Weight decay, Ridge |

## Elastic Net

Kombinácia L1 + L2:
$$\mathcal{L} = \mathcal{L}_{data} + \lambda_1 \|w\|_1 + \lambda_2 \|w\|_2^2$$

## Dropout ako regularizácia

V deep learning sa Dropout (náhodné nulovanie aktivácií s pravdepodobnosťou $p$ počas tréningu) správa ako implicitná regularizácia — vytvára ensemble modelov.

## Výber $\lambda$

- Príliš malá $\lambda$ → nedostatočná regularizácia → overfitting
- Príliš veľká $\lambda$ → underfitting → model ignoruje dáta
- Optimálna hodnota sa hľadá cross-validáciou na validačnej sade

## Súvisiace pojmy

[[05 Konvolučné neurónové siete — tréning a regularizácia]] · [[batch normalization]] · [[train-validation-test split]]
