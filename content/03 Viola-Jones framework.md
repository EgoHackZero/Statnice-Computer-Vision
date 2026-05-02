# 03 Viola-Jones Framework — *Rámec Viola-Jones*

> *Otázka: Frameworku Viola-Jones, integrálny obrazu, Haarove črty, adaptívny boosting, attentional cascade?*

## Stručný prehľad

*Framework Viola-Jones* (Viola & Jones, 2001) je prelomový systém pre detekciu tvárí v reálnom čase. Kombinuje štyri kľúčové myšlienky: jednoduché *Haarove vlnkové príznaky (Haar-like features)*, *integrálny obraz (integral image)* pre ich rýchly výpočet, *AdaBoost* pre výber najdôležitejších príznakov a ich kombináciu do silného klasifikátora, a *kaskádu klasifikátorov (attentional cascade)* pre rýchle odmietnutie nezaujímavých regiónov. Výsledkom je detektor, ktorý dokáže v reálnom čase spracovať snímky z kamery s vysokou presnosťou — v roku 2001 bol prvýkrát použiteľný na bežnom PC bez GPU.

## Haarove vlnkové príznaky (*Haar-like features*)

*Haarové črty* sú jednoduché obdĺžnikové príznaky inšpirované Haarovými vlnkami. Každý príznak sa vypočíta ako **rozdiel súčtov intenzít pixelov** v dvoch alebo viacerých susedných obdĺžnikoch.

**Typy Haarových čŕt:**
- **Dvojobdĺžnikové** (*two-rectangle*): biely − čierny obdĺžnik; detegujú horizontálne alebo vertikálne hrany a prechody intenzít (napr. svetlejší čelo nad tmavšími očami)
- **Trojobdĺžnikové** (*three-rectangle*): biely − čierny − biely; detegujú línie (napr. hrebeň nosa medzi tmavšími očnicami)
- **Štvtrobdĺžnikové** (*four-rectangle*): detegujú diagonálne prechody (napr. rohové štruktúry)

**Počet príznakov:**
V detekciovom okne 24 × 24 pixelov možno umiestniť príznaky v rôznych polohách, veľkostiach a orientáciách. Viola a Jones spočítali viac ako **160 000 kandidátnych príznakov** — oveľa viac, ako by bolo možné vyhodnocovať naraz.

**Prečo práve Haarové černty?**
Haarové príznaky nie sú náhodné — zodpovedajú vizuálnym charakteristikám tváre:
- Oblasť očí je tmavšia ako líca (dvojobdĺžnikový príznak zachytáva tento kontrast)
- Hrebeň nosa je svetlejší ako okolité tienisté oblasti
- Oblasť úst má tmavší priebeh ako brada

Jednotlivý príznak je slabý — nevie sám detekovať tvár. AdaBoost ich ale kombinuje.

## Integrálny obraz (*Integral Image*)

Priame vypočítanie súčtu pixelov v obdĺžniku o veľkosti $w \times h$ by trvalo $O(wh)$ operácií. Pri 160 000 príznakoch na každé zo stoviek tisíc okien v obraze by to bolo výpočtovo neúnosné. **Integrálny obraz** to rieši.

**Definícia:**
$$II(x, y) = \sum_{x' \leq x,\; y' \leq y} I(x', y')$$

Hodnota integrálneho obrazu v bode $(x, y)$ je súčet všetkých pixelov *vľavo hore* od tohto bodu (vrátane). Vypočíta sa jedným prechodom obrazom v čase $O(WH)$ (kde $W, H$ sú rozmery obrazu).

**Súčet obdĺžnika v $O(1)$:**
Súčet pixelov v obdĺžniku s rohmi $A$ (ľavý horný), $B$ (pravý horný), $C$ (ľavý dolný), $D$ (pravý dolný):

$$\text{Sum}(ABCD) = II(D) + II(A) - II(B) - II(C)$$

len 4 odčítania — **konštantný čas bez ohľadu na veľkosť obdĺžnika**. Každý Haarov príznak (rozdiel dvoch obdĺžnikov) = ~6–9 operácií.

