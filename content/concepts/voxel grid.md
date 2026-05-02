# Voxel Grid

*Voxel grid* je trojrozmerná pravidelná mriežka, kde každá bunka (voxel) reprezentuje kváder priestoru a nesie hodnotu (obsadenosť, farbu, TSDF, hustotu,...).

*Voxel* = volumetrický pixel = najmenšia jednotka 3D diskretizovaného priestoru.

## Typy voxelových reprezentácií

### Binárna mriežka (Occupancy Grid)

Každý voxel: `0` (voľný) alebo `1` (obsadený)

- Jednoduchá, priamočiara
- Pamäť: $N^3$ bitov pre mriežku $N \times N \times N$
- Pre $N=256$: 16 MB; pre $N=512$: 128 MB

### TSDF (Truncated Signed Distance Function)

Každý voxel uchováva vzdialenosť k najbližšiemu povrchu (s znamienkom: negatívne = vnútro, pozitívne = vonkajšok), skrátená na $[-t, +t]$:

$$TSDF(v) = \text{clip}\!\left(\frac{d(v)}{\mu}, -1, 1\right)$$

kde $d(v)$ je vzdialenosť voxela $v$ od povrchu, $\mu$ je truncation distance.

**Výhody TSDF:**
- Povrch je implicitne definovaný izoplochu $TSDF = 0$
- Umožňuje inkrementálnu fúziu z viacerých depth máp (KinectFusion, TSDF Fusion)
- Extrahovateľný povrch cez Marching Cubes

### Probability/Occupancy Map

Každý voxel má pravdepodobnosť obsadenosti — aktualizuje sa Bayesovskou fúziou nových meraní. Používa sa v robotickej navigácii (log-odds update rule).

## Výhody voxelovej reprezentácie

- **Pravidelná štruktúra** → efektívne 3D konvolúcie (3D CNN, VoxNet)
- **Topologická konzistencia** → jednoduchá detekcia kolízií
- **Implicitná reprezentácia** → povrch nie je explicitný, interpoluje sa
- **Fúzia dát** → jednoduché zlúčenie viacerých meraní (TSDF)

## Nevýhody

- **Pamäťová náročnosť:** $O(N^3)$ — kubický rast. Pre $N = 1024$ a float32: 4 GB
- **Plytvanie:** väčšina voxelov vo voľnom priestore je prázdnych
- **Riešenie:** Octree / Sparse Voxel Grids — alokujú len neprázdne regióny

## Sparse Voxel Representations

**Octree:** Priestor rekurzívne rozdeľovaný na 8 podkubov; alokujú sa len listy s dátami → $O(S)$ kde $S$ je počet neprázdnych voxelov.

**Sparse 3D CNN (MinkowskiEngine, spconv):** Efektívne 3D konvolúcie len na neprázdnych voxeloch → umožňuje high-resolution voxelové siete bez pamäťovej katastrofy.

## Algoritmy na voxelových dátach

| Algoritmus | Vstup | Výstup |
|---|---|---|
| **Marching Cubes** | TSDF/scalar field | Trojuholníková mesh (izoplocha) |
| **Volumetric path tracing** | TSDF | Fotorealistický render |
| **3D CNN klasifikácia** | Occupancy grid | Kategória objektu (VoxNet) |
| **Flood fill** | Binárna mriežka | Detekcia dutín, vnútra |

## Formáty

- **.vox** (MagicaVoxel)
- **.raw/.bin** (binárne polia)
- TSDF ako NumPy array / HDF5

## Praktické nasadenia

- **KinectFusion (Microsoft):** TSDF fúzia z RGB-D kamery v reálnom čase → 3D rekoštrukcia interiéru
- **NeRF:** neuronová radiačná funkcia — naučená voxelová reprezentácia hustoty a farby
- **3D objektová detekcia:** VoxNet, PointPillars (konvertuje point cloud na voxely) pre autonómne vozidlá
- **Medicínska CT segmentácia:** voxely ako prirodzená reprezentácia CT dát (DICOM)

## Súvisiace pojmy

[[07 2.5D, 3D a volumetrické reprezentácie]] · [[point cloud]] · [[polygon mesh]] · [[dense MVS and depthmap fusion]]
