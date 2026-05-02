# Block Matching and SGM

Dve hlavné metódy výpočtu disparity zo stereo obrazov: lokálny *block matching* a globálne-konzistentný *Semi-Global Matching*.

## Block Matching (lokálna metóda)

### Princíp

Pre každý pixel $(u_L, v)$ v ľavom rektifikovanom obraze:
1. Vyber okno $W$ veľkosti $w \times w$ okolo pixelu
2. Porovnaj s každým oknom v pravom obraze na rovnakom riadku $v$ pre každú disparitu $d \in [d_{min}, d_{max}]$
3. Zvoľ disparitu $d^*$ s **minimálnou** matching cost

### Matching Cost funkcie

**SSD (Sum of Squared Differences):**
$$\text{SSD}(d) = \sum_{(i,j) \in W} \left(I_L(u_L+i, v+j) - I_R(u_L-d+i, v+j)\right)^2$$

**SAD (Sum of Absolute Differences):** rýchlejší, menej citlivý na outliere:
$$\text{SAD}(d) = \sum_{(i,j)\in W} |I_L(\ldots) - I_R(\ldots)|$$

**NCC (Normalized Cross-Correlation):** invariantná voči lineárnym zmenám jasu:
$$\text{NCC}(d) = \frac{\sum_{W}(I_L - \bar{I}_L)(I_R - \bar{I}_R)}{\sqrt{\sum_W(I_L-\bar{I}_L)^2 \cdot \sum_W(I_R-\bar{I}_R)^2}}$$

### Voľba veľkosti okna

| Veľkosť | Výhody | Nevýhody |
|---|---|---|
| Malé (3–5px) | Zachytáva jemné detaily | Citlivé na šum, zlé na textúreless |
| Veľké (21–51px) | Odolné voči šumu | Rozmazáva hrany disparity |

### OpenCV

```python
stereo = cv2.StereoBM_create(numDisparities=128, blockSize=15)
disparity = stereo.compute(imgL, imgR)
```

---

## Semi-Global Matching (SGM, Hirschmuller 2005/2008)

SGM kombinuje výhody lokálneho matchingu (rýchlosť) s globálnou konzistenciou (hladkosť disparity).

### Energetická funkcia

$$E(D) = \sum_p \left[C(p, D_p) + \sum_{q \in N_p} P_1 \mathbf{1}[|D_p - D_q| = 1] + \sum_{q \in N_p} P_2 \mathbf{1}[|D_p - D_q| > 1]\right]$$

- $C(p, D_p)$: lokálna matching cost pixela $p$ pri disparite $D_p$
- $P_1$: malá penalizácia za zmenu disparity o 1 pixel (hladký povrch)
- $P_2 > P_1$: veľká penalizácia za skok disparity > 1 (hrana objektu)

### Agregácia po cestách

Namiesto NP-ťažkej 2D optimalizácie SGM rieši 1D dynamické programovanie pozdĺž **8 (alebo 16) smerov**:

$$L_r(p, d) = C(p, d) + \min\!\left[L_r(p-r, d),\, L_r(p-r, d\pm1) + P_1,\, \min_{d'} L_r(p-r, d') + P_2\right]$$

Finálna cena = súčet po všetkých smeroch:
$$S(p, d) = \sum_r L_r(p, d)$$

Výsledná disparita: $D_p = \arg\min_d S(p, d)$

### Left-Right Consistency Check

1. Vypočítaj disparitu z ľavého do pravého obrazu: $D_L$
2. Vypočítaj disparitu z pravého do ľavého obrazu: $D_R$
3. Bod $(u_L, v)$ je platný iba ak: $|D_L(u_L) - D_R(u_L - D_L(u_L))| \leq 1$
4. Nekonzistentné body → oklúzia alebo chybný matching → označiť ako `invalid` → interpolovať z okolia

### OpenCV implementácia SGM

```python
stereo = cv2.StereoSGBM_create(
    minDisparity=0,
    numDisparities=128,
    blockSize=5,
    P1=8 * 3 * 5**2,   # 8 * channels * blockSize²
    P2=32 * 3 * 5**2,  # 32 * channels * blockSize²
    disp12MaxDiff=1,   # L-R consistency threshold
    uniquenessRatio=15,
    speckleWindowSize=100
)
disparity = stereo.compute(imgL_gray, imgR_gray).astype(np.float32) / 16.0
```

## Porovnanie BM vs. SGM

| | Block Matching | SGM |
|---|---|---|
| Textúreless oblasti | Zlé | Dobré (globálna hladkosť) |
| Hrany disparity | Rozmazané | Ostré |
| Rýchlosť | Rýchlejšie | Pomalšie |
| Pamäť | Nízka | Stredná |
| Implementácia | `cv2.StereoBM` | `cv2.StereoSGBM` |
| Použitie | Jednoduché scény | Autonómne vozidlá, priemerná robotika |

## Súvisiace pojmy

[[09 Problém korešpondencie, disparita a hĺbka]] · [[disparity and depth]] · [[epipolar geometry]]
