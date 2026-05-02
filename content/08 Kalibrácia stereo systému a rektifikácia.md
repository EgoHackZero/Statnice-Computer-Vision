# 08 Kalibrácia stereo systému a rektifikácia

> *Otázka: Kalibrácie systému dvoch kamier a princíp pasívnej stereovízie, rektifikácia obrazov.*

## Stručný prehľad

*Pasívna stereovízia (passive stereovision)* je princíp získavania 3D informácií pomocou **dvoch bežných kamier** bez aktívneho osvetlenia — len z geometrie rozdielov medzi dvoma perspektívnymi pohľadmi. Aby bolo možné presne triangulovať 3D polohu bodu, musia byť obe kamery **kalibrované** — musíme poznať ich vnútorné parametre (ohniská, stred obrazu, skreslenie) a vzájomnú polohu (rotáciu a transláciu). *Rektifikácia* je geometrická transformácia, ktorá zarovná oba obrazy tak, aby zodpovedajúce body ležali na rovnakých horizontálnych riadkoch — dramaticky zjednoduší algoritmy hľadania korešpondencií.

## Pasívna vs. aktívna stereovízia

| Vlastnosť | Pasívna stereovízia | Aktívna stereovízia (napr. štruktúrované svetlo) |
|---|---|---|
| Osvetlenie | Prirodzené alebo ambi­entné | Projektor (IR pruhy, bodky) |
| Textúreless povrchy | Problematické | Funguje (vzor dáva textúru) |
| Vonkajšie prostredie | Áno (slnko nevadí) | Obmedzené (IR rušenie) |
| Rýchlosť | Vysoká (bez pohyblivých čas­tí) | Stredná–vysoká |
| Cena | Nízka (2 kamery) | Stredná (projektor + kamera) |

## Kalibrácia jednej kamery (Zhangova metóda)

Kalibrácia každej kamery sa vykoná samostatne pred stereo kalibráciou.

**Postup (Zhang 2000):**
1. Zachyť **sériu snímok** (~15–30) kalibračného vzoru (šachovnica) z **rôznych uhlov a vzdialeností**
2. V každej snímke nájdi **rohové body šachovnice** (subpixelová presnosť; OpenCV: `cv2.findChessboardCorners()`)
3. Poznáme ich **3D súradnice** vo vzore (napr. roh $(i, j, 0)$ pre roh v riadku $i$, stĺpci $j$ pri rozteči 25 mm)
4. Z korešpondencií 3D ↔ 2D sa odhadnú parametre kamery

**Výstup kalibrácie:**

**Matica vnútorných parametrov** (*intrinsic matrix*) $K$:
$$K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}$$
kde:
- $f_x, f_y$: ohniskové vzdialenosti v pixeloch (pre obdĺžnikové pixely $f_x \neq f_y$)
- $c_x, c_y$: súradnice optického stredu (principal point) — zvyčajne blízko stredu obrazu

**Koeficienty skreslenia** (*distortion coefficients*):
- Radiálne (*radial distortion*): $k_1, k_2, k_3$ — „sudok" (barrel) alebo „vankúš" (pincushion)
- Tangenciálne (*tangential distortion*): $p_1, p_2$ — spôsobené nedokonalým zarovnaním šošovky

**Vonkajšie parametre** (*extrinsic parameters*) pre každú snímku: $R_i, \mathbf{t}_i$ — poloha vzoru voči kamere

**Bundle adjustment** (finálna optimalizácia): minimalizuje *reprojekčnú chybu* — rozdiel medzi skutočnou polohou rohu v snímke a projektovanou polohou použitím odhadnutých parametrov:
$$\min_{K, d, R_i, t_i} \sum_{i,j} \left\| \mathbf{m}_{ij} - \pi(K, d, R_i, \mathbf{t}_i, M_j) \right\|^2$$

## Fundamentálna matica a esenciálna matica

### Fundamentálna matica (*Fundamental Matrix*) $F$

Popisuje *epipólovú geometriu* medzi dvoma kamerami — vzťah medzi zodpovedajúcimi bodmi bez nutnosti poznať vnútorné parametre kamier.

**Algebraický vzťah:**
Pre každý pár zodpovedajúcich bodov $\mathbf{x}_1$ (v obraze 1) a $\mathbf{x}_2$ (v obraze 2) v homogénnych súradniciach platí:
$$\mathbf{x}_2^T F \mathbf{x}_1 = 0$$

