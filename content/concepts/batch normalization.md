# Batch Normalization

*Batch normalization (BN)* je technika normalizácie aktivácií v neurónovej sieti, ktorá stabilizuje a zrýchľuje tréning.

## Motivácia

Počas tréningu sa rozdelenie vstupov každej vrstvy mení, keď sa aktualizujú parametre predchádzajúcich vrstiev — tento jav sa nazýva *internal covariate shift*. BN ho potláča normalizáciou aktivácií per-batch.

## Algoritmus

Pre mini-batch $\mathcal{B} = \{x_1, \ldots, x_m\}$ a jednu feature dimenziu:

$$\mu_\mathcal{B} = \frac{1}{m} \sum_{i=1}^{m} x_i$$

$$\sigma_\mathcal{B}^2 = \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_\mathcal{B})^2$$

$$\hat{x}_i = \frac{x_i - \mu_\mathcal{B}}{\sqrt{\sigma_\mathcal{B}^2 + \epsilon}}$$

$$y_i = \gamma \hat{x}_i + \beta$$

kde $\gamma$ (scale) a $\beta$ (shift) sú trénovateľné parametre; $\epsilon \approx 10^{-5}$ zabraňuje deleniu nulou.

## Tréning vs. inferencia

| Fáza | $\mu$ a $\sigma$ |
|---|---|
| Tréning | Vypočítané z aktuálneho mini-batchu |
| Inferencia | Použijú sa *running statistics* (exponenciálny kĺzavý priemer cez celý tréning) |

Počas tréningu sa aktualizuje: $\mu_{run} \leftarrow (1-\alpha)\mu_{run} + \alpha \mu_\mathcal{B}$, typicky $\alpha = 0{,}1$.

## Umiestnenie v sieti

Typicky: `Conv → BN → ReLU` (BN pred aktiváciou), hoci niektoré architektúry radšej `Conv → ReLU → BN`.

## Výhody

- Umožňuje vyšší learning rate (tréning konverguje rýchlejšie)
- Redukuje citlivosť na inicializáciu váh
- Pôsobí ako regularizátor → menej nutný dropout
- Zmierňuje problém miznúceho gradientu (*vanishing gradient*)

## Obmedzenia

- Vyžaduje dostatočne veľký batch (m ≥ 16); pri malom batchi je $\mu_\mathcal{B}$ zašumená
- Pri sekvenčných dátach (NLP, RNN) je batch statistics nonstacionárny → používa sa LayerNorm alebo InstanceNorm namiesto BN

## Varianty

| Varianta | Normalizuje po | Použitie |
|---|---|---|
| Batch Norm | Batch dimenzii | CNN klasifikácia |
| Layer Norm | Feature dimenzii | Transformer, RNN |
| Instance Norm | Priestorových dimenziách | Style transfer |
| Group Norm | Skupinách kanálov | Malé batche, object detection |

## Súvisiace pojmy

[[05 Konvolučné neurónové siete — tréning a regularizácia]] · [[L1 and L2 regularization]] · [[skip connections]]
