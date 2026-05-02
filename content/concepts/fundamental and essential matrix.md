# Fundamental and Essential Matrix

## Fundamentálna matica $F$

*Fundamentálna matica* $F$ je 3×3 matica, ktorá kóduje epipólovú geometriu medzi dvoma kamerami **bez nutnosti poznania vnútorných parametrov** (nekalibrované kamery).

### Algebraická podmienka

Pre každý pár zodpovedajúcich bodov $\mathbf{x} = (u_1, v_1, 1)^T$ a $\mathbf{x}' = (u_2, v_2, 1)^T$ v homogénnych pixelových súradniciach:

$$\mathbf{x}'^T F \mathbf{x} = 0$$

### Vlastnosti $F$

- Matica $3 \times 3$, hodnosť 2 (singulárna: $\det(F) = 0$)
- 7 stupňov voľnosti (nie 8 — singularita $\det(F)=0$ a skalárna neurčitosť redukujú z 9 na 7)
- Platí pre **ľubovoľné** dve kamery (aj nekalibrované, aj s rôznymi intrinsics)
- Explicitný tvar: $F = K_2^{-T} [t]_\times R K_1^{-1}$

### Výpočet $F$ (8-bodový algoritmus)

Z $n \geq 8$ korešpondujúcich párov $(\mathbf{x}_i, \mathbf{x}'_i)$ zostavíme lineárny systém:

$$u_1'u_1 f_{11} + u_1'v_1 f_{12} + u_1' f_{13} + v_1'u_1 f_{21} + \ldots + f_{33} = 0$$

Každý pár dáva jeden riadok → matica $A$ veľkosti $n \times 9$. Riešenie: posledný pravý singulárny vektor SVD($A$). Potom vynútime hodnosť 2 ďalšou SVD + nulovaním najmenšieho singulárneho čísla.

**V praxi: RANSAC** na filtrovanie outlierov → robustný odhad $F$.

---

## Esenciálna matica $E$

*Esenciálna matica* $E$ je špeciálny prípad $F$ pre **kalibrované kamery** — kóduje len relatívnu polohu kamier (rotácia + translácia) bez vnútorných parametrov.

### Vzťah medzi $E$ a $F$

$$E = K_2^T F K_1$$

kde $K_1, K_2$ sú intrinsic matice oboch kamier. Alternativne:

$$E = [t]_\times R$$

kde $[t]_\times$ je skew-symmetric matica zodpovedajúca $\mathbf{t} \times$:

$$[t]_\times = \begin{bmatrix} 0 & -t_z & t_y \\ t_z & 0 & -t_x \\ -t_y & t_x & 0 \end{bmatrix}$$

### Vlastnosti $E$

- 3×3, hodnosť 2
- **5 stupňov voľnosti** (3 pre R + 3 pre t − 1 pre skalárovú neurčitosť mierky = 5)
- Platí len pre **kalibrované** kamery (vyžaduje $K_1, K_2$)
- Epipólová podmienka v normalizovaných súradniciach $\hat{\mathbf{x}}'^T E \hat{\mathbf{x}} = 0$

### Dekompozícia $E$ → $(R, \mathbf{t})$

SVD: $E = U \Sigma V^T$

Existujú **4 možné riešenia** $(R_1, +\mathbf{t})$, $(R_1, -\mathbf{t})$, $(R_2, +\mathbf{t})$, $(R_2, -\mathbf{t})$. Správne sa overí trianguláciou: správna konfigurácia dá kladnú hĺbku ($Z > 0$) pre väčšinu bodov.

---

## Porovnanie $F$ vs. $E$

| | Fundamentálna $F$ | Esenciálna $E$ |
|---|---|---|
| Vyžaduje kalibráciu? | Nie | Áno ($K_1, K_2$) |
| Pracuje v | Pixelových súradniciach | Normalizovaných súradniciach |
| Stupne voľnosti | 7 | 5 |
| Minimálny počet bodov | 7 (7-point alg.) alebo 8 | 5 (5-point alg.) |
| Výstup | Epipólová geometria | Relatívna poloha $(R, t)$ |

## Súvisiace pojmy

[[epipolar geometry]] · [[camera intrinsics and Zhang calibration]] · [[08 Kalibrácia stereo systému a rektifikácia]] · [[triangulation]]