**Výsledok:** Vďaka integrálnemu obrazu možno vyhodnotiť jeden Haarov príznak v konštantnom čase, čo umožňuje reálno-časové skenovanie tisícok okien.

## AdaBoost — Adaptívny boosting

*AdaBoost (Adaptive Boosting)* slúži na **výber najdôležitejších príznakov** spomedzi 160 000 kandidátov a ich kombináciu do **silného klasifikátora**.

**Postup trénovania AdaBoost:**

1. **Inicializácia:** každej trénovacej vzorke (tvár / netvár) sa priradí rovnaká váha $w_i = 1/N$.
2. **Iterácia** (pre $t = 1, 2, \ldots, T$):
   - Vyber *slabý klasifikátor* $h_t$ — jeden Haarov príznak s optimálnym prahom — ktorý minimalizuje váhovanú chybu $\varepsilon_t = \sum_i w_i \cdot \mathbf{1}[h_t(x_i) \neq y_i]$.
   - Vypočítaj váhu klasifikátora: $$\alpha_t = \frac{1}{2} \ln\!\left(\frac{1 - \varepsilon_t}{\varepsilon_t}\right)$$
   - Aktualizuj váhy vzoriek — zväčši váhy nesprávne klasifikovaných: $$w_i^{(t+1)} = w_i^{(t)} \cdot e^{-\alpha_t y_i h_t(x_i)}$$, potom normalizuj.
3. **Silný klasifikátor:** $$H(x) = \text{sign}\!\left(\sum_{t=1}^{T} \alpha_t h_t(x)\right)$$

**Výsledok:** Viola-Jones vyberá ~200 najlepších príznakov z 160 000. Každý z nich je jednoduchý (jeden Haarov príznak s prahom), ale ich váhovaná kombinácia dosiahne vysokú presnosť klasifikácie tvár/netvár.

**Prečo AdaBoost funguje:**
Reweighting sústredí ďalšie iterácie na *ťažké príklady* — vzorky, ktoré predchádzajúce slabé klasifikátory nevedeli správne zaradiť. Postupne sa buduje klasifikátor, ktorý zvláda aj zložité prípady.

## Kaskáda klasifikátorov (*Attentional Cascade*)

V reálnom obraze je drvivá väčšina okien *netváre* — jednoduché negatíva. Bolo by plytvanie aplikovať plný 200-príznakový klasifikátor na každé okno. **Kaskáda** to rieši.

**Princíp:**
Klasifikátory sú usporiadané do *kaskády* — séria stupňov (*stages*) s rastúcou zložitosťou:
- **Stupeň 1:** 2–5 príznakov; veľmi rýchly; navrhnutý tak, aby zamietol ~50 % netvárovych okien
- **Stupeň 2:** 10–15 príznakov; aplikuje sa len na okná, ktoré prešli stupňom 1
- **Stupeň $k$:** viac príznakov, pomalší, aplikovaný na čoraz menej okien
- **Posledný stupeň:** 200+ príznakov; len pre okná, ktoré prešli všetkými predchádzajúcimi

**Dôležité vlastnosti každého stupňa:**
- Navrhnutý pre **vysokú citlivosť (recall ~100 %)** — nesmie zamietnuť reálnu tvár
- Akceptuje veľa falošne pozitívnych (false positives) — tie odfiltrujú ďalšie stupne
- Okno odmietnuté v *ktoromkoľvek* stupni → okamžite klasifikované ako netvár

**Výsledok:**
Väčšina okien (napr. 95 %) je zamietnutá už po stupni 1 (2 príznaky). Len malá časť okien preside celou kaskádou — tá je potom starostlivo skúmaná. Priemerný počet príznakov vyhodnotených na jedno okno je rádovo menší ako 200.

Viola a Jones dosiahli spracovanie obrazu 384 × 288 pixelov za ~0,067 sekundy na CPU z roku 2001 — čas reálnej detekcie.

## Matematika

