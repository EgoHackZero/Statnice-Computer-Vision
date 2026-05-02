# 02 SIFT — descriptor and Generalized Hough Transform

> *Otázka: SIFT, deskriptor kľúčového bodu, orientácia a škála, postup detekcie a lokalizácie objektu s využitím zovšeobecnenej Houghovej transformácie.*

## Stručný prehľad

Po detekcii a filtrovaní kľúčových bodov (viď [[01 SIFT — scale space and keypoint detection]]) SIFT vytvára pre každý kľúčový bod **128-rozmerný deskriptor** — kompaktný číselný popis lokálneho okolia bodu, ktorý je invariantný voči škále, rotácii a čiastočne aj voči zmenám osvetlenia. Deskriptory umožňujú spoľahlivo párovať kľúčové body medzi rôznymi obrazmi toho istého objektu. Následne sa zhody deskriptorov využívajú v *Zovšeobecnenej Houghovej transformácii (Generalized Hough Transform, GHT)*, ktorá akumuluje "hlasy" pre rôzne hypotézy o polohe, škále a orientácii objektu v scéne. Tento dvoj-etapový proces (deskriptor + GHT) tvoril základ Lowehho systému rozpoznávania objektov v preplnených scénach a jeho princípy pretrvávajú v moderných metódach SfM a AR.

---

## Invariantnosť voči škále a orientácii

### Škálová invariantnosť

Každý kľúčový bod nesie informáciu o **škále** $s$ — hodnote $\sigma$, pri ktorej bol nájdený DoG extrém. Táto škála určuje, aká veľká oblasť obrazu zodpovedá danému kľúčovému bodu:

- Deskriptor sa vždy vypočítava z oblasti **proporcionálnej škále** $\sigma$ kľúčového bodu: konkrétne oblasť $16\sigma \times 16\sigma$ pixelov okolo kľúčového bodu
- Ak je objekt 2× väčší (bližšie ku kamere), jeho kľúčové body budú mať 2× väčšie $\sigma$ a deskriptor sa extrahuje z 2× väčšej oblasti — výsledok je rovnaký

Vďaka tomu rovnaký rohový bod fasády budovy vyprodukuje zhodný deskriptor bez ohľadu na to, či fotografia bola urobená z 10 m alebo 5 m.

### Rotačná invariantnosť

Pred výpočtom deskriptora sa oblasť kľúčového bodu **rotuje o dominantnú orientáciu** $\theta$ kľúčového bodu (priradená v predchádzajúcom kroku pomocou 36-binového histogramu gradientov):

1. Koordinátový systém oblasti kľúčového bodu sa otočí tak, aby osa $x$ smerovala v smere $\theta$
2. Deskriptor sa počíta v tomto **lokálnom súradnicovom systéme**
3. Keď sa objekt otočí o uhol $\phi$, dominantná orientácia $\theta$ sa zmení o $\phi$, oblasť sa pred výpočtom rotuje rovnako — deskriptor ostáva nezmenený

> Kľúčový bod je tak plnou štvoricou $(x, y, s, \theta)$: poloha v obraze, škála a orientácia. Deskriptor je výhradne funkciou lokálneho obsahu v tomto normalizovanom rámci.

---

## Konštrukcia 128-rozmerného deskriptora

Deskriptor SIFT je konštruovaný nasledujúcim postupom (viď tiež [[01 SIFT — scale space and keypoint detection]]):

### Postup krok za krokom

**1. Extrakcia oblasti:**
Z Gaussovsko vyhladeného obrazu na škále $\sigma$ kľúčového bodu sa extrahuje oblasť $16\sigma \times 16\sigma$ pixelov, centrovaná na kľúčovom bode a **rotovaná do lokálnej orientácie** $\theta$.

**2. Výpočet gradientov:**
V každom z $16 \times 16 = 256$ pixelov sa vypočíta gradient — veľkosť $m(x,y)$ a smer $\theta(x,y)$. Príspevky sú vážené Gaussovským oknom so $\sigma = 8$ (polovica veľkosti oblasti), aby pixely blízko stredu mali väčší vplyv.

**3. Rozdelenie na subregióny:**
Oblast $16 \times 16$ sa rozdelí na mriežku $4 \times 4$ subregionov, každý o veľkosti $4 \times 4$ pixelov.

