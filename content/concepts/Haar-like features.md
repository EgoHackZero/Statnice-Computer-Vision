# Haar-like features

*Haar-like features* (slovensky: Haarove črty) sú jednoduchá trieda vizuálnych príznakov inšpirovaná Haarovými vlnkami (*Haar wavelets*), pomenovanými podľa maďarského matematika Alfréda Haara. Každá Haarova črta je definovaná ako rozdiel súčtov intenzít pixelov v dvoch alebo viacerých susedných obdĺžnikových regiónoch. Črty sú primitívne, ale niektoré z nich silno reagujú na špecifické vizuálne vzory (hrany, čiary, symetrie) charakteristické pre tváre alebo iné objekty.

## Detaily

**Typy Haarových čŕt.** Viola a Jones definovali tri základné kategórie:

1. **Dvojobdĺžnikové črty** (*two-rectangle features*): rozdiel súčtov dvoch susedných obdĺžnikových oblastí rovnakej veľkosti. Zachytávajú horizontálne alebo vertikálne hrany — napr. svetlé čelo oproti tmavším očniciam.

2. **Trojobdĺžnikové črty** (*three-rectangle features*): centrálny obdĺžnik sa odpočíta od dvoch bočných obdĺžnikov (alebo naopak). Zachytávajú čiarové štruktúry — napr. svetlý mostík nosa medzi tmavšími očami.

3. **Štvrobdĺžnikové črty** (*four-rectangle features*): diagonálne dvojice susedných obdĺžnikov sa odčítajú navzájom. Zachytávajú diagonálne hrany a centrálno-okolité (*center-surround*) vzory.

**Matematická definícia.** Pre dvojobdĺžnikovú črtu:

$$f = \sum_{(x,y) \in R_{\text{biela}}} I(x,y) \;-\; \sum_{(x,y) \in R_{\text{čierna}}} I(x,y)$$

kde $R_{\text{biela}}$ a $R_{\text{čierna}}$ sú kladná a záporná oblasť. Pomocou [[integral image]] sa každá takáto suma vypočíta v $O(1)$.

**Počet možných čŕt.** Pre detekčné okno $24 \times 24$ pixelov existuje viac ako $160\,000$ možných Haarových čŕt rôznych veľkostí a pozícií. Samotné črty sú veľmi slabé — väčšina z nich je irelevantná pre konkrétny detektor. Preto [[AdaBoost]] vyberá iba najinformatívnejšiu podmnožinu.

**Príkladné diskriminatívne črty pri detekcii tváre.** Výskumníci zistili, že AdaBoost typicky vyberie ako prvé niekoľko vizuálne zmysluplných čŕt: (1) horizontálna dvojobdĺžniková črta cez oči a líca — zachytáva, že oblasť očí je tmavšia ako líca, (2) vertikálna trojobdĺžniková črta cez nozdrily a strednú časť nosa — zachytáva svetlý mostík nosa, (3) dvojobdĺžniková črta na oblasť úst. Tieto prvé vybrané črty majú silnú diskriminatívnu silu a sú zodpovedné za väčšinu hrubého filtrovania v [[attentional cascade]].

**Rýchlosť výpočtu.** Kľúčová vlastnosť Haarových čŕt je, že hodnotu ľubovoľnej čŕty možno vypočítať pomocou konštantného počtu operácií vďaka [[integral image]], nezávisle od veľkosti detekčného okna. Skenovaním okna po celom obraze s rôznymi škálami možno vykonať milióny výpočtov za sekundu na štandardnom hardvéri.

## Kde sa používa

- [[03 Viola-Jones framework]] — základný stavebný prvok detektora tváre
- Detekcia chodcov, evidenčných čísel, úsmevu, očí (variant Viola-Jones pre rôzne objekty)
- Historicky tiež v sledovaní objektov (*tracking*) pred érou hlbokého učenia

## Súvisiace pojmy

- [[integral image]]
- [[AdaBoost]]
- [[attentional cascade]]
- [[03 Viola-Jones framework]]
