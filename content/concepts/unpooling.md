# Unpooling

*Unpooling* je súhrnný názov pre techniky zväčšovania priestorového rozlíšenia feature máp v dekodérových častiach neurónových sietí (napr. U-Net decoder path).

## Prehľad techník

### 1. Nearest-Neighbor Upsampling

Každý pixel sa duplikuje do bloku $s \times s$ susedných pixelov:

```
Vstup:  [a b]    Výstup (2×): [a a b b]
        [c d]                  [a a b b]
                               [c c d d]
                               [c c d d]
```

- Žiadne trénovateľné parametre
- Rýchle, blokovitý vizuálny výsledok
- Zvyčajne nasleduje konvolúcia 3×3 na zjemnenie

### 2. Bilineárny Upsampling

Interpoluje hodnoty medzi pixelmi pomocou váhovaného priemeru štyroch susedov. Plynulejší výsledok ako nearest-neighbor.

$$f(x,y) = \frac{1}{(x_2-x_1)(y_2-y_1)}\left[f_{11}(x_2-x)(y_2-y) + f_{21}(x-x_1)(y_2-y) + \ldots\right]$$

- Bez artefaktov
- Kombinuje sa s konvolúciou 3×3 → *odporúčaná alternatíva* transponovanej konvolúcie

### 3. Transponovaná konvolúcia

Pozri [[transposed convolution]]. Naučiteľné parametre, ale riziko checkerboard artefaktov.

### 4. Max-Unpooling (SegNet)

Počas max-pooling fázy sa zapamätajú *argmax switches* — pozície maxím:

```
Pooling:
[9 3]           Max = 9, pozícia (0,0)
[2 7]     →     Max = 7, pozícia (1,1)

Unpooling (s uloženými pozíciami):
[9 0 0 0]
[0 7 0 0]
```

- Zachováva presné priestorové polohy detailov
- Riedky výstup (mnoho núl) → vyžaduje dodatočnú konvolúciu
- Vyššia pamäťová náročnosť (ukladáme argmax masky)

## Porovnanie

| Metóda | Param. | Artefakty | Pamäť | Presnosť |
|---|---|---|---|---|
| Nearest-neighbor | Nie | Blokový | Nízka | Nízka |
| Bilinear + Conv | Áno (conv) | Nie | Nízka | Vysoká |
| Transponovaná konv | Áno | Checkerboard | Nízka | Vysoká |
| Max-unpooling | Nie | Nie | Vysoká | Stredná |

## Kde sa používa

- **U-Net decoder:** typicky transponovaná konvolúcia alebo bilinear+conv
- **SegNet:** max-unpooling (zrkadlí SegNet encoder s max-pooling)
- **DeepLab:** bilineárny upsampling z poslednej feature mapy (plytký decoder)
- **GAN generator:** transponovaná konvolúcia (DCGAN)

## Súvisiace pojmy

[[06 Sémantická segmentácia a U-Net]] · [[transposed convolution]] · [[skip connections]]
