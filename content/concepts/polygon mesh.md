# Polygon Mesh

*Polygon mesh* je explicitná reprezentácia 3D povrchu ako sieť vrcholov (vertices), hrán (edges) a polygónov (faces — typicky trojuholníky alebo štvoruholníky).

## Štruktúra dát

**Základné zložky:**
- **Vertices $V$:** zoznam 3D súradníc $\{(x_i, y_i, z_i)\}$
- **Faces $F$:** trojice (alebo štvorice) indexov vrcholov popisujúcich polygón
- **Normals:** jednotkové vektory kolmé na povrch (per-vertex alebo per-face)
- **UV súradnice:** mapovanie na textúrovú mapu (2D)

**Príklad (trojuholníkový mesh):**
```
Vertices: [(0,0,0), (1,0,0), (0,1,0), (0,0,1)]
Faces:    [(0,1,2), (0,1,3), (0,2,3), (1,2,3)]  ← tetraéder
```

## Typy meshov

| Typ | Popis | Použitie |
|---|---|---|
| **Triangle mesh** | Len trojuholníky | Univerzálny, GPU rendering |
| **Quad mesh** | Štvoruholníky | Animácia, sculpting |
| **Watertight (closed) mesh** | Uzavretý povrch bez dier | Fyzikálna simulácia, 3D tlač |
| **Open mesh** | S hranicami (napr. povrch terénu) | Terénne modely, neúplné skeny |

## Algoritmy konverzie z iných reprezentácií

### Z point cloudu

**Poisson Surface Reconstruction (Kazhdan & Hoppe):**
- Vstup: body + normály
- Rieši Poissonovu rovnicu → indikátorová funkcia interiér/exteriér → izoplocha
- Výhoda: hladký, watertight output; Nevýhoda: môže extrapolovať do prázdnych oblastí

**Ball Pivoting Algorithm (BPA):**
- Guľa s polomerom $r$ sa „prevalí" po bodoch → zachytáva trojuholníky
- Presnejší na hranách, ale citlivý na parametre

### Z voxelov

**Marching Cubes (Lorensen & Cline 1987):**
- Každý voxelový kub má 8 rohov → binárne on/off → 256 konfigurácií → lookup table trojuholníkov
- Výhoda: rýchle, priamočiare; Nevýhoda: „schodovitý" výsledok → potrebuje Laplacian smoothing

## Operácie na meshoch

- **Decimation:** redukcia počtu trojuholníkov pri zachovaní tvaru (QEM, Quadric Error Metrics)
- **Subdivision:** zjemnenie meshu (Catmull-Clark, Loop)
- **Remeshing:** pravidelná redistribúcia vrcholov (izotropný remesh)
- **Boolean operations:** union, intersection, difference dvoch meshov
- **Laplacian smoothing:** pohyb vrchola k priemeru susedov → vyhladzuje povrch

## Textúrovanie

1. **UV unwrapping:** 3D mesh rozložíme do 2D (UV space)
2. **UV atlas:** všetky trojuholníky zabalíme do textúrovej mapy
3. **Projekcia fotografií:** z každého trojuholníka vyberieme najlepší obraz → baking textúry
4. **Blending:** eliminovanie švov medzi rôznymi pohľadmi

## Pamäť a výkon

Pre mesh s $V$ vrcholmi a $F$ trojuholníkmi:
- Vertices: $V \times 12$ B (3× float32)
- Faces: $F \times 12$ B (3× int32)
- Normály: $V \times 12$ B
- Typický pomer: $F \approx 2V$ (Eulerova formula)

## Formáty

| Formát | Typ | Poznámka |
|---|---|---|
| **.obj** | Text | Najrozšírenejší, podporuje MTL textúry |
| **.ply** | Bin/Text | Point cloud aj mesh, flexibilný |
| **.fbx** | Bin | Priemyselný štandard (Unity, Unreal) |
| **.glb/.gltf** | Bin | Web 3D (Three.js, Babylon.js) |
| **.stl** | Bin/Text | 3D tlač (len geometria) |
| **.dae (Collada)** | XML | Výmena medzi DCC nástrojmi |

## Praktické nasadenia

- Herný priemysel a film: photogrammetry → mesh → UV texturovanie (Metashape, ZBrush, Blender)
- 3D tlač: STL/OBJ mesh → slicing (Cura) → G-code
- Simulácia fyziky: watertight mesh → CFD alebo FEM analýza
- Virtuálne múzeá: mesh + fotorealistická textúra → WebGL prehliadka

## Súvisiace pojmy

[[07 2.5D, 3D a volumetrické reprezentácie]] · [[point cloud]] · [[voxel grid]] · [[dense MVS and depthmap fusion]]
