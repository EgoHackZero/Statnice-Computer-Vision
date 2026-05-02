# Aplikácie v praxi

> *Prierezová stránka: kde sa stretávajú všetky metódy z 10 tém na reálnych problémoch.*

Táto stránka organizuje praktické nasadenia podľa **aplikačnej domény**, nie podľa metódy. Pre každú doménu ukazujeme, ktoré techniky sa kombinujú do komplexného systému.

---

## 1. Autonómne vozidlá

**Kompletný perception stack autonómneho vozidla:**

| Vstup | Metóda | Výstup |
|---|---|---|
| Stereo kamery | [[08 Kalibrácia stereo systému a rektifikácia\|Stereo kalibrácia]] + [[09 Problém korešpondencie, disparita a hĺbka\|SGM matching]] | 3D vzdialenosť k objektom |
| Kamera | [[06 Sémantická segmentácia a U-Net\|U-Net segmentácia]] | Maska cesty, pruhov, chodcov |
| Kamera + LiDAR | Sensor fusion | Detekcia bounding boxov vo 3D |
| LiDAR point cloud | [[07 2.5D, 3D a volumetrické reprezentácie\|Voxelová mriežka]] | Occupancy map |
| GPS + kamery + SfM | [[10 Structure-from-Motion a Multi-View Stereo\|SfM pipeline]] | HD mapa (cm presnosť) |

**Datasety:** KITTI, Waymo Open Dataset, nuScenes, CityScapes  
**Hardvér:** NVIDIA DRIVE Orin, Mobileye EyeQ, Velodyne HDL-64E LiDAR, ZED 2 stereo kamera  
**Spoločnosti:** Waymo, Tesla (Vision-only FSD), Mobileye, Cruise, NVIDIA DRIVE  

**Príklad — KITTI benchmark (stereo):**  
Baseline $B = 0{,}54$ m, $f = 721$ px → pre chodca na 20 m: $d = fB/Z = 721 \cdot 0{,}54 / 20 \approx 19{,}5$ px disparity. SGM dosahuje EPE (End-Point Error) < 1 px.

---

## 2. Medicínska diagnostika a chirurgia

**Segmentácia medicínskych obrazov:**

- [[06 Sémantická segmentácia a U-Net\|U-Net]] je **de-facto štandard** pre biomedicínsku segmentáciu:
  - Segmentácia nádorov (MRI, CT) — detekcia gliómu, karcinómu
  - Segmentácia orgánov (pečeň, obličky, srdce) z CT pre plánovanie operácie
  - Detekcia lézií retinálnych ciev z fundus fotografií (diabetická retinopatia)
  - Histologická analýza — segmentácia buniek z mikroskopických snímok

**Stereo chirurgia:**
- Da Vinci chirurgický robot: [[08 Kalibrácia stereo systému a rektifikácia\|stereo endoskop]] (kalibrovaný stereo pár) + [[09 Problém korešpondencie, disparita a hĺbka\|disparita → hĺbka]] → chirurg vidí 3D obraz vnútra tela
- Stereo laparoskopy: 3D vizualizácia pre miniinvazívne operácie

**3D rekonštrukcia z RTG:**
- CBCT (cone-beam CT) → [[07 2.5D, 3D a volumetrické reprezentácie\|voxelová reprezentácia]] → povrchová rekonštrukcia (Marching Cubes) → 3D tlač implantátov

**Knižnice:** MONAI (Medical Open Network for AI), ITK, SimpleITK, nnU-Net  
**Klinické nasadenia:** AI-Rad Companion (Siemens Healthineers), DXA bodyComposition (Hologic), retinálna diagnostika (Google DeepMind x Moorfields Eye Hospital)

---

## 3. Robotika a priemyselná automatizácia

**Robotické uchopovanie (bin picking):**
1. Stereo kamery / RGB-D senzor → [[07 2.5D, 3D a volumetrické reprezentácie\|point cloud]] dielu
2. [[10 Structure-from-Motion a Multi-View Stereo\|MVS / dense rekonštrukcia]] → 3D model polohy
3. Gripper planning v 3D priestore → presnosť ~2 mm pri $Z < 0{,}5$ m

