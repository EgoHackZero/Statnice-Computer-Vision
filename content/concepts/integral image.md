# Integral image

*Integral image* (slovensky: integrálny obraz), známy aj ako *summed-area table*, je dátová štruktúra odvodená z pôvodného obrazu, ktorá umožňuje vypočítať sumu pixelov v ľubovoľnom osovo zarovnanom obdĺžnikovom regióne v konštantnom čase $O(1)$ — bez ohľadu na veľkosť regiónu.

## Detaily

**Definícia.** Pre vstupný obraz $I(x, y)$ je integrálny obraz definovaný ako:

$$II(x,\, y) = \sum_{x' \leq x,\; y' \leq y} I(x', y')$$

Každá hodnota $II(x, y)$ teda obsahuje sumu všetkých pixelov nachádzajúcich sa naľavo od stĺpca $x$ a nad riadkom $y$ vrátane samotného pixela $(x, y)$. Integrálny obraz sa vypočíta jedným prechodom cez celý obraz pomocou jednoduchého rekurentného vzťahu:

$$II(x, y) = I(x, y) + II(x-1, y) + II(x, y-1) - II(x-1, y-1)$$

Výpočet teda trvá $O(N)$, kde $N$ je celkový počet pixelov, a je potrebný iba raz pre každý vstupný obraz.

**Výpočet sumy obdĺžnikového regiónu.** Po zostavení integrálneho obrazu možno sumu pixelov v ľubovoľnom obdĺžniku s rohmi $A$ (vľavo hore), $B$ (vpravo hore), $C$ (vľavo dole) a $D$ (vpravo dole) vypočítať pomocou štyroch hodnôt $II$:

$$\text{Sum}(ABCD) = II(D) + II(A) - II(B) - II(C)$$

kde $A$ je bod vľavo hore *pred* obdĺžnikom (index $[r_1-1, c_1-1]$), $B$ vpravo hore, $C$ vľavo dole a $D$ vpravo dole. Táto operácia vyžaduje iba štyri násobenia pamäte a tri aritmetické operácie — teda skutočne $O(1)$.

**Prečo je to dôležité.** Bez integrálneho obrazu by výpočet sumy pixelov v obdĺžniku $w \times h$ vyžadoval $O(w \cdot h)$ operácií. Pri skúmaní okna $24 \times 24$ pixelov v rámci detekcie tváre a vyhodnocovaní stoviek Haarových čŕt by bol priamy prístup rádovo pomalší. Integrálny obraz oddeľuje cenu výpočtu od veľkosti regiónu, čo je kľúčové pre efektívnu prácu s veľkými filtrovacími oknami v reálnom čase.

**Využitie v SURF.** Okrem Viola-Jones frameworku používa integrálny obraz aj metóda [[04 SURF — Speeded-Up Robust Features]], kde umožňuje aplikovať krabicové (*box*) filtre ľubovoľnej veľkosti rovnako rýchlo — to je základ pre detekciu bodov záujmu pomocou determinantu Hessiánovej matice na viacerých škálach bez potreby explicitnej konštrukcie obrazovej pyramídy.

## Kde sa používa

- [[03 Viola-Jones framework]] — výpočet Haarových čŕt v okne $24 \times 24$
- [[04 SURF — Speeded-Up Robust Features]] — krabicové filtre pre aproximáciu druhých derivácií
- Algoritmy pre výpočet sumy plôch, efektívne konvolúcie, integrálne histogramy

## Súvisiace pojmy

- [[Haar-like features]]
- [[Hessian matrix and blob detection]]
- [[03 Viola-Jones framework]]
- [[04 SURF — Speeded-Up Robust Features]]