**Integrálny obraz:**
$$II(x, y) = \sum_{x' \leq x} \sum_{y' \leq y} I(x', y')$$
Súčet obdĺžnika: $\text{Sum} = II(D) + II(A) - II(B) - II(C)$

**AdaBoost váha klasifikátora:**
$$\alpha_t = \frac{1}{2} \ln\!\frac{1 - \varepsilon_t}{\varepsilon_t}$$

**Silný klasifikátor:**
$$H(x) = \text{sign}\!\left(\sum_{t=1}^{T} \alpha_t h_t(x)\right)$$

**Aktualizácia váh:**
$$w_i \leftarrow w_i \cdot e^{-\alpha_t y_i h_t(x_i)}, \quad \text{potom normalizovať na } \sum w_i = 1$$

## Porovnania a kompromisy

| Vlastnosť | Viola-Jones (Haar cascade) | HOG + SVM | Deep CNN (MTCNN, RetinaFace) |
|---|---|---|---|
| Rýchlosť | Veľmi rýchla (CPU reálny čas) | Stredná | Pomalšia (GPU) |
| Presnosť | Dobrá pre frontálne tváre | Lepšia pre chodcov | Najlepšia |
| Profil / rotácia | Slabá | Lepšia | Výborná |
| Citlivosť na oklúziu | Slabá | Stredná | Dobrá |
| Tréning | Jednoduchý | Jednoduchý | Komplexný, veľa dát |

## Časté zlyhania a obmedzenia

- **Profil a natočená tvár:** Viola-Jones je trénovaný hlavne na frontálne tváre; profil tvár (> 45°) sa nedeteguje
- **Oklúzia:** Zakrytie čela, očí alebo nosa znižuje odozvu kaskády
- **Silné tiene a nerovnomerné osvetlenie:** Kontrasty Haarových čŕt sú závislé od osvetlenia
- **Dospievanie, masky, slnečné okuliare:** Menia vzhľad Haarových príznakov
- **Orientácia v obraze:** Kaskáda je invariantná voči mierke (viacnásobné skenovanie), ale nie voči rotácii v rovine obrazu — bežná implementácia deteguje len tváre ±30°

## Praktické využitie

1. **Detekcia tváre v kamerách a fotoaparátoch** — väčšina digitálnych fotoaparátov (Canon, Nikon, Sony) integruje Viola-Jones pre automatické zaostrovanie na tvár; implementácia priamo na čipe
2. **OpenCV Haar kaskáda** — `cv2.CascadeClassifier('haarcascade_frontalface_default.xml')`; detekcia v reálnom čase na CPU; dostupná pre tvár, oči, ústa, plné telo
3. **Bezpečnostné kamery a CCTV** — detekcia prítomnosti osôb v záberoch; trigger pre nahrávanie
4. **Robotická interakcia** — detekcia tváre pre orientáciu robota k používateľovi (napr. Pepper robot)
5. **Mobilné aplikácie AR** — historicky Snapchat a podobné aplikácie pred érou deep learningu

**Knižnice a nástroje:**
- `OpenCV`: `cv2.CascadeClassifier`, `cv2.detectMultiScale()`
- Dostupné predtrénované kaskády: tvár, oči, uši, nos, ústa, celé telo, horná časť tela
- Tréning vlastnej kaskády: `opencv_traincascade` s vlastnými pozitívnymi/negatívnymi vzorkami

## Súvisiace pojmy

[[integral image]], [[Haar-like features]], [[AdaBoost]], [[attentional cascade]], [[04 SURF — Speeded-Up Robust Features]]

---
*Spýtaj sa sám seba:*
1. Čo je integrálny obraz a ako umožňuje výpočet súčtu obdĺžnika v $O(1)$?
2. Prečo AdaBoost používa reweighting príkladov po každej iterácii?
3. Aké typy Haarových čŕt existujú a aký vizuálny príznak tváre zachytáva každý z nich?
4. Čo sa stane s oknom, ktoré neprejde stupňom 1 kaskády?
5. Prečo Viola-Jones zle deteguje tváre z profilu?
