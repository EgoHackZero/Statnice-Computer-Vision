# 10 Structure-from-Motion a Multi-View Stereo

> *Otázka: Structure-from-Motion, a Multiple View Stereo, proces dense 3D rekonštrukcie.*

## Stručný prehľad

*Structure-from-Motion (SfM)* je algoritmus, ktorý zo série **neusporiadaných fotografií** (bez nutnosti poznať polohu kamery) automaticky odhaduje 3D geometriu scény aj parametre všetkých kamier. Výsledkom je riedky *oblak bodov (sparse point cloud)* a sada camera poses. *Multi-View Stereo (MVS)* na to nadväzuje — zo znalosti presných pozícií kamier (z SfM) a overlapping pohľadov vypočíta **hustý oblak bodov (dense point cloud)** alebo priamo 3D model. Spolu tvoria základ modernej fotogrametrie a 3D rekonštrukcie, napájanej dnes softvérmi ako COLMAP, Meshroom, RealityCapture a Agisoft Metashape.

## Structure-from-Motion (SfM)

### Motivácia a vstup

- **Vstup:** séria fotografií tej istej scény z rôznych uhlov; poradie ani pozícia kamery nie sú známe
- **Výstup:** intrinsic matice $K_i$, extrinsic matice ($R_i, \mathbf{t}_i$) pre každý obraz + riedky 3D model
- **Kľúčový princíp:** ak viacero obrazov zobrazuje rovnaký fyzický bod, jeho 3D polohu možno triangulovať z ich projekcií

### Inkrementálny SfM pipeline (6 krokov)

**Krok 1: Detekcia 2D príznakov (*Feature Detection*)**
V každom obrázku sa detegujú záujmové body a vypočítajú ich deskriptory:
- SIFT — najrobustnejší, ale pomalší
- ORB — rýchly, voľne dostupný
- SuperPoint — deep learning-based detektor

**Krok 2: Párování príznakov (*Feature Matching*)**
Pre každý pár obrazov sa nájdu zodpovedajúce deskriptory:
- k-NN hľadanie (k=2) + [[Lowe ratio test]] ($d_1/d_2 < 0{,}8$) → akceptovateľné zhody
- RANSAC: odhad fundamentálnej matice $F$ z 8+ párov → outliere sa zahodia
- Zostávajú len **inlier** zhody konzistentné s epipólovou geometriou

**Krok 3: Tvorba trajektórií (*Track Formation*)**
Každý 3D bod je viditeľný vo viacerých obrazoch. Spojíme zodpovedajúce 2D detekcie naprieč obrazmi do *tracks*:
- Track $T_j = \{(i, \mathbf{m}_i^j)\}$ — zoznam (obraz $i$, pozorovaná 2D poloha) pre 3D bod $j$
- Dlhé tracks (viditeľné v mnohých obrazoch) sú cennejšie

**Krok 4: Inicializácia (počiatočný pár)**
Vyber sa dvojica obrazov s dostatočnou bázou a zhodami:
- Odhad $F$ → dekompozícia → $(R, \mathbf{t})$
- Triangulácia počiatočných 3D bodov

**Krok 5: Inkrementálne pridávanie kamier (*Incremental PnP*)**
Každý ďalší obraz sa pridáva:
- *PnP (Perspective-n-Point)* problém: z n zodpovedajúcich párov (3D bod ↔ 2D detekcia) odhad $(R, \mathbf{t})$ nového obrazu
- RANSAC na filtrovanie outlierov
- Triangulácia nových 3D bodov z nových zhôd

**Krok 6: Bundle Adjustment (BA)**
Finálna globálna optimalizácia — pozri nižšie.

### Bundle Adjustment

*Bundle adjustment* je nelineárna optimalizácia, ktorá **súčasne minimalizuje reprojekčnú chybu** cez všetky kamery a všetky 3D body:

$$E(P, M) = \sum_{j} \sum_{i \in V(j)} \left\| \pi(P_i, M_j) - \mathbf{m}_i^j \right\|^2$$

kde:
- $P_i$: parametre kamery $i$ (K, R, t)
- $M_j$: 3D súradnice bodu $j$
- $\pi(P_i, M_j)$: projekcia 3D bodu do obrazu $i$
- $\mathbf{m}_i^j$: skutočne pozorovaná 2D poloha bodu $j$ v obraze $i$
- $V(j)$: množina kamier, z ktorých je bod $j$ viditeľný

**Riešenie:** Levenberg-Marquardt iteratívna optimalizácia (kombinácia gradient descent a Gauss-Newton).

**Výsledok BA:**
- Redukuje akumulovanú chybu (*drift*) z inkrementálneho pridávania kamier
- Výstup: presne kalibrované kamery + riedky 3D mrak bodov

**Škálový problém:**
SfM produkuje rekonštrukciu len s *relatívnou* mierkou (meter vs. milimeter nie je rozlíšiteľný). Absolútna mierka vyžaduje:
- Znamú dĺžku v scéne (merací pás, šachovnica s rozmermi)
- GPS polohy kamier (aerial fotogrametria)
- IMU informácie

