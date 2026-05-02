# 06 Sémantická segmentácia a U-Net

> *Otázka: Sémantická segmentácia s CNN, U-Net, "unpooling" a transponovaná konvolúcia?*

## Stručný prehľad

*Sémantická segmentácia (semantic segmentation)* je úloha počítačového videnia, v ktorej každému pixelu vstupného obrazu priradíme triednú nálepku (napr. „cesta", „auto", „chodec", „pozadie"). Na rozdiel od detekcie objektov (bounding boxy) alebo inštančnej segmentácie (maska každého individuálneho objektu) je výsledkom pixelovú mapu rovnakej veľkosti ako vstupný obraz. *U-Net* (Ronneberger et al., 2015) je architektúra CNN s tvarom písmena „U", ktorá je dnes štandardom pre biomedicínsku a priemyselnú segmentáciu. Kľúčovými komponentmi sú encoder-decoder štruktúra s *preskočenými spojeniami (skip connections)* a *transponovaná konvolúcia (transposed convolution)* na upsampling.

## Čo je sémantická segmentácia

**Definícia:**
- Vstup: obraz $I \in \mathbb{R}^{H \times W \times 3}$
- Výstup: pixelová mapa tried $S \in \{1, \ldots, C\}^{H \times W}$ — každý pixel dostane jeden z $C$ tried

**Porovnanie s príbuznými úlohami:**

| Úloha | Výstup | Rozlišuje inštancie? |
|---|---|---|
| Klasifikácia | Jedna trieda pre celý obraz | Nie |
| Detekcia (object detection) | Bounding boxy + triedy | Nie (iba poloha) |
| Sémantická segmentácia | Pixelová mapa tried | Nie |
| Inštančná segmentácia | Pixelová maska per-objekt | Áno |
| Panoptická segmentácia | Kombinácia oboch | Áno |

**Kľúčová výzva:**
CNN s pooling vrstvami postupne znižujú priestorové rozlíšenie, čím strácajú presné hranice objektov. Segmentácia požaduje *presné pixelové hranice* → treba riešiť konflikt medzi sémantickým kontextom (veľké receptívne pole) a lokálnym detailom (vysoké rozlíšenie).

## FCN — Fully Convolutional Network (historický kontext)

*FCN (Fully Convolutional Network)*, Long et al. (2015), bola prvá architektúra schopná plne konvolučnej segmentácie. Nahradila *fully connected* vrstvy klasifikačných CNN (napr. VGG) konvolúciami 1×1 → výstup: feature mapa ľubovoľnej veľkosti → transponovaná konvolúcia na upsampling na pôvodnú veľkosť. FCN preukázala, že CNN môžu priamo produkovať hustý pixelový výstup bez sliding window.

## Architektúra U-Net

U-Net (Ronneberger, Fischer, Brox — MICCAI 2015) bola navrhnutá pre biomedicínsku segmentáciu, kde je dostupných málo anotovaných dát. Dosahuje výnimočné výsledky aj na malých datasetoch vďaka data augmentácii a skip connections.

![[U_net.png]]

**Encoder (zostupná cesta / downsampling path):**
- Opakuje blok: [Conv 3×3 → ReLU → Conv 3×3 → ReLU → MaxPool 2×2]
- Každý blok zdvojnásobí počet kanálov: 64 → 128 → 256 → 512
- Po 4 blokoch: bottleneck (1024 kanálov) — najnižšie priestorové rozlíšenie, najväčší sémantický kontext

**Decoder (vystupujúca cesta / upsampling path):**
- Opakuje blok: [Transposed Conv 2×2 (upsampling 2×) → concatenate s encoder feature map → Conv 3×3 → ReLU → Conv 3×3 → ReLU]
- Každý blok znižuje počet kanálov na polovicu: 512 → 256 → 128 → 64
- Výstupná vrstva: Conv 1×1 → $C$ kanálov (počet tried) → Softmax po pixeloch

**Skip connections:**
Kopírujú feature mapy z *odpovedajúcej úrovne encodera* do dekodera pred konkatenáciou. Encoder videl lokálne detaily (hrany, textúry) s vysokým rozlíšením — decoder ich dostane späť priamo, namiesto toho, aby ich musel rekonštruovať z bottlenecku. Výsledok: presné hranice segmentácie.

## Možnosti unpoolingu (*Unpooling Options*)

