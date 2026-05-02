# 09 Problém korešpondencie, disparita a hĺbka

> *Otázka: Riešenia problému korešpondencie pri stereovízii, triangulácia bodu v priestore, disparita a hĺbka.*

## Stručný prehľad

Po kalibrácii a rektifikácii stereo páru zostáva kľúčová otázka: **ktorý pixel v pravom obraze zodpovedá danému pixelu v ľavom obraze?** Toto je *problém korešpondencie (correspondence problem)*. Rozdiel v horizontálnej polohe zodpovedajúcich bodov nazývame *disparita (disparity)* — priamym vzorcom sa z nej vypočíta *hĺbka (depth)* každého bodu. Základnou metódou riešenia je *block matching* (lokálne porovnanie okien); pokročilejšia *Semi-Global Matching (SGM)* pridáva globálne podmienky hladkosti a dosahuje výrazne lepšie výsledky, najmä v oblasti bez textúry.

## Problém korešpondencie (*Correspondence Problem*)

**Definícia:**
Pre každý pixel $(u_L, v)$ v ľavom rektifikovanom obraze hľadáme zodpovedajúci pixel $(u_R, v)$ v pravom rektifikovanom obraze — ten, ktorý zobrazuje rovnaký 3D bod scény.

**Prečo je to ťažké:**
- Rovnaký bod vyzerá v dvoch kamerách *trochu inak* (zmena perspektívy, osvetlenie)
- Textúreless oblasti nemajú unikátny vizuálny vzor → mnoho možných zhôd
- Oklúzia: niektoré body sú viditeľné len v jednom obraze
- Po rektifikácii hľadáme len na **rovnakom riadku** $v$ → 1D problém (namiesto 2D)

**Rozsah disparity** $[d_{min}, d_{max}]$: závisí od minima a maxima hĺbky scény; väčší baseline → väčší rozsah

## Block Matching (*Blokovýmatching*)

Block matching je lokálny algoritmus: pre každý pixel porovnáva jeho okolie (okno) s oknami na rovnakom riadku pravého obrazu.

**Algoritmus:**
1. Pre každý pixel $(u_L, v)$ v ľavom obraze vyber **okno** $W$ veľkosti $w \times w$ (napr. 5×5, 11×11)
2. Porovnaj toto okno s oknami v pravom obraze na **rovnakom riadku** $v$ pre každú disparitu $d \in [d_{min}, d_{max}]$
3. Vypočítaj mieru **podobnosti** pre každú disparitu:

**SSD (Sum of Squared Differences):**
$$\text{SSD}(d) = \sum_{(i,j) \in W} \left(I_L(u_L+i,\, v+j) - I_R(u_L-d+i,\, v+j)\right)^2$$

**SAD (Sum of Absolute Differences):** $\displaystyle\text{SAD}(d) = \sum_{(i,j) \in W} \left|I_L(\ldots) - I_R(\ldots)\right|$ — rýchlejšie ako SSD

**NCC (Normalized Cross-Correlation):** normalizuje priemer a rozptyl okna → invariantná voči lineárnym zmenám osvetlenia (multiplicatívne a aditivné zmeny)

4. Zvoľ disparitu $d^*$ s **minimálnou** SSD/SAD (alebo **maximálnou** NCC)

**Voľba veľkosti okna:**
- Malé okno ($w = 3$–5): zachytáva jemné detaily, citlivé na šum, horé na textúreless plochách
- Veľké okno ($w = 21$–51): odolné voči šumu, hladší výsledok, ale rozmazáva hrany disparity

## Triangulácia bodu v priestore

Ak poznáme disparitu $d$, 3D súradnice bodu sa vypočítajú trianguláciou:

**Disparita:**
$$d = u_L - u_R \quad [\text{pixely}]$$

**Hĺbka (vzdialenosť od kamery):**
$$Z = \frac{f \cdot B}{d}$$

kde:
- $f$ = ohnisková vzdialenosť v pixeloch (z kalibrácie)
- $B$ = baseline — vzdialenosť medzi optickými stredmi kamier [m]
- $d$ = disparita [pixely]

**3D poloha bodu:**
$$X = \frac{(u_L - c_x) \cdot Z}{f_x}, \quad Y = \frac{(v - c_y) \cdot Z}{f_y}, \quad Z = \frac{f \cdot B}{d}$$

