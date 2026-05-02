# 01 SIFT — scale space and keypoint detection

> *Otázka: Scale Invariant Feature Transform, jednotlivé kroky, priestoru škál, kandidáti na kľúčové body (keypoints) a ich filtrovanie.*

## Stručný prehľad

*Scale-Invariant Feature Transform (SIFT)* je algoritmus na detekciu a popis lokálnych príznakov obrazu, ktorý navrhol David G. Lowe (publikovaný v IJCV 2004). Hlavná vlastnosť SIFT je **invariantnosť voči zmene mierky a rotácii** — kľúčový bod nájdený v blízkom aj vzdialenom pohľade na ten istý objekt bude mať zhodný deskriptor. SIFT pracuje vo *priestore škál (scale space)*, kde sa obraz analyzuje na viacerých úrovniach rozmazania a rozlíšenia súčasne. Výsledkom algoritmu je množina *kľúčových bodov (keypoints)* — každý so súradnicami $(x, y)$, škálou $\sigma$, orientáciou $\theta$ a 128-rozmerným deskriptorom — ktoré slúžia ako základný stavebný kameň pre rozpoznávanie objektov, panoramatické spájanie snímok alebo 3D rekonštrukciu scény.

---

## Krok 1: Konštrukcia priestoru škál — Gaussovská pyramída

*Priestor škál (scale space)* je matematický rámec, kde obraz $I(x,y)$ je reprezentovaný rodinou vyhladených obrazov $L(x, y, \sigma)$ parametrizovaných hodnotou $\sigma$ (štandardná odchýlka Gaussovho jadra):

$$L(x, y, \sigma) = G(x, y, \sigma) * I(x,y)$$

kde $G(x, y, \sigma) = \frac{1}{2\pi\sigma^2} e^{-(x^2+y^2)/(2\sigma^2)}$ je Gaussovské jadro.

### Oktávy a intervaly

SIFT organizuje priestor škál do **oktáv (octaves)**. V každej oktáve je obraz rozmazaný radom Gaussovských jadier s rastúcou $\sigma$, pričom sú zachované všetky medzivýsledky:

- **Počet oktáv:** typicky 4 (závisí od rozlíšenia vstupného obrazu)
- **Počet intervalov na oktávu:** $s = 3$ (t. j. detegujú sa extrémy na 3 škálových úrovniach)
- **Počet Gaussovských obrazov na oktávu:** $s + 3 = 6$ (pre korektné výpočty DoG na krajných škálach)
- **Počiatočná sigma:** $\sigma_0 = 1.6$ — empiricky optimálna hodnota podľa Lowea
- **Faktor mierky medzi susednými obrazmi:** $k = 2^{1/s} = 2^{1/3} \approx 1.26$

Škály v rámci jednej oktávy sú teda: $\sigma, k\sigma, k^2\sigma, k^3\sigma, k^4\sigma, k^5\sigma$.

Na začiatku každej novej oktávy sa obraz **podvzorkuje (downsampled)** faktorom 2, čo redukuje výpočtovú náročnosť a zároveň rozširuje efektívny rozsah škál. Vstupný obraz sa pred prvou oktávou voliteľne upsampleuje 2× pre lepšiu lokalizáciu malých štruktúr; zároveň sa predom rozmazá Gaussom $\sigma_{init} = 0.5$ (predpokladaná šumová sigma kamery).

Viac detailov: [[scale space and Gaussian pyramid]]

---

## Krok 2: *Difference of Gaussians (DoG)* — aproximácia LoG

*Laplasián Gaussiánu (Laplacian of Gaussian, LoG)* $\nabla^2 G$ je ideálny detektor „blobov" (guľatých štruktúr) — jeho odpoveď je maximálna tam, kde existuje koncentrovaná intenzita na správnej škále. LoG je však výpočtovo nákladný. SIFT ho aproximuje *Difference of Gaussians (DoG)*:

$$D(x, y, \sigma) = L(x, y, k\sigma) - L(x, y, \sigma)$$

Táto aproximácia vychádza z matematického vzťahu:

$$\frac{\partial G}{\partial \sigma} \approx \frac{G(x, y, k\sigma) - G(x, y, \sigma)}{k\sigma - \sigma}$$

teda $\sigma \nabla^2 G \approx \frac{D(x,y,\sigma)}{k-1}$, čo znamená DoG je (konštantne škálovanou) aproximáciou normalizovaného LoG. V každej oktáve vzniká teda $s + 2 = 5$ DoG obrazov odčítaním susedných Gaussovských obrazov.

Viac detailov: [[Difference of Gaussians]]

---

## Krok 3: Detekcia kľúčových bodov — extrémy v priestore škál