**4. Histogramy orientácií:**
V každom zo 16 subregionov sa zostaví **8-binový histogram orientácií** (každý bin = 45°, pokrýva 360°). Príspevok každého pixelu je proporcionálny veľkosti jeho gradientu (vopred váženej Gaussovským oknom).

**5. Konkatenácia:**
16 histogramov × 8 hodnôt = $16 \times 8 = \mathbf{128}$ čísel → 128-rozmerný vektor

**6. Normalizácia a orezanie:**
- Deskriptor sa normalizuje na jednotkovú L2-normu: $\mathbf{d} \leftarrow \mathbf{d} / \|\mathbf{d}\|_2$
- Hodnoty presahujúce 0.2 sa odrežú (clip): $d_i \leftarrow \min(d_i, 0.2)$ — toto redukuje vplyv nelineárnych zmien osvetlenia
- Opätovná L2-normalizácia: $\mathbf{d} \leftarrow \mathbf{d} / \|\mathbf{d}\|_2$

### Vlastnosti deskriptora

- **Robustnosť voči malým posuvom:** Vďaka hlasovaniu gradientov do susedných bínov (soft binning) malá zmena polohy spôsobí len graduálnu zmenu deskriptora
- **Illumination invariance:** Normalizácia a clipping 0.2 robia deskriptor robustným voči lineárnym aj miernym nelineárnym zmenám jasu
- **Invariantnosť voči afínnemu skresleniu:** Čiastočná — SIFT toleruje skreslenia pohľadu do ~30–45°

### Matematické zhrnutie deskriptora

$$f_k = \sum_{(x,y) \in R_k} m(x,y) \cdot w(x,y) \cdot [\theta(x,y) \in \text{bin}_j], \quad k = 1,\ldots,128$$

kde $R_k$ je $k$-tý subregiónový bin, $w(x,y)$ je Gaussovská váha a $[\cdot]$ je indikátorová funkcia binu.

Výsledný deskriptor po normalizácii: $\hat{\mathbf{d}} = \text{clip}_{0.2}(\mathbf{d}/\|\mathbf{d}\|) / \|\text{clip}_{0.2}(\mathbf{d}/\|\mathbf{d}\|)\|$

---

## Párovanie deskriptorov — Lowehov pomerový test

Po extrakcii deskriptorov zo dvoch obrazov nasleduje **párovanie** — pre každý deskriptor $\mathbf{q}$ z obrazu 1 hľadáme najlepší zodpovedajúci deskriptor z obrazu 2 pomocou Euklidovskej vzdialenosti.

### Problém ambiguity

Prostý *nearest-neighbor matching* (najbližší sused) má vysokú mieru falzopozitívnych zhôd (false positives), pretože v obraze s opakujúcimi sa vzormi existuje veľa deskriptorov podobných $\mathbf{q}$. Nevieme rozlíšiť, či najbližší sused je skutočnou zhodou alebo náhodnou podobnosťou.

### Lowehov pomerový test (Lowe ratio test)

Lowe (2004) navrhol elegantný test: porovnaj vzdialenosť k **najbližšiemu** ($d_1$) a **druhému najbližšiemu** ($d_2$) susedovi:

$$\frac{d_1}{d_2} < \tau, \quad \tau = 0.8$$

- Ak je pomer **malý** ($< 0.8$): najbližší sused je výrazne bližšie ako druhý — zhoda je jednoznačná → **akceptujeme**
- Ak je pomer **blízko 1**: existujú dva takmer rovnako podobné deskriptory — zhoda je nejednoznačná (ambiguous) → **odmietame**

Podľa Lowehho experimentov tento test **eliminuje ~90% falzopozitívnych zhôd** pri strate len ~5% skutočne správnych zhôd — dramatické zlepšenie pomeru signál/šum pre následné kroky.

> **Príklad:** Kľúčový bod na okne budovy môže byť podobný mnohým iným okenným rohom. Ak $d_1/d_2 = 0.95$, existujú minimálne dve takmer rovnako dobré zhody — nevieme, ktorá je správna → odmietame. Ak $d_1/d_2 = 0.45$, jedna zhoda výrazne dominuje → akceptujeme.

Podrobnosti: [[Lowe ratio test]]

