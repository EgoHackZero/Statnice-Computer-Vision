# Camera Intrinsics and Zhang Calibration

## Kamerový model (pinhole)

Premietanie 3D bodu $\mathbf{M} = (X, Y, Z)^T$ na pixel $(u, v)$:

$$\begin{bmatrix} u \\ v \\ 1 \end{bmatrix} \sim K [R | \mathbf{t}] \begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}$$

kde $\sim$ znamená rovnosť v homogénnych súradniciach (s konštantnou mierkou $\lambda$).

## Matica vnútorných parametrov $K$ (Intrinsic Matrix)

$$K = \begin{bmatrix} f_x & s & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}$$

| Parameter | Popis | Typická hodnota |
|---|---|---|
| $f_x, f_y$ | Ohnisková vzdialenosť v pixeloch (x, y) | 500–3000 px |
| $c_x, c_y$ | Principal point — optický stred obrazu | $\approx$ polovica rozlíšenia |
| $s$ | Skewness (šikmosť pixelov) | $\approx 0$ (moderné kamery) |

## Koeficienty skreslenia (Distortion)

### Radiálne skreslenie

$$x_{corr} = x(1 + k_1 r^2 + k_2 r^4 + k_3 r^6)$$

kde $r^2 = x^2 + y^2$ v normalizovaných (principal-point-centrovaných) súradniciach.
- $k_1 > 0$ → *pincushion* (vankúšové skreslenie)
- $k_1 < 0$ → *barrel* (sudové skreslenie)

### Tangenciálne skreslenie

$$x_{t} = x + [2p_1 xy + p_2(r^2 + 2x^2)]$$
$$y_{t} = y + [p_1(r^2 + 2y^2) + 2p_2 xy]$$

Spôsobené nedokonalým zarovnaním optickej šošovky a senzora.

## Zhangova kalibrácia (Zhang 2000)

Najpoužívanejšia metóda kalibrácie — potrebná len šachovnica a bežné fotoaparát.

### Postup

1. **Sním ~15–30 fotografií šachovnice** z rôznych uhlov, vzdialeností a natočení (rotácia ≥ 45° v každej osi)
2. **Detekuj rohy šachovnice** na subpixelovú presnosť: `cv2.findChessboardCorners()` + `cv2.cornerSubPix()`
3. **Korešpondencie 3D ↔ 2D:** 3D súradnice rohov v súradnicovej sústave vzoru sú known (napr. 25mm rozstup)
4. **Odhad homográfie** pre každý obraz — lineárny systém
5. **Odvodenie K** z podmienok na homográfiach (IAC — Image of Absolute Conic)
6. **Bundle adjustment** — nelineárna optimalizácia minimalizujúca reprojekčnú chybu:

$$\min_{K, d, R_i, t_i} \sum_{i,j} \left\| \mathbf{m}_{ij} - \pi(K, d, R_i, \mathbf{t}_i, M_j) \right\|^2$$

### Reprojekčná chyba

Priemerná euklidovská vzdialenosť medzi skutočnými rohmi šachovnice a ich projektovanými polohami. Dobrá kalibrácia: **< 1 pixel** (typicky 0.2–0.5 px).

### Odporúčania pre prax

- Pokryť celý obraz — rohy šachovnice musia byť vo všetkých častiach zorného poľa (nie len stred)
- Variácia vzdialenosti: blízko aj ďaleko od kamery (pre odhad $f$ vs. $k_1$)
- Tuhá šachovnica (nie vytlačená na papieri — môže sa prehýbať)
- Synchronizácia pre stereo kalibráciu — obe kamery zachytia rovnakú šachovnicu simultánne

## OpenCV implementácia

```python
ret, K, dist, rvecs, tvecs = cv2.calibrateCamera(
    objpoints,   # 3D body vzoru (list of np.array)
    imgpoints,   # 2D body v každom obraze
    gray.shape[::-1],
    None, None
)
print(f"Reprojection error: {ret:.3f} px")
print(f"K = {K}")
```

## Vonkajšie parametre (Extrinsics)

Pre každý obraz kalibrácie dostaneme aj:
- $R_i$ (rotácia 3×3), $\mathbf{t}_i$ (translácia 3×1) → poloha šachovnice voči kamere

V stereo systémoch je jeden pár $(R, \mathbf{t})$ kľúčový: relatívna poloha ľavej kamery voči pravej.

## Súvisiace pojmy

[[08 Kalibrácia stereo systému a rektifikácia]] · [[epipolar geometry]] · [[fundamental and essential matrix]] · [[triangulation]]