Po zostavení DoG pyramídy SIFT hľadá **lokálne extrémy (maxima a minima)** v trojrozmernom DoG priestore $(x, y, \sigma)$.

Každý pixel DoG obrazu sa porovná so svojimi **26 susedmi**:
- 8 susedov v rovnakej škálovej úrovni (v rovnakom DoG obraze)
- 9 susedov v DoG obraze o jednu škálu vyšší ($k\sigma$)
- 9 susedov v DoG obraze o jednu škálu nižší ($\sigma/k$)

Ak je hodnota pixelu väčšia alebo menšia ako všetkých 26 susedov, stáva sa **kandidátom na kľúčový bod**. Tento krok poskytuje hrubý odhad $(x, y, \sigma)$ pre každý potenciálny kľúčový bod — teda bod, kde existuje výrazný "blob" intenzity na danej škále.

> **Prečo DoG funguje ako detektor blobov?** Gaussovské vyhladenie potlačuje štruktúry menšie ako $\sigma$. DoG odčítaním zvýrazní štruktúry presne na škále $\sigma$ — oblasť kde je DoG extrémne pozitívna/negatívna zodpovedá svetlému/tmavému blobu o veľkosti $\sim 2\sigma$.

---

## Krok 4: Lokalizácia kľúčových bodov — Taylorov rozvoj

Extrémy nájdené v diskrétnom priestore sú len hrubé odhady. Pre **sub-pixelovú presnosť** SIFT fituje 3D kvadratickú funkciu pomocou Taylorovho rozvoja DoG $D(\mathbf{x})$ okolo kandidátneho bodu $\mathbf{x}_0 = (x_0, y_0, \sigma_0)^T$:

$$D(\mathbf{x}) \approx D(\mathbf{x}_0) + \frac{\partial D^T}{\partial \mathbf{x}} \Delta\mathbf{x} + \frac{1}{2} \Delta\mathbf{x}^T \frac{\partial^2 D}{\partial \mathbf{x}^2} \Delta\mathbf{x}$$

Deriváciou podľa $\Delta\mathbf{x}$ a položením výsledku na nulu dostaneme presnejšiu polohu extrému:

$$\Delta\hat{\mathbf{x}} = -\left(\frac{\partial^2 D}{\partial \mathbf{x}^2}\right)^{-1} \frac{\partial D}{\partial \mathbf{x}}$$

Ak je posun $|\Delta\hat{\mathbf{x}}| > 0.5$ v ktorejkoľvek dimenzii, kandidát sa presunie do susedného pixelu/škály a postup sa opakuje (max. 5 iterácií). Vďaka tomu sú kľúčové body lokalizované s presnosťou na zlomok pixelu aj v priestore škál.

---

## Krok 5: Filtrovanie — prahová hodnota kontrastu

Mnohé kandidátne extrémy ležia v oblastiach s nízkym kontrastom — sú nestabilné voči šumu a nemajú výpovednú hodnotu. Ak je interpolovaná hodnota $|D(\hat{\mathbf{x}})|$ pod prahom:

$$|D(\hat{\mathbf{x}})| < 0.03$$

kandidát sa **zahodí**. Táto hodnota 0.03 je Lowehov odporúčaný prah (v rozsahu hodnôt DoG normalizovaných do $[0,1]$). Výsledok: len výrazné, kontrastné kľúčové body prežijú.

---

## Krok 6: Eliminácia hrán — Hessián a pomer vlastných čísel

*DoG* extrémy sa vyskytujú aj pozdĺž hrán, kde je gradient v jednom smere silný, ale v kolmom smere slabý — taký kľúčový bod je ľahko destabilizovaný šumom a má zlú lokalizáciu pozdĺž hrany. Na elimináciu hranových bodov SIFT používa **Hessiánovu maticu** DoG v bode:

$$H = \begin{pmatrix} D_{xx} & D_{xy} \\ D_{xy} & D_{yy} \end{pmatrix}$$

kde $D_{xx}, D_{yy}, D_{xy}$ sú druhé parciálne derivácie DoG (aproximované konečnými diferencami). Vlastné čísla $H$ — označme ich $\alpha$ (väčšie) a $\beta$ (menšie) — udávajú zakrivenie DoG v dvoch hlavných smeroch:

- **Bod záujmu (roh, blob):** obe vlastné čísla sú veľké a podobné → $\alpha \approx \beta$
- **Hrana:** jedno vlastné číslo je omnoho väčšie → $\alpha \gg \beta$

Keďže priame výpočet vlastných čísel je zbytočne drahý, Lowe používa nasledujúci test cez stopu a determinant Hessiánu (stačia, bez výpočtu vlastných čísel):