**Vlastnosti $F$:**
- 3×3 matica, hodnosť 2 (singulárna)
- 7 stupňov voľnosti (nie 8 — kvôli hodnosti 2 a skalárnej neurčitosti)
- Platí pre **ľubovoľné dve kamery**, aj nekalibrované
- Dá sa odhadnúť z ~8+ korešpondenčných párov (8-bodový algoritmus + RANSAC)

### Esenciálna matica (*Essential Matrix*) $E$

Špeciálny prípad fundamentálnej matice pre **kalibrované** kamery:
$$E = K_2^T F K_1$$

$E$ kóduje len *vonkajšiu* geometriu — relatívnu rotáciu $R$ a transláciu $\mathbf{t}$ medzi kamerami. Z $E$ sa získa:
$$E = [t]_\times R$$
kde $[t]_\times$ je matica skew-symmetric zodpovedajúca krížovému súčinu $\mathbf{t} \times$.

Dekompozíciou $E$ cez SVD dostaneme 4 možné páry $(R, \mathbf{t})$ → správna sa overí (pozitívna hĺbka bodu pred kamerami).

### Homogénne súradnice

*(x, y)* v kartézskych súradniciach → $(\lambda x, \lambda y, \lambda)$ v homogénnych ($\lambda \neq 0$).

**Prečo sa používajú:**
- Projekcia, rotácia a translácia sú *lineárne transformácie* v homogénnych súradniciach → maticová multiplikácia
- Umožňujú reprezentovať body v nekonečne (napr. smer pohybu: $(x, y, 0)$)
- Premietanie 3D → 2D: $\mathbf{m} \sim K [R | \mathbf{t}] \mathbf{M}_{3D}$

## Kalibrácia stereo systému

Po individuálnej kalibrácii oboch kamier sa kalibruje *relatívna poloha* medzi nimi.

**Postup:**
1. Súčasne zachytíme snímky šachovnice **oboma kamerami** v rovnakom okamihu (synchronizovane)
2. V oboch snímkach nájdeme rohové body
3. Optimalizáciou dostaneme:
   - $K_1, d_1$, $K_2, d_2$: vnútorné parametre každej kamery
   - $R, \mathbf{t}$: rotácia a translácia ľavej kamery voči pravej (alebo naopak)
   - $E, F$: esenciálna a fundamentálna matica stereo páru

**Výstup stereo kalibrácie:**
- $K_1, K_2$: intrinsic matice
- $d_1, d_2$: distortion koeficienty
- $R, \mathbf{t}$: relatívna poloha kamier (*stereo extrinsics*)
- $E$: esenciálna matica
- $F$: fundamentálna matica

## Epipólová geometria

**Epipól** (*epipole*) $\mathbf{e}$ je priesečník spojnice dvoch optických stredov kamier s obrazovou rovinou. V rektifikovanom páre sú epipóly na „nekonečne" (vodorovný pohyb).

**Epipólová čiara** (*epipolar line*): keď je daný bod $\mathbf{x}_1$ v obraze 1, jeho korešpondenciu $\mathbf{x}_2$ v obraze 2 hľadáme **výlučne na epipólovej čiare** $\mathbf{l}_2 = F \mathbf{x}_1$. Tým sa 2D hľadanie redukuje na 1D.

## Rektifikácia obrazov

**Motivácia:** Bez rektifikácie sú epipólové čiary šikmé a rôznych sklon závisí od relatívnej polohy kamier → hľadanie korešpondencií je 2D problém.

**Rektifikácia** (*rectification*): geometrická transformácia oboch obrazov tak, aby:
- Zodpovedajúce body ležali na **rovnakej horizontálnej línii** (rovnaký riadok $v$)
- Epipólové čiary boli **horizontálne a rovnobežné**
- 2D korešpondenčný problém sa zredukoval na **1D vyhľadávanie** pozdĺž riadku

**Algoritmus (napr. Bouguet — implementovaný v OpenCV):**
1. Vypočítaj projekčné matice $P_1, P_2$ pre rektifikované kamery (nová optická os = horizontálna)
2. Vypočítaj mapy transformácií (homografie pre každú kameru): `cv2.stereoRectify()`
3. Aplikuj mapy: `cv2.remap()` → rektifikované obrazy
4. Zároveň sa aplikuje korekcia skreslenia (undistortion)

