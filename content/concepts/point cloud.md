# Point Cloud

*Point cloud (oblak bodov)* je množina 3D bodov $\{(x_i, y_i, z_i)\}_{i=1}^N$ reprezentujúca povrch alebo štruktúru trojrozmernej scény. Každý bod môže mať aj atribúty: farba (RGB), normála, intenzita odrazu (LiDAR), čas, istota.

## Vlastnosti

- **Štruktúra:** neusporiadaná množina (žiadna implicitná topológia, žiadne susedstvo)
- **Hustota:** typicky $10^4$ – $10^8$ bodov podľa senzora
- **Pamäť:** $\approx 12–28$ bajtov/bod (xyz + atribúty) → efektívnejší ako voxelová mriežka rovnakého rozlíšenia

## Zdroje point cloudu

| Senzor | Princíp | Typická hustota |
|---|---|---|
| **LiDAR** | Time-of-flight lasera | $10^5$–$10^7$ bodov/scan |
| **Stereo kamery + MVS** | [[dense MVS and depthmap fusion\|Depthmap fusion]] | $10^5$–$10^6$ |
| **Structured light** | Premietaný vzor + kamera | $10^5$–$10^6$, presnosť ~0.1 mm |
| **ToF kamera** | Time-of-flight (IR pulzy) | $10^4$–$10^5$, nižšia presnosť |
| **RGB-D (Kinect, RealSense)** | IR structured light + stereo | $10^5$, interiéry |

## Algoritmy na point cloudoch

**Vyhľadávanie susedov:**
- k-NN (k najbližších susedov) — pre normálové odhady, segmentáciu
- Radius search — pre lokálne deskriptory
- **KD-tree** (efektívne pre 3D, O(log N)) alebo **Octree** (adaptívne delenie priestoru)

**Registrácia (zarovnanie dvoch cloudov):**
- **ICP (Iterative Closest Point):** iteratívne minimalizuje vzdielosť medzi zodpovedajúcimi pármi bodov
- **NDT (Normal Distributions Transform):** modeluje cloud ako súčet gaussiánov

**Segmentácia:**
- **RANSAC plane fitting:** nájde najväčšiu rovinu (napr. podlaha) a odfiltruje ju
- **Euclidean cluster extraction:** klastrovanie podľa vzdialenosti susedov
- PointNet, PointNet++ (deep learning pre point cloudy)

**Redukcia:**
- **Voxel grid downsampling:** každý voxel → jeden reprezentatívny bod (centroid)
- **Statistical outlier removal:** odstraňuje izolované noise body

## Formáty

| Formát | Popis |
|---|---|
| **.ply** | Polygon format — flexibilné, podporuje XYZ+RGB+normály |
| **.pcd** | Point Cloud Data — štandard PCL knižnice |
| **.las / .laz** | LiDAR industry štandard (komprimovaný .laz) |
| **.xyz** | Jednoduchý text, XYZ per riadok |
| **.e57** | ISO štandard pre 3D imagery |

## Konverzia na iné reprezentácie

- Point cloud → **mesh**: Poisson Surface Reconstruction, BPA, Alpha Shape
- Point cloud → **voxely**: voxel grid downsampling, TSDF fúzia
- Point cloud → **depth map**: projekcia do kamery s known extrinsics

## Praktické využitie

- LiDAR mapping v autonómnych vozidlách (Waymo, Tesla, Cruise)
- Architektonická a inžinierska dokumentácia (BIM z TLS skenov)
- Lesníctvo: objem biomasy z leteckého LiDAR skenu
- Priemyselná metrológia: kontrola odchýlok CAD modelu vs. sken dielu

## Knižnice

- **PCL (Point Cloud Library):** C++, komplexná
- **Open3D:** Python/C++, moderná, vizualizácia
- **PyTorch3D:** deep learning na point cloudoch
- **PDAL:** pipeline pre LiDAR data

## Súvisiace pojmy

[[07 2.5D, 3D a volumetrické reprezentácie]] · [[voxel grid]] · [[polygon mesh]] · [[dense MVS and depthmap fusion]] · [[bundle adjustment]]