```python
import cv2
import numpy as np

sift = cv2.SIFT_create()
img1 = cv2.imread('object.jpg', cv2.IMREAD_GRAYSCALE)
img2 = cv2.imread('scene.jpg', cv2.IMREAD_GRAYSCALE)

kp1, des1 = sift.detectAndCompute(img1, None)
kp2, des2 = sift.detectAndCompute(img2, None)

# BFMatcher s L2 normou pre SIFT deskriptory
bf = cv2.BFMatcher(cv2.NORM_L2)
matches = bf.knnMatch(des1, des2, k=2)  # 2 najbližšie susedy

# Lowehov pomerový test
good_matches = []
for m, n in matches:
    if m.distance / n.distance < 0.8:
        good_matches.append(m)

print(f"Všetky zhody: {len(matches)}, Dobré zhody: {len(good_matches)}")
```

---

## Zovšeobecnená Houghova transformácia pre lokalizáciu objektu

Po získaní sady dobrých zhôd deskriptorov stojíme pred otázkou: **kde sa nachádza objekt v scéne?** Jednotlivé zhody nám dávajú lokálne korešpondencie, ale potrebujeme z nich vyvodiť globálnu hypotézu o polohe (a škále, orientácii) objektu.

*Zovšeobecnená Houghova transformácia (Generalized Hough Transform, GHT)* je hlasovacia technika, ktorá tento problém rieši elegatne. Viac o základoch: [[Generalized Hough Transform]]

### Fáza trénovania — budovanie R-tabuľky modelu

Pre referenčný (modelový) obraz objektu:

1. SIFT kľúčové body sa detegujú a ich deskriptory sa uložia
2. Pre každý kľúčový bod $i$ modelu sa uloží **relatívny vektor** k referenčnému bodu objektu (napr. ťažisko bounding boxu):

$$\mathbf{r}_i = \mathbf{c}_{model} - \mathbf{p}_i^{model}$$

kde $\mathbf{c}_{model}$ je centrum objektu a $\mathbf{p}_i^{model}$ je poloha $i$-tého kľúčového bodu v modeli. Okrem vektora $\mathbf{r}_i$ sa uloží aj škálový pomer a orientácia kľúčového bodu voči modelu.

Výsledkom je **R-tabuľka (hlasovacie záznamy)** — pre každý deskriptor vieme, kde by objekt mal byť, ak nájdeme túto charakteristiku v scéne.

### Fáza detekcie — hlasovanie v akumulátore

Pre každý kľúčový bod $j$ v novom (testovom) obraze, ktorý sa zhoduje s modelovým kľúčovým bodom $i$:

1. Odhad polohy centra objektu v scéne:

$$\hat{\mathbf{c}} = \mathbf{p}_j^{scene} + s \cdot R(\theta) \cdot \mathbf{r}_i$$

kde $s$ je pomer škál ($\sigma_j^{scene} / \sigma_i^{model}$), $R(\theta)$ je rotačná matica zodpovedajúca zmene orientácie, $\mathbf{r}_i$ je uložený relatívny vektor z modelu.

2. Hlas sa pridá do **4D akumulátora** $(c_x, c_y, \text{škála}, \text{orientácia})$ — inkrementuje sa bin zodpovedajúci odhadnutej polohe centra pri odhadnutej škále a orientácii.

Formálne:

$$\text{Acc}(c_x, c_y, s, \theta) \mathrel{+}= 1 \quad \text{pre každú zhodu } j \leftrightarrow i$$

### Identifikácia kandidátov

Po spracovaní všetkých zhôd sa v akumulátore identifikujú **lokálne maximá** — oblasti s najväčším počtom hlasov. Každý takýto peak reprezentuje hypotézu o prítomnosti objektu.

Lowe uvádza: *"Subsets of keypoints that agree on the object's location, scale, and orientation are identified using an efficient hash table implementation of the Generalized Hough Transform."*

**Kritérium:** Cluster zosilnený aspoň **3 zhodami** je považovaný za silnú hypotézu (pravdepodobnosť náhodnej zhody je štatisticky nízka).

### Verifikácia afínnou transformáciou

Identifikovaný peak akumulátora je ešte len hrubou hypotézou. Pre každý kandidát sa vykoná **geometrická verifikácia**:

