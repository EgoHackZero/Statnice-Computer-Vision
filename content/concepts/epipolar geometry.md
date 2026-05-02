# Epipolar Geometry

*Epipólová geometria* popisuje geometrický vzťah medzi dvoma kamerovými pohľadmi na tú istú scénu. Je fundamentálna pre stereo videnie, SfM a multi-view rekonštrukciu.

## Základné pojmy

### Epipól

*Epipól* $\mathbf{e}$ je priesečník spojnice optických stredov $C_1C_2$ s obrazovou rovinou.

- Epipól ľavej kamery $\mathbf{e}_L$ = projekcia optického stredu pravej kamery do ľavého obrazu
- Epipól pravej kamery $\mathbf{e}_R$ = projekcia optického stredu ľavej kamery do pravého obrazu

Pri kamery pohybujúcej sa priamo dopredu (baseline = smer pohybu): epipól je v smere pohybu — tzv. *focus of expansion*.

### Epipólová rovina

Pre 3D bod $\mathbf{M}$ tvoria tri body $C_1$, $C_2$, $\mathbf{M}$ *epipólovú rovinu*. Táto rovina pretína oba obrazy v *epipólových čiarach*.

### Epipólová čiara

*Epipólová čiara* $\mathbf{l}'$ je priesečník epipólovej roviny s obrazovou rovinou. Kľúčová vlastnosť:

> Ak poznáme bod $\mathbf{x}$ v ľavom obraze, jeho korešpondencia $\mathbf{x}'$ v pravom obraze **musí ležať na epipólovej čiare** $\mathbf{l}' = F\mathbf{x}$.

Tým sa 2D hľadanie redukuje na 1D problém — obrovský výpočtový zisk.

## Epipólová podmienka

Pre zodpovedajúce body $\mathbf{x} \leftrightarrow \mathbf{x}'$ v homogénnych súradniciach:

$$\mathbf{x}'^T F \mathbf{x} = 0$$

kde $F$ je fundamentálna matica. Geometricky: bod $\mathbf{x}'$ leží na epipólovej čiare $F\mathbf{x}$.

## Rektifikovaná epipólová geometria

Po [[08 Kalibrácia stereo systému a rektifikácia\|rektifikácii]]:
- Oba obrazy sú transformované tak, aby epipólové čiary boli **horizontálne a rovnobežné**
- Epipóly sú „odsunuté na nekonečno" v horizontálnom smere
- Korešpondujúce body majú rovnakú y-súradnicu → 1D hľadanie pozdĺž riadku

Rektifikácia prakticky eliminuje potrebu explicitného výpočtu epipólových čiar pri stereo matchingu.

## Využitie epipólovej geometrie

| Aplikácia | Ako sa používa |
|---|---|
| **Stereo matching** | Hľadaj korešpondenciu len pozdĺž epipólovej čiary |
| **Outlier filtrácia** | Zhody nesplňujúce $\mathbf{x}'^T F \mathbf{x} \approx 0$ sú outliery (RANSAC) |
| **Camera pose recovery** | Z $F$ → $E$ → $(R, \mathbf{t})$ relatívna poloha kamery |
| **Multi-view stereo** | Výber kompatibilných snímok pre MVS |

## Vzťah k SfM

V SfM pipeline sa odhaduje $F$ (alebo $E$) medzi každým párom obrazov. Ak odmietnuté páry (málo inlierov alebo degenerovaná konfigurácia), ten pár sa nepoužije. Kvalita odhadnutého $F$ priamo ovplyvňuje kvalitu triangulovaných 3D bodov.

## Degenerácie

Epipólová geometria zlyhá (je nedefinovaná alebo nestabilná) ak:
- Všetky 3D body sú v jednej rovine (planárna scéna) → $F$ nie je jednoznačne definovaná, ale existuje homografia
- Čistá rotácia kamery (žiadna translácia) → epipóly sú v nekonečne, $E$ nevymedzí $(R, t)$ jednoznačne

## Súvisiace pojmy

[[08 Kalibrácia stereo systému a rektifikácia]] · [[fundamental and essential matrix]] · [[camera intrinsics and Zhang calibration]] · [[triangulation]]
