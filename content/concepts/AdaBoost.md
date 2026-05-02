# AdaBoost

*AdaBoost* (z anglického *Adaptive Boosting*) je metóda strojového učenia navrhnutá Freundom a Schapireom v roku 1997. Ide o algoritmus *boostingu* — techniky, ktorá kombinuje viacero slabých klasifikátorov (*weak classifiers*) do jedného silného klasifikátora (*strong classifier*). Slabý klasifikátor je ľubovoľný model, ktorého presnosť je len mierne lepšia ako náhodný odhad (napr. jednoduchý prahový test nad jednou črtou). AdaBoost iteratívne trénuje tieto slabé klasifikátory a prikladá väčšiu váhu príkladom, ktoré boli doteraz nesprávne klasifikované.

## Detaily

**Algoritmus AdaBoost — krok za krokom.** Predpokladajme trénovaciu množinu $\{(x_i, y_i)\}_{i=1}^{N}$, kde $y_i \in \{-1, +1\}$, a iterácie $m = 1, \ldots, M$:

1. **Inicializácia váh:** $w_i^{(1)} = \frac{1}{N}$ pre všetky vzorky.
2. **Trénovanie slabého klasifikátora:** V každej iterácii $m$ nájdi slabý klasifikátor $h_m(x)$, ktorý minimalizuje vážený chybový podiel:
$$\varepsilon_m = \sum_{i=1}^{N} w_i^{(m)} \cdot \mathbf{1}\bigl[h_m(x_i) \neq y_i\bigr]$$
3. **Váha klasifikátora:** Vypočítaj dôveryhodnosť (*confidence weight*) tohto klasifikátora:
$$\alpha_m = \frac{1}{2} \ln\!\left(\frac{1 - \varepsilon_m}{\varepsilon_m}\right)$$
Ak je $\varepsilon_m < 0.5$ (lepší ako náhoda), potom $\alpha_m > 0$. Čím nižšia chyba, tým väčšia váha.
4. **Aktualizácia váh vzoriek:** Váhy sa aktualizujú tak, aby misklasifikované príklady dostali väčšiu váhu:
$$w_i^{(m+1)} = w_i^{(m)} \cdot \exp\bigl(-\alpha_m \cdot y_i \cdot h_m(x_i)\bigr)$$
Ak $h_m$ klasifikoval $x_i$ správne, súčin $y_i \cdot h_m(x_i) = +1$ a váha klesá; ak nesprávne, $y_i \cdot h_m(x_i) = -1$ a váha rastie.
5. **Normalizácia váh:** $w_i^{(m+1)} \leftarrow w_i^{(m+1)} / \sum_j w_j^{(m+1)}$, aby váhy tvorili pravdepodobnostné rozdelenie.
6. **Výsledný silný klasifikátor:**
$$H(x) = \text{sign}\!\left(\sum_{m=1}^{M} \alpha_m \, h_m(x)\right)$$

**Interpretácia $\alpha_m$.** Pri $\varepsilon_m \to 0$ (takmer dokonalý klasifikátor) $\alpha_m \to \infty$; pri $\varepsilon_m = 0.5$ (náhoda) $\alpha_m = 0$; pri $\varepsilon_m > 0.5$ $\alpha_m < 0$ (klasifikátor sa invertuje). Trénovací chybový podiel silného klasifikátora klesá exponenciálne s počtom kôl, čo AdaBoost robí výnimočne efektívnym.

**AdaBoost v kontexte Viola-Jones.** V [[03 Viola-Jones framework]] je každý slabý klasifikátor $h_m$ jednoduchý prahový test nad jednou [[Haar-like features|Haarovou črtou]]:
$$h_m(x) = \begin{cases} +1 & \text{ak } f_m(x) > \theta_m \\ -1 & \text{inak} \end{cases}$$
kde $f_m$ je hodnota konkrétnej Haarovej čŕty a $\theta_m$ je naučený prah. AdaBoost tak plní dvojitú úlohu: **výber čŕt** (zo $160\,000$ kandidátov vyberie $\approx 200$ najdiskriminatívnejších) a **tréning klasifikátora** (zostaví ich vážený konsenzus).

**Výhody a obmedzenia.** AdaBoost je relatívne jednoduchý na implementáciu, nepotrebuje ladenie hyperparametrov a je teoreticky podložený (PAC-learning). Je však citlivý na šum v trénovacích dátach a *outliers*, pretože misklasifikovaným vzorkám neustále zvyšuje váhu, čo môže viesť k preučeniu pri hlučných dátach.

## Kde sa používa

- [[03 Viola-Jones framework]] — výber Haarových čŕt a tréning kaskády
- [[attentional cascade]] — každá kaskádová vrstva je samostatný AdaBoost klasifikátor
- Obecne v strojovom učení ako súčasť *ensemble* metód (GBM, XGBoost sú príbuzné)

## Súvisiace pojmy

- [[Haar-like features]]
- [[attentional cascade]]
- [[03 Viola-Jones framework]]
