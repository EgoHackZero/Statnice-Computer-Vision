# Skip Connections

*Skip connections* (preskočené spojenia) sú priame cesty, ktoré prenášajú feature mapy z ranejších vrstiev siete do neskorších vrstiev, obchádzajúc niekoľko intermediate vrstiev.

## Typy skip connections

### 1. Concatenation skip connections (U-Net)

Feature mapa z encodera sa **zreťazí (concatenate)** s feature mapou zodpovedajúcej úrovne dekodera pozdĺž dimenzie kanálov:

```
Encoder level 3: [B, 256, 64, 64]
Decoder upsamp:  [B, 256, 64, 64]
Po concatenate:  [B, 512, 64, 64] → Conv → [B, 256, 64, 64]
```

Používané v: U-Net, SegNet, FCN-skip

### 2. Additive skip connections (ResNet)

Feature mapa sa **pričíta** k výstupu bloku:

$$\mathbf{y} = F(\mathbf{x}, \{W_i\}) + \mathbf{x}$$

kde $F(\mathbf{x})$ je residuálna funkcia (napr. 2× Conv-BN-ReLU). Ak rozmery nesúhlasia, použije sa projekcia (1×1 konvolúcia).

## Prečo sú dôležité

### Pre segmentáciu (U-Net)

Encoder stratí priestorové detaily poolingom → decoder musí rekonštruovať hranice objektov. Skip connections prenášajú **low-level príznaky** (hrany, textúry) priamo z encodera pred ich stratou:

- Bez skip connections: decoder musí obnovovať detaily z úzkeho bottlenecku → rozmazané hranice
- So skip connections: decoder dostane presné priestorové informácie → ostré hranice segmentácie

### Pre tréning hlbokých sietí (ResNet)

Hluboké siete (20+ vrstiev) trpia *vanishing gradient* — gradienty sa tlmia späť cez mnoho vrstiev. Additive skip connections vytvárajú *gradient highway* — gradient teče priamo cez skip path bez útlmu.

Navyše: ak by bol blok zbytočný, optimalizátor môže nastaviť $F \to 0$ → skip connections umožňujú **efektívne plytšie siete** (identity shortcuts).

## Matematické zdôvodnenie (ResNet)

Backprop cez residuálny blok:
$$\frac{\partial \mathcal{L}}{\partial \mathbf{x}} = \frac{\partial \mathcal{L}}{\partial \mathbf{y}} \cdot \frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \frac{\partial \mathcal{L}}{\partial \mathbf{y}} \left(1 + \frac{\partial F}{\partial \mathbf{x}}\right)$$

Člen $1$ zaručuje, že gradient nikdy neklesne na nulu — dokonca aj ak $\frac{\partial F}{\partial \mathbf{x}} \approx 0$.

## Použitie

| Architektúra | Typ skip | Účel |
|---|---|---|
| U-Net | Concatenation | Zachovanie priestorových detailov (segmentácia) |
| ResNet | Addition | Tréning hlbokých sietí (klasifikácia) |
| DenseNet | Concatenation (všetky predchádzajúce) | Maximálny feature reuse |
| FPN | Addition + upsampling | Multi-scale detekcia objektov |

## Súvisiace pojmy

[[06 Sémantická segmentácia a U-Net]] · [[transposed convolution]] · [[batch normalization]]
