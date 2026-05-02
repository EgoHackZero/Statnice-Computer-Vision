# Lowe Ratio Test

*Lowov ratio test* (pomer test) je jednoduchý, ale účinný filter na odmietnutie nejednoznačných zhôd kľúčových bodov pri SIFT (a iných deskriptorových) párovaní.

## Detaily

**Problém nejednoznačných zhôd:**
Pri hľadaní korešpondencií sa pre každý kľúčový bod v obraze A nájde najbližší sused (*nearest neighbor, NN*) v databáze deskriptorov obrazu B. Niekedy však existujú dva kandidáti, ktorí sú si navzájom veľmi podobní — žiaden z nich nie je s istotou správna zhoda (napr. pri opakujúcich sa vzoroch).

**Riešenie — ratio test (Lowe 2004):**
Namiesto akceptovania každého NN porovnáme vzdialenosť najbližšieho ($d_1$) a *druhého* najbližšieho ($d_2$) suseda:

$$\frac{d_1}{d_2} < \tau \quad \Rightarrow \quad \text{zhoda je akceptovaná}$$

David Lowe experimentálne stanovil prah $\tau = 0{,}8$. Ak je pomer menší ako 0,8, najbližší sused je výrazne lepšia zhoda ako druhý najbližší → zhoda je jednoznačná → akceptujeme. Ak je pomer blízky 1,0, oba kandidáti sú si podobne vzdialení → zhoda je nejednoznačná → odmietame.

**Interpretácia:**
- Malý pomer: kľúčový bod má v databáze *jedného* jasného protihráča → vysoká dôvera
- Veľký pomer (~1,0): kľúčový bod sa podobá na viacero databázových bodov → pravdepodobne opakovanie vzoru alebo pozadí šum

**Účinnosť:**
Lowe uvádza, že ratio test s τ = 0,8 eliminuje ~90 % falošných zhôd (false positives), pričom zahodí len ~5 % správnych zhôd (false negatives). Je to veľmi dobrý kompromis pre väčšinu aplikácií.

**Implementácia:**
```python
# OpenCV: BFMatcher s ratio testom (Lowov spôsob)
bf = cv2.BFMatcher()
matches = bf.knnMatch(desc1, desc2, k=2)  # 2 najlepší susedia
good = []
for m, n in matches:
    if m.distance / n.distance < 0.8:
        good.append(m)
```

**Alternatívy:**
- Prahová vzdialenosť (absolútna): menej robustná, závisí od škálovania deskriptorov
- Vzájomné overenie (*cross-check*): A→B zhoda musí byť zhodná s B→A; rýchle, ale menej citlivé
- RANSAC po párovaní: geometrická verifikácia odfiltruje zvyšné zlé zhody

## Kde sa používa

- [[02 SIFT — descriptor and Generalized Hough Transform]] — základná súčasť SIFT matching pipeline
- [[10 Structure-from-Motion a Multi-View Stereo]] — SfM feature matching krok
- Akýkoľvek systém využívajúci lokálne deskriptory (SIFT, SURF, ORB, AKAZE)

## Súvisiace pojmy

[[02 SIFT — descriptor and Generalized Hough Transform]], [[Generalized Hough Transform]], [[10 Structure-from-Motion a Multi-View Stereo]]