$$\frac{\text{Tr}(H)^2}{\text{Det}(H)} = \frac{(\alpha + \beta)^2}{\alpha \beta} < \frac{(r+1)^2}{r}$$

kde $r = 10$ je prahový pomer vlastných čísel. Ak podmienka **nie je splnená** (pomer prekročí prah), bod je hranový a zahodí sa. Pre $r = 10$ je prah $\frac{11^2}{10} = 12.1$.

> **Intuícia:** $\frac{(\alpha+\beta)^2}{\alpha\beta}$ je minimálne pre $\alpha = \beta$ (hodnota 4) a rastie, čím viac sa $\alpha$ a $\beta$ líšia. Veľký pomer značí hranu.

---

## Krok 7: Priradenie orientácie — rotačná invariantnosť

Posledný krok prípravy kľúčového bodu je priradenie **dominantnej orientácie**, aby bol deskriptor invariantný voči rotácii obrazu.

Pre každý prežívajúci kľúčový bod SIFT:
1. Vezme okolie veľkosti $16\sigma \times 16\sigma$ pixelov v Gaussovsko rozmazanom obraze na škále $\sigma$ kľúčového bodu
2. V každom pixeli tohto okolia vypočíta gradient — veľkosť $m$ a smer $\theta$:
   $$m(x,y) = \sqrt{(L(x+1,y)-L(x-1,y))^2 + (L(x,y+1)-L(x,y-1))^2}$$
   $$\theta(x,y) = \text{atan2}(L(x,y+1)-L(x,y-1),\ L(x+1,y)-L(x-1,y))$$
3. Zostaví **36-binový histogram orientácií** (každý bin = 10°, rozsah 0–360°), pričom príspevky sú vážené veľkosťou gradientu a Gaussovským oknom so $\sigma_{window} = 1.5\sigma$
4. **Dominantný smer** (najvyšší peak histogramu) sa priradí ako orientácia kľúčového bodu
5. Ak existuje ďalší peak s výškou $\geq 80\%$ dominantného peaku, vytvorí sa **nový kľúčový bod** s rovnakou polohou a škálou, ale s touto sekundárnou orientáciou

Tento mechanizmus zabezpečuje, že deskriptor sa vždy počíta v súradnicovom systéme lokálnej orientácie, čím je invariantný voči ľubovoľnej rotácii obrazu.

Po kroku 7 má každý kľúčový bod plnú 4-ticu: $(x, y, s, \theta)$ — poloha, škála, orientácia. Nasleduje výpočet 128-rozmerného deskriptora (viď [[02 SIFT — descriptor and Generalized Hough Transform]]).

---

## Matematika — zhrnutie kľúčových vzorcov

**Gaussovský priestor škál:**
$$L(x, y, \sigma) = G(x, y, \sigma) * I(x,y), \quad G(x,y,\sigma) = \frac{1}{2\pi\sigma^2}e^{-(x^2+y^2)/(2\sigma^2)}$$

**Difference of Gaussians:**
$$D(x, y, \sigma) = L(x, y, k\sigma) - L(x, y, \sigma) \approx (k-1)\sigma^2 \nabla^2 L$$

**Taylorova lokalizácia — sub-pixelový posun:**
$$\Delta\hat{\mathbf{x}} = -\left(\frac{\partial^2 D}{\partial \mathbf{x}^2}\right)^{-1} \frac{\partial D}{\partial \mathbf{x}}$$

**Kontrastný prah:**
$$|D(\hat{\mathbf{x}})| \geq 0.03$$

**Hranový test (Hessián):**
$$\frac{\text{Tr}(H)^2}{\text{Det}(H)} < \frac{(r+1)^2}{r}, \quad r = 10 \;\Rightarrow\; \text{prah} = 12.1$$

**Faktor škály medzi intervalmi:**
$$k = 2^{1/s}, \quad s = 3 \;\Rightarrow\; k \approx 1.2599$$

---

## Porovnania a kompromisy

| Algoritmus | Invariantnosť | Rýchlosť | Presnosť | Licencia |
|---|---|---|---|---|
| **SIFT** | škála, rotácia, čiastočne perspektíva | pomalý (float) | veľmi vysoká | patentovaný (do 2020), teraz voľný |
| **SURF** | škála, rotácia | 3× rýchlejší ako SIFT | mierne nižšia | patentovaný |
| **ORB** | rotácia, čiastočne škála | 100× rýchlejší ako SIFT | nižšia | voľný (BSD) |
| **AKAZE** | nelineárny priestor škál | stredne rýchly | porovnateľná so SIFT | voľný |