**Interpretácia vzorca $Z = fB/d$:**
- Bližší objekt → väčší horizontálny posun → **väčšia disparita** → menšia hĺbka ✓
- Vzdialenejší objekt → menšia disparita
- Ak $d \to 0$: bod je „v nekonečne" (napr. obloha) — numerická nestabilita
- Presnosť hĺbky: $\Delta Z \approx Z^2 \Delta d / (f B)$ → kvadratická závislosť od hĺbky

**Príklad:**
$f = 600$ px, $B = 0{,}12$ m, $d = 20$ px → $Z = 600 \times 0{,}12 / 20 = 3{,}6$ m

## Problémy pri výpočte disparity

| Problém | Príčina | Riešenie |
|---|---|---|
| Textúreless oblasti | Žiadny unikátny vizuálny vzor | SGM, väčšie okno, regularizácia |
| Opakujúce sa vzory | Mnoho lokálnych minim | RANSAC verifikácia, globálna optimalizácia |
| Oklúzia (*occlusion*) | Bod viditeľný len v 1 obraze | Left-right consistency check |
| Lesklé/priehľadné povrchy | Iná farba z rôznych uhlov | Špeciálne kovtingy, aktívne stereo |
| Hrany disparity | Objekt pred pozadím | Adaptívne okno, segmentačne-riadená disparita |
| Neúplná rektifikácia | Reziduálna vertikálna disparita | Presná kalibrácia, sub-pixel rektifikácia |

## Semi-Global Matching (SGM, Hirschmuller 2008)

SGM je výrazné zlepšenie nad lokálnym block matchingom — pridáva **globálne obmedzenie hladkosti** disparity bez nutnosti riešiť plne globálnu optimalizáciu (NP-ťažkú).

**Princíp:**
SGM definuje celkovú energiu:
$$E(D) = \sum_p \left[C(p, D_p) + \sum_{q \in N_p} P_1 \mathbf{1}[|D_p - D_q| = 1] + \sum_{q \in N_p} P_2 \mathbf{1}[|D_p - D_q| > 1]\right]$$

kde $C(p, D_p)$ je lokálna cena zhody, $P_1$ penalizuje malú zmenu disparity, $P_2$ penalizuje veľkú zmenu (skokové zmeny — hrany).

**Agregácia po cestách:**
Namiesto plnej globálnej optimalizácie SGM aggreguje ceny pozdĺž **8 (alebo 16) smerov** (vodorovne, zvisle, diagonálne) cez celý obraz — každý smer je 1D dynamické programovanie:
$$L_r(p, d) = C(p, d) + \min\!\left[L_r(p-r, d),\, L_r(p-r, d\pm1) + P_1,\, \min_{d'} L_r(p-r, d') + P_2\right]$$

Výsledná cena = súčet po všetkých smeroch: $S(p, d) = \sum_r L_r(p, d)$

**Left-Right Consistency Check:**
- Vypočítaj disparitu z ľavého do pravého obrazu: $D_L$
- Vypočítaj disparitu z pravého do ľavého obrazu: $D_R$
- Bod $(u_L, v)$ je **platný**, ak $|D_L(u_L) - D_R(u_L - D_L(u_L))| \leq 1$
- Nekonzistentné body → oklúzia alebo chyba matchingu → označiť ako **neplatné** (interpolovať z okolia)

**Výhody SGM:**
- Výrazne lepšie výsledky ako lokálny block matching v textúreless oblastiach
- Zachováva ostré hrany disparity (hranica objekt-pozadie)
- Rozumná výpočtová náročnosť (quasi-lineárna v počte pixelov)
- Implementovaný: `cv2.StereoSGBM_create()` v OpenCV

## Výpočet sub-pixelovej disparity

Celočíselná disparita $d$ má obmedzenú presnosť. Sub-pixelové spresenie sa dosiahne **fitovaním paraboly** cez tri hodnoty ceny okolo minima:

$$d^* = d + \frac{C(d-1) - C(d+1)}{2\!\left(C(d-1) - 2C(d) + C(d+1)\right)}$$

kde $C(d)$ je hodnota chyby (SSD alebo inej metriky) pri disparite $d$.