### Výstup SfM

- Intrinsic matice $K_i$ pre každú kameru (alebo spoločná $K$ ak rovnaká kamera)
- Extrinsic ($R_i, \mathbf{t}_i$) — pozícia a orientácia každej kamery v 3D priestore
- Riedky 3D point cloud (~10 000–1 000 000 bodov typicky)
- Tracks (zodpovedajúce body naprieč obrazmi)

## Multi-View Stereo (MVS)

### Čo je MVS a vzťah k SfM

*Multi-View Stereo (MVS)* nadväzuje na SfM — berúc jeho výstup (presné camera poses + intrinsics), vypočíta **husté korešpondencie** naprieč všetkými obrazmi a produkuje **hustý 3D model**.

- SfM: riedky výstup, odhad kamier
- MVS: hustý výstup, využíva kamery od SfM
- Výsledok MVS: **hustý oblak bodov** (milióny bodov) alebo 3D mesh

### Metódy MVS

**Patch-based MVS (PMVS, Furukawa & Ponce 2010):**
- Rastie 3D plochy (*patches*) z počiatočných bodov
- Každý patch = malá lokálna rovná plocha s normálou
- Iteratívna expanzia + filtrovanie

**Depth Map Fusion (COLMAP dense, PatchMatch Stereo):**
Najmodernejší a najúspešnejší prístup — podrobne nižšie.

**Deep learning MVS:**
- MVSNet (Yao et al. 2018): CNN odhaduje hĺbkovú mapu z feature cost volume
- TransMVSNet, UniMVSNet — ďalšie varianty

## Proces Dense 3D Rekonštrukcie — Depthmap Fusion

COLMAP (Schönberger & Frahm 2016) implementuje state-of-the-art pipeline:

### Fáza 1: Odhad hĺbkovej mapy per-obraz (*Per-View Depth Estimation*)

Pre každý obraz $i$ sa odhadne hustá hĺbková mapa $D_i(u, v)$:

**PatchMatch Stereo** (Bleyer et al. 2011):
1. **Inicializácia**: každý pixel dostane náhodný odhad hĺbky $d$ a normály $\mathbf{n}$
2. **Propagácia**: ak sused má dobrý odhad (malú foto-konzistentnú chybu), prevezmeme ho
3. **Random search**: skúšame nové náhodné odhady v okolí aktuálneho
4. **Iterácia**: opakujeme propagáciu + random search (2–5 iterácií)

**Foto-konzistencia (*photo-consistency*):**
Pre hypotézu $(d, \mathbf{n})$ bodu $(u, v)$ projektujeme okolie bodu do *k* iných obrazov a meriame podobnosť textúry (NCC alebo ZNCC). Silná foto-konzistencia = správna hĺbka.

Výsledok: hustá hĺbková mapa pre každý obraz (rovnaké rozlíšenie ako fotografia)

### Fáza 2: Fúzia hĺbkových máp (*Depth Map Fusion*)

Každý obraz produkuje svoju hĺbkovú mapu — treba ich skombinovať do jedného konzistentného point cloudu:

1. Pre každý pixel $(u, v)$ v obraze $i$ s hĺbkou $D_i(u, v)$:
   - Triangulácia → 3D bod $M$
   - Reprojektuj $M$ do všetkých iných obrazov $j$
   - Skontroluj **konzistencia**: hĺbka $M$ v obraze $j$ musí súhlasiť s $D_j$ (tolerancia: napr. ±1%)
   - Ak konzistentné v ≥ 2 iných obrazoch → bod je akceptovaný a pridaný do fúzovaného point cloudu
   - Nekonsistentné body → oklúzia alebo chyba → zahodiť

2. Pre akceptovaný bod: poloha v 3D = priemer jeho triangulácií z konzistentných obrazov

**Výsledok:** Hustý point cloud s miliónmi bodov + normálami

### Fáza 3: Rekonštrukcia povrchu (*Surface Reconstruction*)

Z hustého point cloudu vytvoriť polygon mesh:

**Poisson Surface Reconstruction (Kazhdan & Hoppe 2013):**
- Vstup: point cloud + normály každého bodu
- Rieši Poissonovu rovnicu: nájde indikátorovú funkciu $\chi$ (interior/exterior)
- Z izoplochy $\chi = 0{,}5$ extrahuje hladkú, watertight mesh
- Výhoda: hladký povrch, bez šumu; Nevýhoda: môže vyhladiť ostré hrany, extrapoluje plochy do prázdneho priestoru

**Screened Poisson (vylepšenie):**
- Pridáva podmienku, že mesh musí prechádzať *cez* vstupné body (nie len blízko nich)
- Lepší pre parciálne skeny (neúplné dáta)

**Marching Cubes:**
- Konvertuje voxelovú reprezentáciu (TSDF) na mesh
- Každý voxel-kub: 256 možných konfigurácií → lookup table → trojuholníky
- Výsledok závisí od rozlíšenia voxelov