**Kedy preferovať SIFT:**
- Vyžaduje sa vysoká presnosť párovania (SfM, medicínske snímky)
- Scéna má výrazné zmeny mierky alebo uhla pohľadu
- Výpočtový čas nie je kritický

**Kedy preferovať ORB:**
- Aplikácie reálneho času (robotika, AR na mobile)
- Obmedzená pamäť a výpočtový výkon

---

## Časté zlyhania a obmedzenia

- **Rozmazané obrazy (blur):** Silné rozmazanie odstraňuje textúru a gradienty — SIFT nenájde dostatok kľúčových bodov alebo ich deskriptory budú nezmyselné
- **Opakujúce sa vzory (repetitive textures):** Tapety, mriežky, dlažba — veľké množstvo zdanlivých zhôd (false positives), Lowehov pomer test pomáha, ale nedokáže eliminovať všetky
- **Nízky kontrast:** Homogénne oblasti (obloha, múr) neposkytujú stabilné extrémy; kontrastný prah 0.03 ich vyradí, ale vznikajú veľmi riedke zhody
- **Veľká rotácia mimo roviny (out-of-plane rotation):** SIFT je invariantný voči rotácii v rovine, ale pri výraznom naklonení (> 30°–45°) sa tvar blobov deformuje a deskriptory sa nepárujú
- **Výpočtová náročnosť:** SIFT je pomalý oproti binárnym deskriptorom (ORB, BRIEF) — nie je vhodný pre reálny čas bez GPU akcelerácie
- **Patentová história:** Patent US6711293 platil do 2020, čo obmedzovalo komerčné použitie; teraz je algoritmus voľne dostupný v `cv2.SIFT_create()`

---

## Praktické využitie

1. **Panoramatické spájanie snímok (image stitching):** OpenCV `cv2.Stitcher_create()` interne používa SIFT/ORB pre detekciu zodpovedajúcich bodov medzi snímkami a následne odhaduje homografiu cez RANSAC. SIFT zabezpečuje správne párovanie aj pri rôznych expozíciách a prekryvoch.

2. **3D rekonštrukcia zo snímok (Structure from Motion — SfM):** Softvér COLMAP a OpenMVG používajú SIFT ako štandardný deskriptor. SIFT zhody medzi stovkami fotografií umožňujú odhadnúť pozície kamier a 3D súradnice bodov scény.

3. **Rozpoznávanie objektov:** Lowehov pôvodný systém (demo z 2004) detegoval objekty v preplnených scénach kombináciou SIFT a Zovšeobecnenej Houghovej transformácie — v testovacom datasete dosahoval >95% detekčnú mieru.

4. **Registrácia medicínskych snímok:** MRI, CT a histologické snímky sa registrujú pomocou SIFT, pretože sú invariantné voči zmenám kontrastu a miernemu nakloneniu snímacieho zariadenia. Príklad: registrácia predoperačného MRI s intraoperačným ultrazvukom.

5. **AR aplikácie v múzeách a kultúrnom dedičstve:** Mobilné aplikácie (napr. Google Arts & Culture) identifikujú umelecké diela pohľadom kamery — SIFT alebo jeho nástupcovia párujem exponát s databázou referenčných snímok.

6. **Satelitné snímky (remote sensing):** Registrácia multitemporálnych satelitných snímok (z rôznych dátumov, senzorov) využíva SIFT pre sub-pixelovú presnosť, keďže scény sa líšia sezónnymi zmenami a rôznym nasvietením.

```python
import cv2

# Vytvorenie SIFT detektora
sift = cv2.SIFT_create(nfeatures=0, nOctaveLayers=3, 
                        contrastThreshold=0.04, edgeThreshold=10, sigma=1.6)

img = cv2.imread('image.jpg', cv2.IMREAD_GRAYSCALE)
keypoints, descriptors = sift.detectAndCompute(img, None)

print(f"Počet kľúčových bodov: {len(keypoints)}")
print(f"Tvar deskriptora: {descriptors.shape}")  # (N, 128)
```

---

## Súvisiace pojmy

[[scale space and Gaussian pyramid]], [[Difference of Gaussians]], [[Generalized Hough Transform]], [[Lowe ratio test]], [[02 SIFT — descriptor and Generalized Hough Transform]]

---

*Spýtaj sa sám seba:*

1. Prečo SIFT používa DoG namiesto priameho LoG?
2. Čo robí prahová hodnota kontrastu 0.03 a prečo eliminujeme hrany cez Hessian?
3. Ako dosahuje SIFT invariantnosť voči rotácii — v akom kroku?
4. Čo znamená, že kľúčový bod má "škálu" $\sigma$ a prečo je to dôležité?
5. Prečo potrebujeme $s + 3$ Gaussovských obrazov na oktávu namiesto $s$?