1. Vyberú sa všetky zhody, ktoré hlasovali pre tento peak
2. Pomocou **metódy najmenších štvorcov (least squares)** sa odhadne afínna transformácia $A$ (6 parametrov: rotácia, škála, skreslenie, posun) mapujúca modelové súradnice na scénové:

$$\begin{pmatrix} x_j^{scene} \\ y_j^{scene} \end{pmatrix} \approx A \begin{pmatrix} x_i^{model} \\ y_i^{model} \end{pmatrix} + \mathbf{t}$$

3. Zhody, ktoré sú ďalej ako prah od predikcie ($> 1.5$ pixelu), sú označené ako **outliers** (odľahlé hodnoty) a vyradené
4. Výsledná konzistentnosť sa testuje: ak zostatok zhôd vysvetlí transformáciu so štatisticky signifikantnou pravdepodobnosťou, objekt je **detegovaný**

Modernejšou alternatívou je **RANSAC** (Random Sample Consensus), ktorý robustnejšie odhaduje transformáciu pri väčšom počte outlierov.

### Diagram celého procesu

```
Model:  SIFT kľúčové body → deskriptory + relatívne vektory k centru (R-tabuľka)
                                         ↓
Scéna:  SIFT kľúčové body → deskriptory
                                         ↓
          Nearest-neighbor matching + Lowe ratio test → Dobré zhody
                                         ↓
          GHT hlasovanie: každá zhoda → hlas pre (cx, cy, s, θ)
                                         ↓
          Akumulátor → lokálne maximá (kandidáti)
                                         ↓
          Afínna verifikácia → potvrdené detekcie
```

---

## Matematika — zhrnutie kľúčových vzorcov

**128-rozmerný deskriptor (schematicky):**
$$\mathbf{d} \in \mathbb{R}^{128}, \quad d = [\text{hist}_1, \text{hist}_2, \ldots, \text{hist}_{16}], \quad \text{hist}_k \in \mathbb{R}^8$$

**Normalizácia deskriptora:**
$$\hat{\mathbf{d}} = \frac{\text{clip}(\mathbf{d}/\|\mathbf{d}\|_2,\ 0.2)}{\|\text{clip}(\mathbf{d}/\|\mathbf{d}\|_2,\ 0.2)\|_2}$$

**Lowehov pomerový test:**
$$\frac{d_1}{d_2} < 0.8 \quad \Rightarrow \quad \text{akceptuj zhodu}$$

kde $d_1 = \|\mathbf{q} - \mathbf{nn}_1\|_2$ a $d_2 = \|\mathbf{q} - \mathbf{nn}_2\|_2$.

**GHT hlas pre centrum objektu:**
$$\hat{\mathbf{c}} = \mathbf{p}_j + s \cdot R_\theta \cdot \mathbf{r}_i$$

$$\text{kde } s = \frac{\sigma_j^{scene}}{\sigma_i^{model}}, \quad R_\theta = \begin{pmatrix} \cos\Delta\theta & -\sin\Delta\theta \\ \sin\Delta\theta & \cos\Delta\theta \end{pmatrix}, \quad \Delta\theta = \theta_j^{scene} - \theta_i^{model}$$

**Afínna verifikácia (najmenšie štvorce):**
$$\min_A \sum_k \left\| \mathbf{p}_k^{scene} - A\mathbf{p}_k^{model} \right\|_2^2$$

---

## Porovnania a kompromisy

| Aspekt | SIFT + GHT | SIFT + RANSAC | Deep learning (SuperPoint + SuperGlue) |
|---|---|---|---|
| Presnosť | vysoká | veľmi vysoká | najvyššia |
| Rýchlosť | stredná | stredná | pomalá (GPU) |
| Robustnosť voči outlierom | stredná | veľmi vysoká | vysoká |
| Interpretovateľnosť | vysoká | vysoká | nízka (black-box) |
| Potreba trénania | nie | nie | áno (GPU) |

**SIFT + GHT** je vhodný keď:
- Máme malú trénovaciu vzorku (len referenčný obraz)
- Potrebujeme interpretovateľný dôkaz detekcie
- Scéna obsahuje výrazné textúrované objekty

**RANSAC** nahrádza GHT v SfM pipeline (COLMAP), pretože lepšie spracúva veľký počet outlierov.

---

## Časté zlyhania a obmedzenia

