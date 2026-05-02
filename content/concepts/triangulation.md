# Triangulation

*Triangulácia* je výpočet 3D polohy bodu zo zodpovedajúcich 2D projekcií v dvoch alebo viacerých obrazoch, pri znalosti pozícií kamier.

## Geometrická intuícia

Každá 2D projekcia $(u, v)$ v kamere definuje **lúč** (ray) z optického stredu kamery do 3D priestoru. Ak máme dve kamery s rôznymi pozíciami, dva lúče zodpovedajúce rovnakému bodu by sa mali **pretnúť** v 3D polohe bodu.

V praxi kvôli šumu sa lúče nepretínajú presne → hľadáme bod minimalizujúci vzdialenosť od oboch lúčov.

## Lineárna triangulácia (DLT)

Pre dve projekcie $\mathbf{m}_1 \sim P_1 \mathbf{M}$ a $\mathbf{m}_2 \sim P_2 \mathbf{M}$ v homogénnych súradniciach:

Každá projekcia dáva 2 lineárne rovnice (z cross-productu):
$$\mathbf{m}_i \times (P_i \mathbf{M}) = \mathbf{0}$$

Rozvinutím:
$$\begin{bmatrix} u_1 \mathbf{p}_3^{1T} - \mathbf{p}_1^{1T} \\ v_1 \mathbf{p}_3^{1T} - \mathbf{p}_2^{1T} \\ u_2 \mathbf{p}_3^{2T} - \mathbf{p}_1^{2T} \\ v_2 \mathbf{p}_3^{2T} - \mathbf{p}_2^{2T} \end{bmatrix} \mathbf{M} = A \mathbf{M} = \mathbf{0}$$

kde $\mathbf{p}_i^{jT}$ sú riadky projekčnej matice $P_j$. Riešenie: posledný pravý singulárny vektor SVD($A$).

## Stereo triangulácia (jednoduchý prípad)

Pre kalibrovaný a rektifikovaný stereo pár so spoločnou ohniskovou vzdialenosťou $f$ a baselineom $B$:

$$Z = \frac{f \cdot B}{d}, \quad X = \frac{(u_L - c_x) Z}{f_x}, \quad Y = \frac{(v - c_y) Z}{f_y}$$

kde $d = u_L - u_R$ je disparita. Podrobne: [[disparity and depth]].

## Optimal Triangulation

Lineárna DLT minimalizuje algebraickú chybu, nie geometrickú. *Optimal triangulation* (Hartley & Sturm) minimalizuje reprojekčnú chybu:

$$\min_{\hat{\mathbf{m}}_1, \hat{\mathbf{m}}_2} d(\mathbf{m}_1, \hat{\mathbf{m}}_1)^2 + d(\mathbf{m}_2, \hat{\mathbf{m}}_2)^2$$

kde $\hat{\mathbf{m}}_1, \hat{\mathbf{m}}_2$ sú korigované projekcie zodpovedajúce nejakému skutočnému 3D bodu. Náročnejší výpočet, ale presnejší.

## Multi-view triangulácia

Pre $n > 2$ kamier:
- Zostavíme sústavu z $2n$ rovníc (2 na kameru)
- Stále lineárny systém → SVD riešenie
- Viac kamier = robustnejší odhad (overdetermined system → lepší LS fit)
- Používané v [[bundle adjustment]] v SfM

## Presnosť a zdroje chýb

| Zdroj chyby | Efekt |
|---|---|
| Šum v detekcii kľúčového bodu | ± 1–2 px chyba → závisí od $Z$ |
| Chyba kalibrácie | Systematická chyba v $K$ |
| Malý baseline | Zlá triangulovanosť (veľká hĺbková neistota) |
| Blízky bod (veľká disparita) | Dobrá presnosť |
| Vzdialený bod (malá disparita) | Zlá presnosť ($\sigma_Z \propto Z^2$) |

**Propagácia chyby:** Pre stereo: $\sigma_Z \approx \frac{Z^2}{fB}\sigma_d$

## Využitie

- [[09 Problém korešpondencie, disparita a hĺbka\|Stereo depth estimation]] — priamo z disparity
- [[10 Structure-from-Motion a Multi-View Stereo\|SfM]] — triangulácia 3D bodov z párov kamier
- Pozičné meranie (sports tracking, mocap) — multi-camera setup

## Súvisiace pojmy

[[epipolar geometry]] · [[disparity and depth]] · [[bundle adjustment]] · [[camera intrinsics and Zhang calibration]]
