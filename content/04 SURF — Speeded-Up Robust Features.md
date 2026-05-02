# 04 SURF — *Speeded-Up Robust Features*

> *Otázka: Popis metódy Speeded-up Robust Features.*

## Stručný prehľad

*SURF (Speeded-Up Robust Features)* je algoritmus na detekciu a popis lokálnych príznakov navrhnutý Bayom, Essom, Van Goola a Tuytelaarsom (2006) ako rýchlejšia alternatíva k SIFT. SURF nahrádza výpočtovo náročné gaussovské konvolúcie *box filtrami* aplikovanými cez *integrálny obraz (integral image)*, čím dosahuje detekciu kľúčových bodov nezávislú od ich mierky v konštantnom čase. Deskriptor je 64-rozmerný (oproti 128D v SIFT), čo urýchľuje párování a zachováva dostatočnú rozlišovacia schopnosť pre väčšinu aplikácií. SURF je škálovanie aj rotačne invariantný, čo ho robí robustným voči bežným zmenám pohľadu.

## SURF vs. SIFT — Kľúčové rozdiely

| Vlastnosť | SIFT | SURF |
|---|---|---|
| **Detektor** | DoG (Difference of Gaussians) | Determinant Hessiánu (det H) |
| **Aproximácia** | LoG cez rozdiel Gaussiánov | Gaussove druhé derivácie cez box filtre |
| **Deskriptor** | 128D gradient histogramy | 64D Haar-vlnkové odozvy |
| **Orientácia** | 36-bin gradient histogram | Haar-vlnky v kruhovom okolí |
| **Rýchlosť** | Referenčná | ~3–7× rýchlejší |
| **Mierková invariantnosť** | Áno | Áno |
| **Rotačná invariantnosť** | Áno | Áno (aj U-SURF bez rotácie) |
| **Robustnosť** | Lepšia pri extrémnych zmenách | Mierne slabšia |

## Detekcia kľúčových bodov — Hessian blob detektor (*Fast Hessian*)

SURF deteguje kľúčové body ako **lokálne maximá determinantu Hessiánovej matice** v priestore mierok.

**Hessianova matica:**
$$H(x, y, \sigma) = \begin{pmatrix} L_{xx}(x,y,\sigma) & L_{xy}(x,y,\sigma) \\ L_{xy}(x,y,\sigma) & L_{yy}(x,y,\sigma) \end{pmatrix}$$

kde $L_{xx}, L_{yy}, L_{xy}$ sú druhé parciálne derivácie obrazu konvolvovaného Gaussianom so σ.

**Determinant:**
$$\det(H) = L_{xx} L_{yy} - L_{xy}^2$$

Veľký $\det(H)$ → bod má silnú krivosť v dvoch smeroch → *blob* → kandidát na kľúčový bod.

**Aproximácia box filtrami:**
SURF approximuje $L_{xx}, L_{yy}, L_{xy}$ jednoduchými obdĺžnikovými filtrami (napr. 9 × 9 pixelov pre základnú mierku). Tieto filtre sú aplikované cez [[integral image]] — výsledok: každá odozva filtra sa vypočíta v **$O(1)$**, nezávisle od jeho veľkosti.

Pre väčšie mierky sa používajú väčšie filtre (15×15, 21×21, 27×27...) — stále $O(1)$.

**Príznak znamienka Laplaciánu (*sign of Laplacian*):**
SURF tiež zaznamenáva znamienko stopy Hessiánovej matice $\text{Tr}(H) = L_{xx} + L_{yy}$ (čo je diskrétna aproximácia LoG). Pri párovaní sa porovnávajú len body s rovnakým znamienkom — svetlá škvrna na tmavom pozadí (kladné Tr) sa nikdy nespáruje s tmavou škvrnou na svetlom (záporné Tr). Toto urýchľuje vyhľadávanie pri zachovaní presnosti.

## Priradenie orientácie (*Orientation Assignment*)

1. Vypočítaj Haar-vlnkové odozvy $d_x$ a $d_y$ v kruhovom okolí polomeru $6\sigma$ okolo kľúčového bodu.
2. Váhuj odozvy Gaussiánom so stredom v kľúčovom bode.
3. Posúvaj okno 60° pozdĺž kružnice; spočítaj vektorový súčet $(\Sigma d_x, \Sigma d_y)$ v každom okne.
4. Smer okna s najväčším súčtom → orientácia kľúčového bodu.

**U-SURF (Upright SURF):**
Variant bez výpočtu orientácie — rýchlejší, vhodný keď sú objekty vždy vzpriamené (napr. letecké snímky, dokumenty). Orientácia je predpokladaná ako 0°.

## Deskriptor kľúčového bodu (*64D Haar Wavelet Descriptor*)

1. Okolo kľúčového bodu sa vyberie štvorcové okno **20σ × 20σ**, otočené podľa priradené orientácie.
2. Rozdelí sa na mriežku **4 × 4 podregióny** (každý 5σ × 5σ).
3. V každom podregióne sa vypočítajú Haar-vlnkové odozvy v smeroch $d_x$ (horizontálny) a $d_y$ (vertikálny) pre všetky pixely podregiónu:
   $$\mathbf{v}_{subregion} = \left(\sum d_x,\; \sum d_y,\; \sum |d_x|,\; \sum |d_y|\right)$$
   — 4-rozmerný vektor na podregrión.