*Unpooling* je súhrnný názov pre techniky na zväčšenie priestorového rozlíšenia feature máp v dekoderi.

**1. Transponovaná konvolúcia (*Transposed Convolution*):**
- *Naučená* metóda upsamplings
- Funguje tak, že vkladá nuly medzi pixely vstupnej feature mapy a potom aplikuje štandardnú konvolúciu
- Stride $s > 1$ znásobí rozlíšenie $s$-násobne (napr. stride 2 → 2× väčší výstup)
- Váhy filtra sa *učia* počas tréningu → sieť si naučí optimálne upsampling pre svoju úlohu
- **Nevýhoda — checkerboard artefakty**: pri kerneli veľkosti, ktorá nie je deliteľná strideom, sa niektoré výstupné pixely pokryjú viac ako iné → viditeľné šachovnicové artefakty v výstupe
- **Riešenie (Odena et al. 2016)**: namiesto transponovanej konvolúcie použiť bilinear upsampling + normálna konvolúcia

**2. Bilineárny upsampling + konvolúcia:**
- *Bilineárny upsampling*: interpoluje hodnoty medzi pixelmi (plynulejší výsledok ako nearest-neighbor)
- Nasleduje konvolúcia → sieť sa naučí spresniť upsampling
- Bez checkerboard artefaktov
- Odporúčané ako drop-in náhrada za transponovanú konvolúciu v moderných implementáciách

**3. Nearest-Neighbor Upsampling:**
- Každý pixel vstupnej feature mapy sa duplikuje do $s \times s$ bloku
- Rýchle, žiadne parametre
- Typicky nasleduje konvolúcia 3×3 pre zjemnenie
- Vizuálne blokovitý výsledok, ale funkčný

**4. Max-Unpooling:**
- Počas *max-poolingu* sa zapamätajú pozície maxím (*argmax switches*)
- Počas unpoolingu sa hodnoty umiestnia späť na pôvodné pozície; ostatné pixely = 0
- Používa sa v SegNet architektúre
- Zachováva presné polohy detailných čŕt, ale riedky výstup vyžaduje dodatočnú konvolúciu
- Limitácia: musíme si pamätať argmax masky → vyššia pamäťová náročnosť

**Porovnanie unpooling techník:**

| Metóda | Trénovateľné? | Artefakty | Pamäť | Presnosť |
|---|---|---|---|---|
| Transponovaná konvolúcia | Áno | Checkerboard | Nízka | Vysoká (ak OK artefakty) |
| Bilinear + konvolúcia | Čiastočne | Nie | Nízka | Vysoká |
| Nearest-neighbor + konv | Čiastočne | Bloky | Nízka | Stredná |
| Max-unpooling | Nie | Nie | Vysoká (argmax) | Stredná |

## Transponovaná konvolúcia — podrobnejšie

