# Attentional cascade

*Attentional cascade* (slovensky: kaskáda s pozornosťou, alebo kaskáda klasifikátorov) je architektonický princíp navrhnutý Violou a Jonesom na dramatické zrýchlenie detekcie objektov v obraze. Základná myšlienka vychádza z pozorovania, že v typickom obraze je drvivá väčšina prehľadávaných podokien (*subwindows*) jednoznačne negatívna (neobsahuje hľadaný objekt). Kaskáda tieto podokná eliminuje čo najskôr pomocou jednoduchých a rýchlych testov, pričom zložitejšiu analýzu aplikuje iba na sľubné kandidáty.

## Detaily

**Štruktúra kaskády.** Kaskáda pozostáva z $K$ stupňov (*stages*), pričom každý stupeň $k$ je boosted klasifikátor s $n_k$ [[Haar-like features|Haarovými črtami]]. Stupne sú zoradené od najjednoduchšieho po najzložitejší:

- **Stupeň 1:** $n_1 \approx 2$–$5$ čŕt. Veľmi vysoká citlivosť (*recall* blízky 1.0), veľa falošne pozitívnych. Odmieta $\approx 50\%$ negatívnych okien.
- **Stupeň 2:** $n_2 \approx 10$ čŕt. Odmieta ďalšiu podstatnú časť zostávajúcich negatívnych okien.
- **...**
- **Stupeň K:** $n_K \approx 200$ čŕt. Finálne rozhodnutie — len okná, ktoré prešli všetkými predchádzajúcimi stupňami.

Podokno je vyhlásené za pozitívne (*tvár*) iba ak prejde **všetkými** stupňami. Ak ho akýkoľvek stupeň odmietne, okno je okamžite zahodené bez ďalšieho spracovania.

**Matematické zdôvodnenie rýchlosti.** Nech $p_k$ je pravdepodobnosť, že negatívne okno prejde stupňom $k$ (t.j. falošne pozitívna miera stupňa). Celková falošne pozitívna miera kaskády je:

$$\text{FPR}_{\text{kaskáda}} = \prod_{k=1}^{K} p_k$$

Ak každý stupeň dosiahne $p_k = 0.5$, po $K = 20$ stupňoch je $\text{FPR} \approx 10^{-6}$. Priemerný počet čŕt vyhodnotených na negatívne okno je oveľa menší ako $\sum_k n_k$, pretože väčšina okien je odmietnutá v prvých stupňoch. Viola a Jones uviedli, že na skenovanie obrazu $384 \times 288$ px stačilo ich systému $\approx 0.067$ sekundy na hardvéri z roku 2001.

**Tréning kaskády.** Každý stupeň sa trénuje ako samostatný [[AdaBoost]] klasifikátor. Cieľom je dosiahnuť pre každý stupeň vysokú detekčnú mieru (napr. $\geq 99.9\%$) pri relatívne vysokej falošne pozitívnej miere (napr. $\approx 50\%$). Po natrénovaní stupňa sa spustí na trénovacej množine, odmietnuté negatívne vzorky sa odhodia a zostávajúce ťažké negatívne vzorky (*hard negatives*) tvoria trénovaciu množinu pre ďalší stupeň — ide o princíp *bootstrappingu*.

**Analógia s ľudskou pozornosťou.** Kaskáda napodobňuje ľudský vizuálny systém: prvý pohľad (*preattentive vision*) rýchlo identifikuje oblasti hodné pozornosti, a až potom sa fokusovaná analýza (*attentive vision*) aplikuje na sľubné regióny. Preto názov *attentional cascade*.

**Dôsledky pre návrh systému.** Kaskádová architektúra je jedným z dôvodov, prečo Viola-Jones dokázal bežať v reálnom čase na CPU bez GPU akcelerácie, čo bolo v roku 2001 revolučné. Rovnaký princíp sa objavuje v moderných detektoroch (napr. anchor-free detektory používajú hierarchické filtrovanie kandidátov).

## Kde sa používa

- [[03 Viola-Jones framework]] — primárne použitie, detekcia tváre v reálnom čase
- OpenCV `cv2.CascadeClassifier` — implementácia kaskáde pre rôzne objekty (tvár, oči, úsmev, chodci)
- Moderné detektory objektov (inšpirované princípom hierarchického odmietania negatívnych kandidátov)

## Súvisiace pojmy

- [[AdaBoost]]
- [[Haar-like features]]
- [[integral image]]
- [[03 Viola-Jones framework]]
