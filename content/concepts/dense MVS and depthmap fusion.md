# Dense MVS and Depthmap Fusion

*Dense Multi-View Stereo (MVS)* je proces výpočtu hustého 3D modelu zo série fotografií so známymi kamerovými pozíciami (typicky z SfM). Kľúčovým algoritmom je *depthmap fusion* — postup COLMAP (Schönberger & Frahm 2016).

## Fáza 1: Per-View Depth Estimation

Pre každý obraz $i$ sa odhadne hustá hĺbková mapa $D_i(u, v)$ pomocou **PatchMatch Stereo**.

### PatchMatch Stereo

Iteratívny algoritmus inšpirovaný PatchMatch (Barnes et al. 2009):

**Krok 1 — Inicializácia:**
Každý pixel $(u, v)$ dostane **náhodný** odhad hĺbky $d$ a normály $\mathbf{n}$ (náhodný unit vektor).

**Krok 2 — Foto-konzistencia (Photo-Consistency):**
Pre hypotézu $(d, \mathbf{n})$ pixela $(u, v)$:
1. Pomocou $d$ a $\mathbf{n}$ definujeme malý *patch* (rovinnú plochu) v 3D
2. Projektujeme okolie pixela do $k$ iných kompatibilných obrazov (zdieľajú pohľad)
3. Meriame podobnosť textúr pomocou **NCC** (Normalized Cross-Correlation) alebo ZNCC
4. Foto-konzistencia $\phi(d, \mathbf{n}) = $ priemerné NCC cez všetky projektované obrazy

Silná foto-konzistencia (blízko 1) = správna hĺbka a normála.

**Krok 3 — Propagácia:**
Ak sused pixela $(u-1, v)$ má lepší odhad (vyššie NCC), prevezmeme jeho $(d, \mathbf{n})$. Predpoklad: povrchy sú lokálne hladké → susedné pixely majú podobnú hĺbku.

**Krok 4 — Random Search:**
Testujeme náhodné odchýlky od aktuálneho odhadu v zmenšujúcich sa intervaloch → lokálne spresňovanie.

**Iterácia:** Striedame propagáciu (raster scan dopredu + dozadu) a random search ~3–5 krát.

**Výsledok:** Hustá hĺbková mapa s rovnakým rozlíšením ako vstupný obraz.

---

## Fáza 2: Depth Map Fusion

Každý obraz produkuje svoju hĺbkovú mapu. Fúzia ich zlúči do jedného konzistentného point cloudu.

### Postup fúzie

Pre každý pixel $(u, v)$ v obraze $i$ s hĺbkou $D_i(u, v)$:

1. **Triangulácia:** vypočítaj 3D bod $\mathbf{M}$ z pixelu $(u, v)$ a $D_i(u, v)$
2. **Reprojekcia:** projektuj $\mathbf{M}$ do všetkých ostatných obrazov $j$
3. **Konzistencia:** skontroluj, či hĺbka bodu $\mathbf{M}$ v obraze $j$ súhlasí s $D_j$:
   $$\frac{|z_j - D_j(u_j, v_j)|}{z_j} < \tau \quad (\text{typicky } \tau = 0{,}01)$$
4. **Akceptácia:** ak konzistentné v ≥ 2 ďalších obrazoch → bod je dôveryhodný, pridá sa do výstupného cloudu
5. **Zlúčenie:** poloha bodu = priemer triangulácií zo všetkých konzistentných pohľadov

Nekonsistentné body (oklúzia, chyba matchingu) → zahodiť.

**Výsledok:** Hustý point cloud s miliónmi bodov + normálami.

---

## Fáza 3: Surface Reconstruction

Z hustého point cloudu sa vytvorí polygon mesh.

### Screened Poisson Surface Reconstruction

$$\Delta \chi = \nabla \cdot \mathbf{V}$$

kde $\mathbf{V}$ je vektorové pole definované normálami vstupných bodov. Rieši sa pomocou oktree a konjugovaných gradientov.

*Screened* varianta (Kazhdan & Hoppe 2013) pridáva podmienku, že mesh musí prechádzať **cez** vstupné body (nie len blízko nich) → lepšie pri neúplných dátach.

### Marching Cubes

Alternatíva: konvertuje TSDF voxelovú reprezentáciu na mesh (256 možných konfigurácií každého voxelového kubu → lookup table trojuholníkov).

---

## Fáza 4: Texturovanie

1. **UV unwrapping:** každý trojuholník dostane súradnice v textúrovom atlase
2. **Výber obrazu:** pre každý trojuholník vyber najlepší pohľad (kolmý + dostatočné rozlíšenie)
3. **Blending:** eliminovanie švov medzi pohľadmi
4. **Output:** fotorealistický textúrovaný 3D model

---

## Softvér

| Softvér | MVS algoritmus |
|---|---|
| **COLMAP** | PatchMatch + depthmap fusion |
| **Meshroom (AliceVision)** | DepthMap + MVS fusion |
| **Agisoft Metashape** | Proprietárny dense matching |
| **OpenMVS** | PatchMatch Stereo |

---

## Zlyhania MVS

- **Textureless plochy** (biela stena): PatchMatch nedokáže určiť foto-konzistentný odhad
- **Lesklé povrchy:** reflexie menia výzor z rôznych uhlov → nekonzistentné
- **Oklúzia:** bod viditeľný len z jedného pohľadu → zamietnutý fúziou
- **Malý overlap:** < 60% → nedostatok konzistentných pohľadov

## Súvisiace pojmy

[[10 Structure-from-Motion a Multi-View Stereo]] · [[bundle adjustment]] · [[point cloud]] · [[polygon mesh]] · [[voxel grid]]