**Inšpekcia kvality (QA):**
- SIFT / SURF ([[01 SIFT — scale space and keypoint detection\|téma 1-2]], [[04 SURF — Speeded-Up Robust Features\|téma 4]]) pre detekciu škrabancov a trhlín porovnaním s referenčným obrazom
- U-Net segmentácia povrchových defektov — presnosť 99%+ pri inline kontrole
- Structured light 3D meranie → mikrometrická presnosť rozmeru dielov

**Navigácia mobilných robotov (SLAM):**
- Varianty [[01 SIFT — scale space and keypoint detection\|SIFT]]/ORB → feature tracking → Visual SLAM (ORB-SLAM3)
- LiDAR SLAM: LOAM, LeGO-LOAM
- Fusion: Visual-Inertial Odometry (VIO) — kamera + IMU

**Frameworky:** ROS 2, OpenCV, PCL (Point Cloud Library), Open3D  
**Hardware:** Intel RealSense D435, Zivid One+, FANUC CRX cobots

---

## 4. Mapovanie a geoinformatika (UAV/drony)

**UAV fotogrametria pipeline:**
```
Letecký plán (70% overlap) → Snímanie z dronu → 
→ SfM (COLMAP/Metashape) → Hustý point cloud (MVS) →
→ DSM/DEM generovanie → Ortofotomapa (cm/px GSD)
```

- [[10 Structure-from-Motion a Multi-View Stereo\|SfM + MVS]] je jadro každého fotogrametrického softvéru
- **GSD (Ground Sampling Distance):** pri výške 100 m, $f = 25$ mm, senzor 35 mm → GSD ≈ 3-4 cm/px
- [[07 2.5D, 3D a volumetrické reprezentácie\|DEM/DSM]] (Digital Elevation/Surface Model) pre poľnohospodárstvo, banské operácie, záplavy

**Konkrétne nasadenia:**
- **Poľnohospodárstvo:** monitoring rastu plodín, mapovanie škôd po krupobití (crop monitoring, NDVI mapy)
- **Stavebníctvo:** volumetrické výpočty výkopov (m³), sledovanie progress stavby
- **Záchranárstvo:** mapovanie zón po katastrofách (zemetrasenie, záplavy)
- **Pôdne meranie:** cadastral surveying s presnosťou < 5 cm

**Softvér:** Agisoft Metashape, OpenDroneMap (open-source), Pix4D, DJI Terra  
**Certifikácie:** EASA UAS kategórie pre komerčné operácie (EU)

---

## 5. AR/VR a Hollywood VFX

**Augmented Reality tracking:**
- [[01 SIFT — scale space and keypoint detection\|SIFT]]/[[04 SURF — Speeded-Up Robust Features\|SURF]] pre *marker-less* AR tracking — detekcia a sledovanie prírodných príznakov v reálnom čase
- [[08 Kalibrácia stereo systému a rektifikácia\|Stereo kalibrácia]] pre presné priestorové umiestnenie virtuálnych objektov
- **ARCore (Google), ARKit (Apple):** využívajú Visual-Inertial Odometry; funkcie SIFT sú nahradené rýchlejšími ORB/FAST pre real-time výkon

**VFX a digitálne dvojníky:**
- [[10 Structure-from-Motion a Multi-View Stereo\|SfM + MVS]] → fotorealistický 3D model herca / prostredia zo série fotografií
- Industrial Light & Magic, Weta FX, DNEG: multi-camera rig (50-200 kamier) → hustý point cloud → mesh → UV texturovanie
- **Príklady:** *Avatar* (30+ kamier, SfM rekonštrukcia nereálnych prostredí), *The Mandalorian* (LED stage + real-time tracking)

**Virtuálna produkcia:**
- Real-time 3D tracking v LED štúdiu: kamera + SIFT → pose estimation → renderovanie pozadia v perspektíve kamery

---

## 6. Bezpečnostné systémy a biometria

**Detekcia tvárí v real-time:**
- [[03 Viola-Jones framework\|Viola-Jones]] je stále základom v embedded systémoch (kamerách, dverných zvončekoch) pre jeho rýchlosť (15+ FPS na CPU)
- Moderné systémy: MTCNN, RetinaFace (deep learning), ale VJ sa používa pre low-power IoT zariadenia

