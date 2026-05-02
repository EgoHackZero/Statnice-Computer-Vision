# Disparity and Depth

*Disparita* je horizontálny posun medzi zodpovedajúcimi bodmi v ľavom a pravom rektifikovanom stereo obraze. Z disparity sa priamo vypočíta hĺbka bodu v priestore.

## Definícia disparity

Pre rektifikovaný stereo pár (oba obrazy zarovnané na rovnaký riadok):

$$d = u_L - u_R \quad [\text{pixely}]$$

kde $u_L$ je x-súradnica bodu v ľavom obraze a $u_R$ x-súradnica v pravom obraze.

**Znamienko:** Bežný stereo pár (ľavá kamera vľavo) → $u_L > u_R$ → $d > 0$ pre blízke objekty.

## Vzorec pre hĺbku

$$\boxed{Z = \frac{f \cdot B}{d}}$$

kde:
- $Z$ = hĺbka bodu (vzdialenosť od kamier) $[\text{m}]$
- $f$ = ohnisková vzdialenosť $[\text{px}]$
- $B$ = baseline — vzdialenosť medzi optickými stredmi kamier $[\text{m}]$
- $d$ = disparita $[\text{px}]$

### Numerický príklad

$f = 600$ px, $B = 0{,}12$ m, $d = 20$ px:
$$Z = \frac{600 \cdot 0{,}12}{20} = 3{,}6 \text{ m}$$

## 3D súradnice bodu

$$X = \frac{(u_L - c_x) \cdot Z}{f_x}, \quad Y = \frac{(v - c_y) \cdot Z}{f_y}$$

Alebo kompaktne v homogénnych súradniciach:
$$\begin{pmatrix} X \\ Y \\ Z \end{pmatrix} = \frac{Z}{f} \begin{pmatrix} u_L - c_x \\ v - c_y \\ f \end{pmatrix}$$

## Závislosť presnosti od hĺbky

Chyba hĺbky propagovaná z chyby disparity $\sigma_d$:

$$\sigma_Z = \frac{Z^2}{f \cdot B} \cdot \sigma_d$$

**Interpretácia:**
- Presnosť hĺbky klesá **kvadraticky** so vzdialenosťou
- Väčší baseline $B$ → lepšia presnosť (ale väčšia oklúzia)
- Väčšie $f$ (zoom objektív) → lepšia presnosť

Príklad: $f=600$ px, $B=0{,}12$ m, $Z=3{,}6$ m, $\sigma_d = 0{,}5$ px → $\sigma_Z = 0{,}18$ m (18 cm)

## Sub-pixelová disparita

Celočíselná disparita sa spresní fitovaním paraboly:

$$d^* = d + \frac{C(d-1) - C(d+1)}{2(C(d-1) - 2C(d) + C(d+1))}$$

kde $C(d)$ je matching cost (SSD, SAD) pri disparite $d$.

## Špeciálne prípady

| Situácia | Disparita | Hĺbka |
|---|---|---|
| Blízky objekt | Veľká $d$ | Malé $Z$ |
| Vzdialený objekt | Malá $d$ | Veľké $Z$ |
| $d \to 0$ | Skoro nulová | $Z \to \infty$ (napr. obloha) |
| $d < d_{min}$ | Mimo rozsahu | Neznáma |

## Disparity mapa vs. hĺbková mapa

| | Disparity mapa | Hĺbková mapa |
|---|---|---|
| Hodnoty | Pixely | Metre |
| Vzťah | $d = fB/Z$ | $Z = fB/d$ |
| Vizualizácia | Biely = blízko | Biely = blízko (alebo ďaleko — závisí od konvencie) |
| Artefakty v $d=0$ | Plochy v nekonečne | $Z \to \infty$ |

## OpenCV konverzia

```python
disparity = stereo.compute(img_L, img_R).astype(np.float32) / 16.0  # OpenCV vracia ×16
depth = (focal_length * baseline) / disparity
depth[disparity <= 0] = 0  # neplatné hodnoty
```

## Súvisiace pojmy

[[09 Problém korešpondencie, disparita a hĺbka]] · [[block matching and SGM]] · [[triangulation]] · [[epipolar geometry]]