**Príklad:**
$C(19) = 1200, \; C(20) = 800, \; C(21) = 1100$:
$$d^* = 20 + \frac{1200 - 1100}{2(1200 - 1600 + 1100)} = 20 + \frac{100}{2 \cdot 700} = 20 + 0{,}071 \approx 20{,}07$$

Sub-pixelová presnosť: pri $f = 600$ px, $B = 0{,}12$ m, $d = 20$ px → $\Delta d = 0{,}071$ px → $\Delta Z \approx 0{,}013$ m = 1,3 cm

## Matematika — zhrnutie

**Disparita:** $d = u_L - u_R$

**Hĺbka:** $Z = \frac{f \cdot B}{d}$

**3D bod:**
$$\begin{pmatrix} X \\ Y \\ Z \end{pmatrix} = \frac{Z}{f} \begin{pmatrix} u_L - c_x \\ v - c_y \\ f \end{pmatrix}$$

**Chyba hĺbky (propagácia chyby disparity $\sigma_d$):**
$$\sigma_Z \approx \frac{Z^2}{f \cdot B} \sigma_d$$

**Sub-pixelová disparita (parabola fit):**
$$d^* = d + \frac{C_{d-1} - C_{d+1}}{2(C_{d-1} - 2C_d + C_{d+1})}$$

## Porovnania

**Block matching vs. SGM:**
| | Block Matching | SGM |
|---|---|---|
| Textúreless oblasti | Zlé | Dobré |
| Hrany disparity | Rozmazané | Ostré |
| Rýchlosť | Rýchle | Pomalšie (ale rýchlejšie ako globálne metódy) |
| Implementácia | `cv2.StereoBM_create()` | `cv2.StereoSGBM_create()` |
| Kvalita | Dostatočná pre jednoduché scény | Quasi-štandard v autonómnych vozidlách |

## Časté zlyhania a obmedzenia

- **Textúreless oblasti** — stenová farba, obloha, dlha tkanina → nevypočítaná hĺbka
- **Oklúzia** — časti scény zakryté bližším objektom → chybná alebo chýbajúca disparita
- **Opakujúci sa vzor** — radiátor, koberec, mrežovanie → false matches
- **Veľká disparita** — blízke objekty mimo rozsahu $[d_{min}, d_{max}]$
- **Pohybujúce sa objekty** — ak sú zachytené v dvoch rámcoch v rôznych polohách → šum

## Praktické využitie

1. **Autonómne vozidlá** — odhadovanie vzdialenosti k chodcom, vozidlám, prekážkam; KITTI benchmark; kombinovanie stereo s LiDARom (sensor fusion)
2. **Robotické uchopovanie** — presné 3D súradnice objektov na stole pre robotickú ruku; presnosť ~5 mm pri $Z < 1$ m
3. **Priemyselná metrológia** — kontrola rozmerov dielov s presnosťou sub-mm pri správnej kalibrácii
4. **Medicínske endoskopy** — stereo laparoskopy pre 3D vizualizáciu vnútra tela; Da Vinci chirurgický robot
5. **Mapovanie z UAV** — stereo kamery na drone pre real-time bezpečnostné vyhýbanie sa prekážkam

**OpenCV implementácia:**
```python
stereo = cv2.StereoSGBM_create(
    minDisparity=0, numDisparities=128, blockSize=5,
    P1=8*3*5**2, P2=32*3*5**2,
    disp12MaxDiff=1, uniquenessRatio=15, speckleWindowSize=100
)
disparity = stereo.compute(img_left_gray, img_right_gray).astype(np.float32) / 16.0
depth = (focal_length * baseline) / disparity
```

## Súvisiace pojmy

[[disparity and depth]], [[block matching and SGM]], [[triangulation]], [[08 Kalibrácia stereo systému a rektifikácia]], [[10 Structure-from-Motion a Multi-View Stereo]]

---
*Spýtaj sa sám seba:*
1. Ako znižuje rektifikácia dimensionalitu problému korešpondencie z 2D na 1D?
2. Napíš vzorec pre hĺbku $Z$ z disparity. Čo sa stane s presnosťou hĺbky keď disparita klesne na 1 px?
3. V čom je SGM lepší ako lokálny block matching v textúreless oblastiach?
4. Čo je left-right consistency check a prečo ho potrebujeme?
5. Vypočítaj sub-pixelovú disparitu ak $C(9) = 500, C(10) = 300, C(11) = 480$.
