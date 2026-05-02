# Generalized Hough Transform

*Zovšeobecnená Houghova transformácia (Generalized Hough Transform, GHT)* je technika hlasovania (*voting*), ktorá umožňuje detekovať a lokalizovať objekt ľubovoľného tvaru na základe lokálnych príznakov bez explicitného modelu celého objektu.

## Detaily

**Klasická Houghova transformácia:**
Pôvodná Houghova transformácia (1962) deteguje jednoduché tvary (čiary, kružnice) akumuláciou hlasov v parametrickom priestore. Napríklad pre čiary: každý hranový bod hlasuje za všetky čiary, na ktorých by mohol ležať — v (ρ, θ) priestore. Skutočná čiara vytvorí výrazný pík.

**Zovšeobecnenie (Ballard 1981):**
GHT rozširuje princíp na ľubovoľné tvary. Namiesto parametrizácie analytického tvaru sa používa *R-tabuľka* (*R-table*): pre každý bod na hranici vzoru sa uloží jeho relatívna poloha voči referenčnému bodu objektu (centroidu), *v súradniciach kľúčového bodu* (uhol + vzdialenosť).

**Ako funguje v SIFT detekcie objektov (Lowe 2004):**

1. **Fáza trénovania (model building):** Pre každý kľúčový bod $m$ vo vzore objektu sa uloží:
   - deskriptor $d_m$ (128D SIFT vektor)
   - *offset* $\Delta_m = (\Delta x_m, \Delta y_m, \Delta s_m, \Delta\theta_m)$ — relatívna poloha, mierka a orientácia centroidu objektu voči tomuto kľúčovému bodu

2. **Fáza detekcie (hlasovanie):** Pre každý kľúčový bod $q$ v testovacom obraze:
   - Nájdi nearest-neighbor v databáze deskriptorov (Lowe ratio test: $d_1 / d_2 < 0.8$)
   - Ak zhoda je prijatá: bod $q$ hlasuje v akumulátore za polohu centroidu:
     $$(\hat{x}_c, \hat{y}_c, \hat{s}_c, \hat{\theta}_c) = (x_q + s_q \cdot \Delta x_m, \; y_q + s_q \cdot \Delta y_m, \; s_q / s_m, \; \theta_q - \theta_m)$$

3. **Detekcia zhlukov:** Ak ≥ 3 hlasy konvergujú v akumulátore do malého regiónu → hypotéza detekcie objektu na danej polohe, mierke a orientácii.

4. **Verifikácia:** Affine least-squares fit: kľúčové body v zhlukovej hypotéze sa opatria affínou transformáciou — outlieri sa vylúčia, finálna transformácia sa overí.

**Výhody:**
- Robustnosť voči oklúzii (stačí niekoľko viditeľných kľúčových bodov)
- Robustnosť voči clutteru (náhodné zhody nevytvárajú konzistentný zhluk)
- Priamo dáva polohu, mierku a orientáciu detekovaného objektu

**Nevýhody:**
- Pamäťová a časová náročnosť akumulátora rastie s počtom DoF (poloha + mierka + rotácia = 4D)
- Falošné detekcie ak objekt má málo kľúčových bodov alebo silne opakujúci sa vzor

## Matematika

Hlasovanie jedného kľúčového bodu za polohu centroidu:
$$\text{vote}: \quad (x_c, y_c) = (x_q - \Delta x \cdot s_q,\; y_q - \Delta y \cdot s_q)$$

kde $(x_q, y_q)$ je poloha kľúčového bodu v obraze, $s_q$ je jeho mierka, $(\Delta x, \Delta y)$ je uložený offset zo vzoru.

## Kde sa používa

- [[02 SIFT — descriptor and Generalized Hough Transform]] — primárne použitie v Lowe 2004 object recognition
- Detekcia ľudí (head-shoulder template) pomocou GHT
- Robotické rozpoznávanie objektov (ak je objekt čiastočne zakrytý)
- Moderná alternatíva: RANSAC + geometrická verifikácia (menej pamäťovo náročné)

## Súvisiace pojmy

[[02 SIFT — descriptor and Generalized Hough Transform]], [[Lowe ratio test]], [[01 SIFT — scale space and keypoint detection]]