**Výsledok:** Dvojica rektifikovaných obrazov kde každý bod $(u_L, v)$ v ľavom obraze má korešpondenciu $(u_R, v)$ v pravom obraze na rovnakom riadku $v$.

**Kontrola kvality rektifikácie:** Vizuálne: horizontálne čiary nakreslené cez oba rektifikované obrazy by mali prechádzať zodpovedajúcimi bodmi.

## Matematika

**Projekčná rovnica (kamera):**
$$\begin{bmatrix} u \\ v \\ 1 \end{bmatrix} \sim K [R | \mathbf{t}] \begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}$$

**Matica K:**
$$K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}$$

**Epipólová podmienka:**
$$\mathbf{x}_2^T F \mathbf{x}_1 = 0, \quad F = K_2^{-T} [t]_\times R K_1^{-1}$$

**Esenciálna matica:**
$$E = K_2^T F K_1 = [t]_\times R$$

**Radiálne skreslenie (korekcia):**
$$x_{corrected} = x(1 + k_1 r^2 + k_2 r^4 + k_3 r^6), \quad r^2 = x^2 + y^2$$

## Porovnania a kompromisy

**Väčší baseline (vzdialenosť kamier):**
- Lepšia hĺbková presnosť pre vzdialené objekty ($Z \propto 1/d$, väčší $d$ pri väčšom $B$)
- Väčšia oklúzia (časti scény viditeľné len jednou kamerou)
- Väčší rozsah disparít → pomalší matching

**Menší baseline:**
- Menej oklúzie
- Horšia hĺbková presnosť

## Časté zlyhania a obmedzenia

- **Nedostatočná rôznorodosť snímok pri kalibrácii** → zlá pokrytosť parametrického priestoru → nepresné parametre
- **Vibrácie a tepelná rozťažnosť** → drift kalibračných parametrov v čase → nutná periodická rekalibrácia
- **Nekvalitná šachovnica** (skreslená, lesklá) → nepresná detekcia rohov → zlá kalibrácia
- **Synchronizácia** → ak kamery nie sú synchronizované, pohybujúci sa objekt spôsobí nekonzistentné rohy
- **Veľké radiálne skreslenie** (fish-eye šošovky) → štandardný model nepostačuje; nutné použiť oce-eye / omnidirectionálnu kalibráciu

## Praktické využitie

1. **Autonómne vozidlá** — stereo kamera (napr. ZED Camera 2, Intel RealSense D435) pre detekciu objektov a odhadovanie vzdialenosti; KITTI Stereo dataset
2. **Robotické manipulátory** — presné meranie polohy objektov na montážnej linke (presnosť ~1 mm pri správnej kalibrácii)
3. **Medicínske stereoskopické endoskopy** — Da Vinci chirurgický robot; 3D vizualizácia vnútra tela
4. **Fotogrammetria** — kalibrácia pri 3D rekonštrukcii (pred SfM): metashape, COLMAP obsahujú zabudované kalibračné procedúry
5. **UAV a drony** — stereo páry na odhad výšky nad terénom, vyhýbanie sa prekážkam (DJI Obstacle Sensing)

**Knižnice:** `cv2.calibrateCamera()`, `cv2.stereoCalibrate()`, `cv2.stereoRectify()`, `cv2.remap()`, `cv2.undistort()`

## Súvisiace pojmy

[[camera intrinsics and Zhang calibration]], [[epipolar geometry]], [[fundamental and essential matrix]], [[09 Problém korešpondencie, disparita a hĺbka]], [[07 2.5D, 3D a volumetrické reprezentácie]]

---
*Spýtaj sa sám seba:*
1. Čo je fundamentálna matica a čo vyjadruje rovnica $\mathbf{x}_2^T F \mathbf{x}_1 = 0$?
2. Prečo esenciálna matica $E$ vyžaduje kalibrované kamery, zatiaľ čo $F$ nie?
3. Čo je rektifikácia a prečo zjednodušuje hľadanie korešpondencií?
4. Ako ovplyvní väčší baseline (vzdialenosť kamier) presnosť merania hĺbky?
5. Čo je reprojekčná chyba a aká hodnota je považovaná za dobrú kalibráciu?