- **Symetrické objekty:** Objekt so symetriou (napr. fľaša, kruh) produkuje zhody na viacerých miestach → viacero peakov akumulátora → nejasnosť v GHT
- **Nedostatok textúry:** Hladké povrchy (plastové nádoby, sklo) nemajú dostatočný počet kľúčových bodov pre spoľahlivé hlasovanie (potrebné min. 3 zhody)
- **Silný clutter (preplnená scéna):** Ak scéna obsahuje stovky podobných objektov, akumulátor má mnoho peakov a verifikácia musí prebehnúť pre každý kandidát → pomalé
- **Veľká zmena škály:** GHT akumulátor je 4D, pri veľkej zmene škály je výpočtový priestor obrovský a binning musí byť hrubší → nižšia presnosť
- **Čiastočná viditeľnosť (occlusion):** Ak je objekt zakrytý, nájde sa menej zhôd. Ak menej ako 3 zhody zostanú po filtrovaní → objekt nie je detegovaný

---

## Praktické využitie

1. **Rozpoznávanie objektov v preplnených scénach (Lowe 2004):** Pôvodný systém testoval databázu 32 objektov v scénach s clutterom a dosiahol >95% detekčnú mieru, čo bolo prelomové. Kód: databáza deskriptorov + GHT akumulátor implementovaný cez hash tabuľku.

2. **Structure from Motion (COLMAP):** COLMAP extrahuje SIFT deskriptory z tisícok fotografií, páruje ich pomocou pomerového testu a overuje zhody RANSAC-om. Výsledkom je hustá 3D mapa a odhadnuté pozície kamier.

3. **Augmented Reality (AR) markery:** Systémy ako Vuforia alebo ARCore používajú SIFT-podobné deskriptory na rozpoznanie plakatov, produktov alebo miest v reálnom čase. Každý referenčný objekt má v databáze uloženú R-tabuľku.

4. **Panoramatické spájanie (image stitching):** `cv2.Stitcher_create()` interně: detekcia (SIFT), pomerový test, RANSAC na estimáciu homografie, warp a blending. Použitie v Google Street View, Microsoft ICE.

5. **Medicínska registrácia snímok:** Registrácia histologických rezov — pre každý rez sa extrahujú SIFT kľúčové body, GHT/RANSAC odhadne transformáciu na zarovnanie sériových rezov pre 3D rekonštrukciu tkaniva.

6. **Vyhľadávanie obrazov podľa obsahu (CBIR — Content-Based Image Retrieval):** Deskriptory SIFT sa indexujú v *Bag of Visual Words (BoVW)* databáze. Dotaz (query image) sa vybavuje nearest-neighbor search v BoVW priestore, akcelerovaný KD-stromom alebo FLANN.

```python
import cv2

# Príklad FLANN-based matcheru (rýchlejší pre veľké databázy)
sift = cv2.SIFT_create()
kp1, des1 = sift.detectAndCompute(img1, None)
kp2, des2 = sift.detectAndCompute(img2, None)

# FLANN parametre pre float deskriptory (SIFT/SURF)
index_params = dict(algorithm=1, trees=5)  # FLANN_INDEX_KDTREE
search_params = dict(checks=50)
flann = cv2.FlannBasedMatcher(index_params, search_params)

matches = flann.knnMatch(des1, des2, k=2)

# Lowehov pomerový test
good = [m for m, n in matches if m.distance / n.distance < 0.8]
print(f"Dobré zhody po ratio teste: {len(good)}")
```

---

## Súvisiace pojmy

[[01 SIFT — scale space and keypoint detection]], [[scale space and Gaussian pyramid]], [[Difference of Gaussians]], [[Generalized Hough Transform]], [[Lowe ratio test]]

---

*Spýtaj sa sám seba:*

1. Prečo je deskriptor 128-rozmerný a nie napr. 64 alebo 256? Aký je kompromis?
2. Čo robí clipping na 0.2 pri normalizácii deskriptora a prečo?
3. Prečo Lowehov pomerový test funguje lepšie ako prostý threshold na absolútnu vzdialenosť?
4. Ako GHT prekonáva problém outlierov — prečo stačia aspoň 3 zhody?
5. Ako by sa GHT správalo, ak by objekt bol symetrický (napr. kruh)? Aké by boli dôsledky?