Transponovaná konvolúcia (niekedy nesprávne nazývaná *„dekonvolúcia"*) nie je matematickou inverzou konvolúcie — nemôže obnoviť pôvodné hodnoty pixelov. Je to *konvolúcia so strideom menším ako 1* aplikovaná na feature mapu s vloženými nulami.

**Príklad (stride = 2):**
1. Medzi každé dva susedné vstupy sa vloží 1 nulový riadok/stĺpec
2. Plocha sa opaduje (padding) — obvykle `same` alebo `valid` padding
3. Aplikuje sa štandardná konvolúcia s filtrom $k \times k$

Výsledkom je výstup s rozlíšením $\approx 2\times$ väčší.

**Checkerboard artefakty:**
Vznikajú ak kernel size $k$ nie je deliteľný strideom $s$. Napríklad $k=3, s=2$: výstupné pixely na rôznych pozíciách dostávajú príspevky od rôzneho počtu vstupných pixelov → nerovnomerný výsledok → viditeľný šachovnicový vzor.

## Stratové funkcie pre segmentáciu

**Cross-entropy po pixeloch:**
$$\mathcal{L}_{CE} = -\frac{1}{HW} \sum_{i,j} \sum_{c=1}^{C} y_{ijc} \log \hat{p}_{ijc}$$
Nevýhoda: pri nevyvážených triedach (napr. malý tumor vs. veľké pozadie) model ignoruje minoritnú triedu.

**Dice loss:**
$$\mathcal{L}_{Dice} = 1 - \frac{2 \sum_i p_i g_i}{\sum_i p_i + \sum_i g_i}$$
kde $p_i$ je predikovaná pravdepodobnosť a $g_i$ je ground-truth maska. Dice loss je navrhnutá tak, aby maximalizovala *IoU* (intersection-over-union) — vhodná pre nevyvážené segmentácie.

**Kombinácia:** $\mathcal{L} = \mathcal{L}_{CE} + \lambda \cdot \mathcal{L}_{Dice}$ — štandard v praxi.

## Matematika

**U-Net encoder step:**
$$F_{l+1} = \text{MaxPool}_{2\times2}\left(\text{ReLU}\!\left(\text{Conv}_{3\times3}\!\left(\text{ReLU}\!\left(\text{Conv}_{3\times3}(F_l)\right)\right)\right)\right)$$

**Transponovaná konvolúcia (stride 2):**
$$\text{Output size} = (\text{Input size} - 1) \times \text{stride} - 2 \times \text{padding} + \text{kernel size}$$

Pre $\text{input}=56, \text{stride}=2, \text{pad}=1, \text{kernel}=2$: $\text{out} = (56-1)\cdot2 - 2 + 2 = 112$

**Dice coefficient:**
$$\text{Dice} = \frac{2|P \cap G|}{|P| + |G|}$$

## Porovnania a kompromisy

**U-Net vs. FCN:**
- FCN: jednoduchší, priamy upsampling z bottlenecku; menej presný pri hraniciach
- U-Net: skip connections zachovávajú lokálne detaily; lepší pre malé datasety

**U-Net vs. DeepLab (atrous/dilated convolution):**
- DeepLab neznižuje rozlíšenie na minimum — namiesto toho používa *dilated convolutions* (rozšírené konvolúcie) na rozšírenie receptívneho poľa bez straty rozlíšenia
- DeepLab zvyčajne lepší pre prirodzené scény (CityScapes); U-Net lepší pre biomedicínu

## Časté zlyhania a obmedzenia

- **Malé objekty v obraze**: pooling znižuje rozlíšenie → malé štruktúry sa strácajú; skip connections čiastočne pomáhajú
- **Nevyvážené triedy**: cross-entropy uprednostňuje majoritnú triedu → nutné použiť Dice/Focal loss
- **Checkerboard artefakty**: pri nesprávnej konfigurácie transponovanej konvolúcie
- **Pamäťová náročnosť**: U-Net uchováva feature mapy z celého encodera (pre skip connections) → vysoká RAM pri veľkých rozlíšeniach
- **Nedostatočná anotácia**: U-Net funguje aj s malými datasetmi (jeho silná stránka), ale potrebuje aspoň niekoľko stoviek anotovaných snímok

## Praktické využitie

1. **Medicínska segmentácia** — segmentácia nádorov (MRI, CT), orgánov (pečeň, obličky), retinálnych ciev (fundus fotografia), histológia; MONAI knižnica pre medicínske U-Net modely
2. **Autonómne vozidlá** — segmentácia cesty, pruhu, chodcov (CityScapes dataset); nasadenie na GPU (NVIDIA Jetson)
3. **Satelitné a letecké snímky** — segmentácia budov, polí, lesov (ISPRS Potsdam dataset); SpaceNet challenge
4. **Priemyslová inšpekcia** — detekcia povrchových defektov (škrabance, trhliny) na výrobných linkách
5. **AR a 3D rekonštrukcia** — segmentácia popredia/pozadia pre background removal (Zoom, Teams virtual background)

**Knižnice a frameworky:**
- PyTorch: `segmentation-models-pytorch` (SMP) — desiatky U-Net variantov
- TensorFlow: `tf.keras` s vlastným U-Net
- MONAI: lekárska segmentácia
- OpenMMLab: `mmsegmentation`

## Súvisiace pojmy

[[transposed convolution]], [[unpooling]], [[skip connections]], [[batch normalization]], [[05 Konvolučné neurónové siete — tréning a regularizácia]]

---
*Spýtaj sa sám seba:*
1. Čo je rozdiel medzi sémantickou a inštančnou segmentáciou?
2. Prečo U-Net potrebuje skip connections — čo by sa stalo bez nich?
3. Čo sú checkerboard artefakty a ako im predísť?
4. Porovnaj max-unpooling a transponovanú konvolúciu: kedy by si použil každý?
5. Prečo je Dice loss vhodnejšia ako cross-entropy pri silne nevyvážených segmentáciách (napr. malý tumor vs. veľké pozadie)?