**Identifikácia a biometria:**
- Face recognition pipeline: Viola-Jones detekcia → SIFT/deep descriptor → cosine similarity matching
- Fingerprint matching: SIFT keypoints na odtlačkoch prstov → GHT pre rotačne invariantné porovnanie

**Dohľadové systémy:**
- CCTV analýza: [[06 Sémantická segmentácia a U-Net\|sémantická segmentácia]] chodcov, vozidiel, predmetov
- [[07 2.5D, 3D a volumetrické reprezentácie\|LiDAR point cloud]] perimeter monitoring (menej ovplyvnený svetlom ako kamery)

---

## 7. Kultúrne dedičstvo a archeológia

**3D digitalizácia pamiatok:**
- [[10 Structure-from-Motion a Multi-View Stereo\|SfM + MVS]] z bežných fotografií → presná 3D replika sochy, fasády, nálezisku
- Príklady: *Scan the World* (3M+ 3D modelov pamiatok), CyArk (digitalizácia ohroznených pamätníkov — Notre Dame pred požiarom 2019)
- Presnosť: 1-5 mm pri správnom overlapping a GCP (ground control points)

**Archeológia:**
- UAV fotogrametria (letecká perspektíva) → odhalenie skrytých štruktúr v krajine
- Terrestrial laser scanning (TLS) → [[07 2.5D, 3D a volumetrické reprezentácie\|hustý point cloud]] so sub-mm presnosťou

**Interaktívne múzeá:**
- Google Arts & Culture — Street View-style prehliadky
- Virtual reconstructions — digitálna rekonštrukcia zrúcanín (Rím, Pompeje)

---

## 8. E-commerce a retail

**3D produktové skeny:**
- [[10 Structure-from-Motion a Multi-View Stereo\|SfM]] zo série fotografií produktu na turntable → interaktívny 3D model na webe
- IKEA Place app, Amazon 3D: zákazník vidí produkt v AR v svojej izbe
- Try-before-you-buy: virtuálne skúšanie odevov, okuliarov, nábytku

**Inventarizácia:**
- [[07 2.5D, 3D a volumetrické reprezentácie\|Point cloud]] zo skladových kamier / mobilného skenu → automatická kontrola stavu políc
- Volumetrické meranie paliet pre logistiku

---

## 9. Stomatológia a chirurgia

**Dentálna fotogrametria:**
- Intraorálny 3D skener → [[07 2.5D, 3D a volumetrické reprezentácie\|point cloud zubov]] → CAD/CAM výroba koruniek
- CEREC, iTero — komerčné systémy so sub-mm presnosťou

**Chirurgické plánovanie:**
- CT → 3D mesh kostí → virtuálna operácia pre ortopédie (náhrada bedrového kĺbu, maxilofaciálna chirurgia)
- Surgical navigation: real-time registrácia 3D modelu na pacienta pomocou kamier a markerov

---

## 10. Vesmír a vedecký výskum

**Planetárna rekonštrukcia:**
- SfM z fotografií sondy / rovera → 3D mapa povrchu
- Mars: Curiosity, Perseverance používajú stereo kamery + SfM pre navigáciu
- Stereo Mastcam-Z na Perseverance: baseline 24 cm, f ≈ 320 px → cm presnosť na 5 m

**Astronomické snímkovanie:**
- Detekcia hviezd a galaxií: blob detection ([[01 SIFT — scale space and keypoint detection\|DoG/LoG]]) → katalogizácia
- Meranie polohy meteoroidov: two-camera triangulation = stereo s veľmi veľkým baseline

---

## Súvisiace témy

[[01 SIFT — scale space and keypoint detection]] · [[03 Viola-Jones framework]] · [[06 Sémantická segmentácia a U-Net]] · [[07 2.5D, 3D a volumetrické reprezentácie]] · [[08 Kalibrácia stereo systému a rektifikácia]] · [[09 Problém korešpondencie, disparita a hĺbka]] · [[10 Structure-from-Motion a Multi-View Stereo]]