4. Konkatenácia 16 podregiónov: $16 \times 4 = \mathbf{64}$ dimenzií.
5. Normalizácia na jednotkovú dĺžku (L2-norma).

**Čo každá zložka zachytáva:**
- $\sum d_x$, $\sum d_y$: prevládajúci gradient (smer intenzitnej zmeny)
- $\sum |d_x|$, $\sum |d_y|$: celková energia gradientu bez ohľadu na smer (robustnosť voči zrkadlovým vzorcom)

**Invariantnosť:**
- *Mierková invariantnosť*: Okno 20σ × 20σ sa škáluje s mierkou kľúčového bodu
- *Rotačná invariantnosť*: Okno je otočené podľa dominantnej orientácie
- *Odolnosť voči osvetleniu*: Normalizácia na jednotkovú dĺžku

## Matematika

**Determinant Hessiánu (s korekciou váhového faktora):**
$$\det(H_{approx}) = D_{xx} D_{yy} - (0{,}9 \cdot D_{xy})^2$$

kde $D_{xx}, D_{yy}, D_{xy}$ sú odozvy box-filter aproximácií; faktor 0,9 kompenzuje chybu approximácie $L_{xy}$.

**Ratio test (Lowe):** 
$$\frac{d_1}{d_2} < 0{,}8 \Rightarrow \text{zhoda akceptovaná}$$

(pozri [[Lowe ratio test]])

**Haar-vlnková odozva v $O(1)$:**
Horizontálna odozva $d_x$ v bode $(x, y)$: súčet pravého poloblochu mínus súčet ľavého → 2 operácie s integrálnym obrazom.

## Porovnania a kompromisy

**SURF vs. ORB:**
- ORB (Oriented FAST + Rotated BRIEF) je voľne dostupný (SURF bol patentovaný), rýchlejší, ale menej robustný pri silných zmenách mierky
- SURF je lepší pre vysokú presnosť bez GPU

**U-SURF vs. SURF:**
- U-SURF: 3× rýchlejší, bez orientácie → vhodný pre upright scény, letecké snímky
- SURF: robustnejší pri rotácii objektov

**64D vs. 128D deskriptor:**
- SURF-128: rozšírená verzia SURF s 128D deskriptorom (zachytáva aj znamienka $d_x, d_y$); porovnateľná rozlišovacia schopnosť so SIFT

## Časté zlyhania a obmedzenia

- **Silné zmeny pohľadu** (*viewpoint changes*): SURF ako lokálne-afinne-invariantný detektor zlyháva pri veľkých perspektívnych deformáciách (> 50° otočenie kamery)
- **Silný rozmazanie** (*motion blur*): degraduje Hessianovu odozvu
- **Textúreless oblasti**: žiadne kľúčové body (rovnako ako SIFT)
- **Patent**: SURF je chránený patentom — v komerčnom softvéri je potrebná licencia; alternatívy: SIFT (GPU implementácia), ORB, AKAZE (zdarma)
- **Opakujúce sa vzory**: problémy s párovaním (rovnako ako SIFT)

## Praktické využitie

1. **Sledovanie objektov v reálnom čase** — `cv2.xfeatures2d.SURF_create()` v OpenCV; detekcia + popis + BFMatcher pre sledovanie v 30+ fps
2. **Panoramatické sčítanie obrazov** — Panorama stitching na mobiloch a kamerách (pred érou deep feature extrakcie)
3. **Porovnávanie produktov v e-commerce** — Rozpoznanie produktov podľa obalu/etikety z rôznych uhlov
4. **Rozšírená realita (AR) na mobilných zariadeniach** — Sledovanie planá obrazca (marker tracking) v reálnom čase
5. **Satelitné snímky** — Registrácia multi-temporálnych snímok (rôzne dátumy snímania rovnaký región)

**Poznámka k dostupnosti v OpenCV:**
SURF je dostupný iba v `opencv-contrib` (nie v základnom opencv-python) a vyžaduje `-D OPENCV_ENABLE_NONFREE=ON` pri kompilácii alebo instaláciu `opencv-contrib-python-headless`. Pre produkčné systémy odporúčame ORB alebo AKAZE (bezplatné alternatívy).

## Súvisiace pojmy

[[Hessian matrix and blob detection]], [[integral image]], [[Lowe ratio test]], [[scale space and Gaussian pyramid]], [[01 SIFT — scale space and keypoint detection]]

---
*Spýtaj sa sám seba:*
1. Aký detektor kľúčových bodov používa SURF a prečo je rýchlejší ako DoG v SIFT?
2. Čo je U-SURF a kedy ho použiť namiesto štandardného SURF?
3. Ako sa konštruuje 64D SURF deskriptor? Čo zachytáva každá zo 4 hodnôt podregiónu?
4. Čo je znamienko Laplaciánu v SURF a na čo slúži pri párovaní?
5. Prečo je SURF patentovaný a aké sú voľne dostupné alternatívy?
