# Difference of Gaussians

*Difference of Gaussians (DoG)* je efektívna aproximácia *Laplaciánu Gaussiánu (LoG)*, ktorá sa používa na detekciu blob-ov v rôznych mierkach pri algoritme SIFT.

## Detaily

**Motivácia:**
Laplacián Gaussiánu $\nabla^2 G$ je vynikajúci detektor blob-ov — dosahuje maximálnu (alebo minimálnu) odozvu v strede homogénnej škvrny (blob-u). Jeho nevýhodou je výpočtová náročnosť priamej konvolúcie s LoG filtrom.

**DoG ako aproximácia LoG:**
Rozdiel dvoch Gaussiánov s rôznymi σ aproximuje LoG:
$$DoG(x, y, \sigma) = L(x, y, k\sigma) - L(x, y, \sigma) \approx (k-1)\sigma^2 \nabla^2 G$$
kde $k = 2^{1/s}$ je konštantný faktor medzi susednými mierkami v rámci oktávy.

SIFT využíva fakt, že súsedné Gaussom vyhladzované obrazy sú v pyramíde *už vypočítané* — DoG je ich jednoduchý odčet. Tak sa vyhneme explicitnej konvolúcii s LoG filtrom, čo je výpočtovo lacnejšie.

**Geometrická interpretácia:**
DoG reaguje na intenzitné blob-y — oblasti s lokálnym maximom/minimom intenzity obklopené kontrastným okolím. Podľa Lowea: *„LoG pôsobí ako detektor blob-ov detegujúci blob-y rôznych veľkostí pri zmene σ. DoG je efektívna aproximácia LoG."*

**Kde sa hľadajú extrémy:**
V DoG priestore sa hľadajú lokálne maximá a minimá v 3D objeme (x, y, σ). Každý pixel sa porovná so svojimi **26 susedmi** — 8 v tej istej mierke + 9 v mierke nad + 9 v mierke pod.

**Vlastnosti:**
- Nezávislý od absolútneho jasu (odčet eliminuje stály DC príspevok)
- Citlivý na blob-y, nie na hrany (hrany majú odozvu iba v jednom smere)
- Potrebné $s + 3$ Gaussom vyhladzovaných obrazov na oktávu pre $s$ DoG obrazov vhodných na hľadanie extremov (1 nahor + 1 nadol od každého)

## Matematika

$$DoG(x, y, \sigma) = G(x, y, k\sigma) - G(x, y, \sigma)$$

Pre $k \approx 1$ (susedné mierky):
$$DoG \approx (k-1) \cdot \sigma^2 \cdot \nabla^2 G(x, y, \sigma)$$

Čím väčšie σ, tým väčšie blob-y sú detegované — priamo mierka kľúčového bodu.

## Kde sa používa

- [[01 SIFT — scale space and keypoint detection]] — primárny použitie, základ celého SIFT detektora
- Porovnanie s SURF: SURF namiesto DoG používa determi­nant Hessian matice → rýchlejšie, podobná presnosť

## Súvisiace pojmy

[[scale space and Gaussian pyramid]], [[01 SIFT — scale space and keypoint detection]], [[Hessian matrix and blob detection]]