### Fáza 4: Texturovanie (*Texturing*)

Premietnutie pôvodných fotografií na 3D mesh:
- UV mapa: každý trojuholník dostane súradnice v textúrovom atlase
- Z každého trojuholníka sa vyberie najlepší obraz (kolmý pohľad, dostatočné rozlíšenie)
- Blending viacerých obrazov → eliminuje švy
- Výsledok: foto-realistický textúrovaný 3D model

## Softvér a nástroje

| Softvér | Typ | Poznámka |
|---|---|---|
| **COLMAP** | Open-source | SfM + dense MVS; state-of-the-art; CLI + GUI |
| **Meshroom / AliceVision** | Open-source | GUI node-graph pipeline; priateľský pre začiatočníkov |
| **OpenSfM** | Open-source | Python; OpenDroneMap základ |
| **VisualSFM** | Freeware | Starší, ale stále používaný |
| **Agisoft Metashape** | Komerčný | Priemyselný štandard; drahý; výborná presnosť |
| **RealityCapture** | Komerčný | Najrýchlejší GPU-accelerated; populárny vo VFX |

## Matematika

**Reprojekčná chyba (jeden bod v jednom obraze):**
$$e_{ij} = \left\| \mathbf{m}_i^j - \pi(K_i, R_i, \mathbf{t}_i, M_j) \right\|$$

**Bundle adjustment celková cena:**
$$E = \sum_j \sum_{i \in V(j)} \rho\!\left(\left\| \pi(P_i, M_j) - \mathbf{m}_i^j \right\|^2\right)$$
kde $\rho$ je robustifikačná funkcia (Huber loss) na elimináciu outlierov.

**Triangulácia (lineárna metóda):**
Z dvoch projekcií $\mathbf{m}_1 = P_1 M$, $\mathbf{m}_2 = P_2 M$ (v homogénnych súradniciach):
$$\begin{bmatrix} \mathbf{m}_1 \times P_1 \\ \mathbf{m}_2 \times P_2 \end{bmatrix} M = 0 \quad \Rightarrow \quad \text{SVD} \to M$$

## Porovnania a kompromisy

**SfM vs. SLAM:**
| | SfM | SLAM |
|---|---|---|
| Čas spracovania | Offline (dávkové) | Online (reálny čas) |
| Presnosť | Vyššia (BA cez všetky snímky) | Nižšia (inkrementálna) |
| Aplikácia | Fotogrametria, kultúrne dedičstvo | Robotika, AR |
| Vstup | Neusporiadaná sada fotografií | Sekvencia snímok v čase |

**MVS dense vs. LiDAR:**
- MVS dense: lacnejší (len kamery), ale citlivý na textúreless plochy, slnko, reflexie
- LiDAR: priame meranie, spoľahlivé za každého počasia, ale drahší senzor

## Časté zlyhania a obmedzenia

- **Textureless plochy** (biela stena, voda): SfM nenájde príznaky → medzery v rekonštrukcii
- **Pohybujúce sa objekty** (chodci, listy) → nekonzistentné zhody → outliere
- **Šachovnicový vzor alebo symetria** → nejednoznačná orientácia kamier
- **Málo overlapping** (< 60% overlap medzi snímaniam) → prerušené tracks → fragmentovaný model
- **Mierka**: bez referencie absolútna mierka neznáma

## Praktické využitie

1. **Fotogrametria pre architektonické dedičstvo** — 3D scan katedrál, sôch, archeologických nálezísk (Scan the World, Getty Conservation, UNESCO World Heritage sites)
2. **Filmový VFX a herný priemysel** — digitálne dvojníky hercov, scény (Industrial Light & Magic, Weta Digital používajú MVS pre photo-realism)
3. **Mapovanie z UAV** — ortofotomapy, DEM/DSM z dronou fotogrametrie; OpenDroneMap, Metashape; pôdohospodárstvo, civilná ochrana
4. **HD mapy pre autonómne vozidlá** — centimetrová presnosť ciest, semaforov, značiek; HERE Maps, TomTom, Google Street View
5. **Stomatológia a chirurgia** — 3D model zubov z intraorálneho skenera; ortopédia — 3D model kostí z RTG
6. **E-commerce 3D produkty** — sken produktu zo série fotografií → interaktívny 3D model na webe (IKEA Place app)

## Súvisiace pojmy

[[bundle adjustment]], [[dense MVS and depthmap fusion]], [[Lowe ratio test]], [[09 Problém korešpondencie, disparita a hĺbka]], [[07 2.5D, 3D a volumetrické reprezentácie]], [[point cloud]], [[polygon mesh]]

---
*Spýtaj sa sám seba:*
1. Čo je bundle adjustment a prečo je nevyhnutný v SfM pipeline?
2. Prečo SfM produkuje len riedky output a ako MVS ho zhustí?
3. Aký je rozdiel medzi SfM a SLAM?
4. Čo je Poisson Surface Reconstruction a z akého vstupu pracuje?
5. Prečo SfM zlyhá na textureless plochách a pohyblivých objektoch?
