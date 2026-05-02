# Transposed Convolution

*Transponovaná konvolúcia* (niekedy nesprávne nazývaná *dekonvolúcia*) je naučiteľná operácia na zväčšenie priestorového rozlíšenia feature máp v dekodéri.

## Motivácia

Štandardná konvolúcia so stride > 1 alebo pooling znižuje rozlíšenie. V encoder-decoder architektúrach (U-Net, segmentačné siete) potrebujeme rozlíšenie opäť zvýšiť — a naučiť sieti, ako to robiť optimálne.

## Princíp (stride 2)

1. Vkladáme nulové riadky/stĺpce **medzi** vstupné pixely (stride − 1 núl medzi každé dva susedné pixely)
2. Aplicujeme štandardnú konvolúciu s filtrom $k \times k$

**Výstupná veľkosť:**
$$\text{out} = (\text{in} - 1) \cdot s - 2p + k$$

Pre in=7, s=2, p=1, k=2: out = $(7-1)\cdot 2 - 2 + 2 = 12$

## Naučiteľné parametre

Narozdiel od bilineárneho upsamplings sú koeficienty filtra trénovateľné → sieť sa naučí optimálne upsampling pre svoju úlohu. Toto je kľúčová výhoda oproti pevným (non-parametrickým) metódam.

## Checkerboard artefakty

Pri veľkosti kernela $k$ nedeliteľnej strideom $s$ (napr. $k=3, s=2$):

- Niektoré výstupné pixely dostávajú príspevky od viac vstupných pixelov ako iné
- Výsledok: nerovnomerné pokrytie → viditeľný šachovnicový vzor (*checkerboard artifacts*)

**Riešenie (Odena et al., Distill 2016):**
Nahradiť transponovanú konvolúciu kombináciou:
1. **Bilinear upsampling** (pevná interpolácia, bez artefaktov)
2. **Konvolúcia 3×3** (naučiteľné spresenie)

## Vzťah ku konvolúcii

Transponovaná konvolúcia je *adjoint* (nie inverzná) ku konvolúcii. Ak konvolúcia $y = Cx$ (maticový zápis), transponovaná konvolúcia počíta $x' = C^T y'$ — odtiaľ názov.

**Nie je to dekonvolúcia** — neobnovuje pôvodné hodnoty pixelov, len zväčšuje priestorové rozlíšenie.

## Použitie v PyTorch

```python
import torch.nn as nn

# Transponovaná konvolúcia: 256 → 128 kanálov, upsampling 2×
nn.ConvTranspose2d(256, 128, kernel_size=2, stride=2)

# Alternatíva bez artefaktov:
nn.Sequential(
    nn.Upsample(scale_factor=2, mode='bilinear', align_corners=True),
    nn.Conv2d(256, 128, kernel_size=3, padding=1)
)
```

## Súvisiace pojmy

[[06 Sémantická segmentácia a U-Net]] · [[unpooling]] · [[skip connections]]
