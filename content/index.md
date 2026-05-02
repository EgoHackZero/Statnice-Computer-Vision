# Počítačové videnie — Statnice

> Komplexný zdroj pre prípravu na štátnu skúšku z predmetu **Pokročilé metódy počítačového videnia**.

Každá stránka zodpovedá jednej otázke zo štátnicovej komisie. Odpovede sú komplexné — pokrývajú každý čiastkový bod otázky, obsahujú matematiku, porovnania alternatív a konkrétne praktické príklady z priemyslu.

---

## Otázky — 10 tém

| # | Téma | Kľúčové pojmy |
|---|---|---|
| [[01 SIFT — scale space and keypoint detection\|01]] | SIFT — scale space a detekcia kľúčových bodov | DoG, Gaussian pyramid, Hessian edge test, orientácia |
| [[02 SIFT — descriptor and Generalized Hough Transform\|02]] | SIFT — deskriptor a Generalized Hough Transform | 128D descriptor, Lowe ratio test, GHT voting |
| [[03 Viola-Jones framework\|03]] | Viola-Jones — rýchla detekcia tvárí | Haar features, integral image, AdaBoost, cascade |
| [[04 SURF — Speeded-Up Robust Features\|04]] | SURF — rýchlejšia alternatíva SIFT | Fast Hessian, box filters, 64D descriptor, U-SURF |
| [[05 Konvolučné neurónové siete — tréning a regularizácia\|05]] | CNN — tréning a regularizácia | BatchNorm, L1/L2, data splits, normalizácia |
| [[06 Sémantická segmentácia a U-Net\|06]] | Sémantická segmentácia a U-Net | FCN, encoder-decoder, skip connections, Dice loss |
| [[07 2.5D, 3D a volumetrické reprezentácie\|07]] | 3D reprezentácie a snímanie priestoru | Point cloud, voxel, mesh, LiDAR, ToF, fotogrametria |
| [[08 Kalibrácia stereo systému a rektifikácia\|08]] | Stereo kalibrácia a rektifikácia | Zhang metóda, K matica, F/E matica, epipólová geometria |
| [[09 Problém korešpondencie, disparita a hĺbka\|09]] | Korešpondencia, disparita a hĺbka | Block matching, SGM, Z = fB/d, sub-pixel |
| [[10 Structure-from-Motion a Multi-View Stereo\|10]] | SfM a Multi-View Stereo | Bundle adjustment, PatchMatch, Poisson rekonštrukcia |

---

## Praktické aplikácie

[[Aplikácie v praxi]] — prierezová stránka, ktorá spája všetkých 10 tém s konkrétnymi priemyselnými aplikáciami: autonómne vozidlá, medicínska diagnostika, robotika, AR/VR, kultúrne dedičstvo, mapovanie z UAV.

---

## Konceptuálny slovník

Klikni na pojem v grafe vpravo alebo prechádzaj priamo:

**Detekcia príznakov:**
[[scale space and Gaussian pyramid]] · [[Difference of Gaussians]] · [[Hessian matrix and blob detection]] · [[Lowe ratio test]] · [[Generalized Hough Transform]]

**Klasická detekcia objektov:**
[[Haar-like features]] · [[integral image]] · [[AdaBoost]] · [[attentional cascade]]

**CNN a segmentácia:**
[[batch normalization]] · [[L1 and L2 regularization]] · [[train-validation-test split]] · [[data normalization and class imbalance]] · [[transposed convolution]] · [[unpooling]] · [[skip connections]]

**3D videnie:**
[[point cloud]] · [[voxel grid]] · [[polygon mesh]] · [[camera intrinsics and Zhang calibration]] · [[epipolar geometry]] · [[fundamental and essential matrix]] · [[triangulation]] · [[disparity and depth]] · [[block matching and SGM]] · [[bundle adjustment]] · [[dense MVS and depthmap fusion]]

---

## Ako používať tento zdroj

1. **Čítaj tému k otázke** — každá stránka obsahuje odpoveď v rozsahu, ktorý zodpovedá ústnej skúške
2. **Naviguj graf** — vpravo hore je interaktívny graf znalostí; kliknutím na uzol sa dostaneš na súvisiacu stránku
3. **Testuj sa sám** — na konci každej stránky je sekcia *"Spýtaj sa sám seba"* s 5 otázkami
4. **Hľadaj** — vyhľadávanie (Ctrl+K) funguje aj s diakritikou a angličtinou
