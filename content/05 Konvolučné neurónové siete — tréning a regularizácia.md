# 05 Konvolučné neurónové siete — tréning a regularizácia

> *Otázka: Konvolučné neurónové siete, batch normalizácia, L1 a L2 normalizácia, podmnožiny dátovej množiny pri učení, príprava a normalizácia dát.*

## Stručný prehľad

Konvolučné neurónové siete (*Convolutional Neural Networks*, CNN) sú základným nástrojom moderného počítačového videnia. Ich tréning si však vyžaduje starostlivú prípravu dát, správnu regularizáciu a dobre nastavenú normalizáciu aktivácií. Táto kapitola pokrýva päť prepojených tém: architektúru CNN, [[batch normalization]], [[L1 and L2 regularization]], [[train-validation-test split]] a [[data normalization and class imbalance]].

---

## A. Konvolučné neurónové siete — základné koncepty

### Prečo CNN fungujú pri spracovaní obrazu

Tri kľúčové vlastnosti predisponujú CNN na spracovanie obrazu:

1. **Lokálne spojenie** (*local connectivity*) — konvolučný filter sa aplikuje iba na malú lokálnu oblasť vstupu (napr. 3×3 pixelov). Obrázky majú lokálnu priestorovú štruktúru: susedné pixely sú vzájomne korelované, vzdialené menej. Plne prepojená vrstva by túto štruktúru ignorovala a vyžadovala by oveľa viac parametrov.

2. **Zdieľanie váh** (*weight sharing*) — ten istý filter (s rovnakými váhami) sa posúva po celom obrázku. Detektor hrán platí rovnako v ľavom hornom rohu aj v strede obrázka. Výsledkom je drastické zníženie počtu parametrov a implicitná invariantnosť voči translácii.

3. **Pooling** (*združovanie*) — typicky *max-pooling* berie maximum z malého okna (2×2) a posúva sa s krokom 2. Znižuje priestorové rozlíšenie, čím zvyšuje invariantnosť voči malým posunom a deformáciám, a zároveň znižuje výpočtovú záťaž.

### Konvolučná operácia

Pre vstup $X$ a filter $W$ s rozmermi $k \times k$ je výstupná *feature mapa* definovaná:

$$y[i,j] = \sum_{m=0}^{k-1} \sum_{n=0}^{k-1} X[i+m,\; j+n] \cdot W[m,n] + b$$

kde $b$ je bias. *Receptívne pole* (*receptive field*) neuronu v hlbšej vrstve pokrýva čím ďalej tým väčšiu oblasť pôvodného vstupu.

### Hierarchia čŕt

Hlbšie siete sú výhodnejšie ako širšie pretože umožňujú hierarchické učenie reprezentácií:
- **Nižšie vrstvy** detegujú jednoduché vzory: hrany, rohy, textúry.
- **Stredné vrstvy** kombinujú tieto prvky do čiastkových tvarov: kolesá, oči, ušnice.
- **Vyššie vrstvy** rozpoznávajú celé objekty: autá, tváre, zvieratá.

Hlboká sieť dokáže vyjadriť exponenciálne zložitejšie funkcie s lineárnym nárastom počtu vrstiev, zatiaľ čo rovnaká kapacita v jednej širokej vrstve by vyžadovala exponenciálne viac neurónov.

### Architektúra AlexNet (2012)

*AlexNet* bol prelomový model, ktorý vyhral ImageNet LSVRC 2012 s výrazným náskokom. Skladá sa z:

| Vrstva | Popis |
|---|---|
| Conv1 | 96 filtrov 11×11, stride 4, ReLU, MaxPool |
| Conv2 | 256 filtrov 5×5, padding, ReLU, MaxPool |
| Conv3 | 384 filtrov 3×3, ReLU |
| Conv4 | 384 filtrov 3×3, ReLU |
| Conv5 | 256 filtrov 3×3, ReLU, MaxPool |
| FC6 | 4096 neurónov, ReLU, **Dropout** |
| FC7 | 4096 neurónov, ReLU, **Dropout** |
| FC8 | 1000 neurónov, **Softmax** |

Kľúčové inovácie AlexNetu: použitie **ReLU** aktivácie (rýchlejšia konvergencia než sigmoid/tanh), **Dropout** v plne prepojených vrstvách, tréning na dvoch GPU paralelne.

---

## B. Batch normalizácia

[[batch normalization]] je technika, ktorá normalizuje aktivácie každej vrstvy v rámci mini-dávky (*mini-batch*) na nulový priemer a jednotkový rozptyl.

### Motivácia: interná kovariantná zmena

*Interná kovariantná zmena* (*internal covariate shift*) je jav, kde sa distribúcia vstupov do každej vrstvy mení počas trénovania, pretože sa menia váhy predchádzajúcich vrstiev. Každá vrstva sa musí neustále prispôsobovať novým distribúciám, čo spomaľuje tréning a vyžaduje opatrné inicializácie a nízke *learning rate*.

### Algoritmus batch normalizácie

Pre mini-dávku $\mathcal{B} = \{x_1, x_2, \ldots, x_m\}$:

**Krok 1 — výpočet štatistík dávky:**
$$\mu_{\mathcal{B}} = \frac{1}{m} \sum_{i=1}^{m} x_i, \qquad \sigma_{\mathcal{B}}^2 = \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_{\mathcal{B}})^2$$

**Krok 2 — normalizácia:**
$$\hat{x}_i = \frac{x_i - \mu_{\mathcal{B}}}{\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}}$$

kde $\epsilon \approx 10^{-5}$ zabraňuje deleniu nulou.

**Krok 3 — škálovanie a posunutie:**
$$y_i = \gamma \hat{x}_i + \beta$$

Parametre $\gamma$ (*scale*) a $\beta$ (*shift*) sú **trénovateľné** — sieť sa naučí, akú distribúciu aktivácií je pre danú vrstvu optimálna. Bez nich by normalizácia mohla zničiť reprezentačnú silu vrstvy (napr. sigmoid by fungoval len v lineárnom režime).

### Tréning vs. inferencia

Toto je **kritický rozdiel**, ktorý sa často pýta na skúške:

| Fáza | Použité štatistiky |
|---|---|
| **Tréning** | Priemer $\mu_{\mathcal{B}}$ a rozptyl $\sigma_{\mathcal{B}}^2$ aktuálnej mini-dávky |
| **Inferencia** | Bežiaci priemer (*running mean*) a bežiaci rozptyl (*running variance*) akumulované počas trénovania |

Počas trénovania sa bežiace štatistiky aktualizujú pomocou exponenciálneho kĺzavého priemeru (*exponential moving average*):
$$\mu_{\text{running}} \leftarrow (1-\alpha)\,\mu_{\text{running}} + \alpha\,\mu_{\mathcal{B}}$$

kde $\alpha$ (napr. 0.1) je *momentum* aktualizácie. Pri inferencii sa model správa deterministicky — každý vstup dostane rovnaké normalizačné konštanty bez ohľadu na ostatné vzorky v dávke.

### Umiestnenie batch normalizácie

Odporúčané poradie vrstiev:
$$\text{Conv} \;\rightarrow\; \text{BN} \;\rightarrow\; \text{ReLU}$$

Batch normalizácia sa vkladá **pred aktivačnú funkciu**, aby zabezpečila, že vstup do aktivácie je vždy v rozumnom rozsahu. Niektoré novšie práce navrhujú aj poradie Conv → ReLU → BN, ale Conv → BN → ReLU je štandardom.

### Výhody batch normalizácie

- **Stabilizuje tréning** — menšia citlivosť na inicializáciu váh.
- **Umožňuje vyššie learning rate** — gradient neexploduje, konvergencia je rýchlejšia.
- **Pôsobí ako regularizátor** — šum z mini-dávkových štatistík zabraňuje overfittingu (čiastočne nahrádza Dropout).
- **Zmenšuje efekt internej kovariantnej zmeny** — vrstvy sa učia stabilnejšie.

---

## C. L1 a L2 regularizácia

[[L1 and L2 regularization]] sú techniky pridávajúce penalizačný člen k stratovej funkcii, aby zabránili overfittingu.

### L2 regularizácia (weight decay, ridge)

Penalizačný člen:
$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{task}} + \frac{\lambda}{2} \sum_i w_i^2$$

Gradient: $\frac{\partial \mathcal{L}}{\partial w_i} = \frac{\partial \mathcal{L}_{\text{task}}}{\partial w_i} + \lambda w_i$

Efekt: váhy sa rovnomerne zmenšujú proporcionálne k ich veľkosti (*weight decay*). Riešenie je husté — žiadna váha nie je presne nula. L2 je **najpoužívanejšia regularizácia v CNN** pretože má hladký, lineárny gradient vhodný pre gradient descent.

### L1 regularizácia (lasso)

Penalizačný člen:
$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{task}} + \lambda \sum_i |w_i|$$

Gradient: $\frac{\partial \mathcal{L}}{\partial w_i} = \frac{\partial \mathcal{L}_{\text{task}}}{\partial w_i} + \lambda \cdot \text{sign}(w_i)$

Efekt: konštantná penalizácia bez ohľadu na veľkosť váhy → niektoré váhy sú zatlačené presne na nulu → **riedke riešenie** (*sparse solution*). L1 robí implicitnú selekciu príznakov.

### Elastic net

Kombinácia L1 a L2:
$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{task}} + \lambda_1 \sum_i |w_i| + \frac{\lambda_2}{2} \sum_i w_i^2$$

### Porovnanie L1 vs. L2

| Vlastnosť | L1 | L2 |
|---|---|---|
| Tvar penalizácie | $\sum |w_i|$ | $\frac{1}{2}\sum w_i^2$ |
| Typ riešenia | Riedke (mnohé váhy = 0) | Husté (mnohé malé váhy) |
| Selekcia príznakov | Áno (implicitná) | Nie |
| Optimalizácia | Zložitejšia (nediferencovateľná v 0) | Jednoduchá (hladký gradient) |
| Typické použitie | Feature selection, interpretovateľnosť | CNN tréning (weight decay) |

---

## D. Podmnožiny dátovej množiny pri učení

[[train-validation-test split]] — rozdelenie dát na tri nezávislé množiny je základnou požiadavkou správneho strojového učenia.

### Tri podmnožiny a ich účel

**Tréningová množina** (*training set*):
- Slúži na **optimalizáciu parametrov** modelu (váhy siete).
- Model "vidí" tieto dáta opakovane počas trénovania.
- Chyba na trénovacej množine sa minimalizuje priamo.

**Validačná množina** (*validation set*):
- Slúži na **výber hyperparametrov**: learning rate, hĺbka siete, veľkosť dávky, miera Dropoutu, λ pre regularizáciu.
- Umožňuje **early stopping** — zastavenie trénovania keď validačná chyba prestane klesať.
- Umožňuje **výber architektúry** — porovnanie rôznych sietí.
- Model "nevidí" tieto dáta pri optimalizácii váh, ale nepriamo ich ovplyvňuje cez výber hyperparametrov.

**Testovacia množina** (*test set*):
- Slúži **výhradne na finálne vyhodnotenie** po ukončení celého procesu vývoja.
- Nikdy sa nesmie použiť na ladenie — inak dochádza k *data leakage* a výsledky nie sú nezaujaté.
- Predstavuje aproximáciu výkonnosti na skutočnom nasadení.

### Typické pomery rozdelenia

| Veľkosť datasetu | Train | Validation | Test |
|---|---|---|---|
| Malý (< 10 000) | 60 % | 20 % | 20 % |
| Stredný | 70 % | 15 % | 15 % |
| Veľký (> 1 M) | 98 % | 1 % | 1 % |

### Pravidlá správneho rozdelenia

- **i.i.d. požiadavka** (*independent and identically distributed*) — každá podmnožina musí byť reprezentatívna pre celú distribúciu dát. Náhodné miešanie pred rozdelením.
- **Stratifikácia** (*stratification*) — pri klasifikácii zachovať proporcie tried v každej podmnožine. Ak je 10 % vzoriek triedy "choroba", každá podmnožina musí obsahovať ~10 % tejto triedy.
- **Chronologické poradie** — pri časových radoch sa nesmú budúce dáta dostať do trénovacej množiny (*data leakage* cez čas). Tréning = minulosť, test = budúcnosť.

### k-násobná krížová validácia

Pre malé datasety sa používa **k-fold cross-validation**:
1. Dataset sa rozdelí na $k$ rovnakých foldov.
2. Model sa trénuje $k$-krát, pričom zakaždým iný fold slúži ako validačná množina.
3. Výsledná metrika je priemer cez všetky $k$ behov.

Typicky $k = 5$ alebo $k = 10$. Znižuje variabilitu odhadu výkonnosti, ale je $k$-krát drahší.

---

## E. Príprava a normalizácia dát

[[data normalization and class imbalance]] — kvalita vstupných dát priamo určuje kvalitu natrénovaného modelu.

### Čistenie dát

Pred trénovaním je nevyhnutné:
- **Odstránenie chýbajúcich hodnôt** — imputácia (priemer, medián) alebo odstránenie vzoriek.
- **Odstránenie duplikátov** — duplicitné vzorky môžu skresliť rozdelenie trénovacej a testovacej množiny.
- **Detekcia outlierov** — extrémne hodnoty môžu destabilizovať gradient descent.

### Normalizácia príznakov

Gradient descent konverguje oveľa rýchlejšie keď majú všetky príznaky podobné škály. Bez normalizácie sa kontúry stratovej funkcie stávajú elipsoidmi s extrémnym pomerom osí, čo spomaľuje konvergenciu.

**Min-Max normalizácia** (škálovanie do [0, 1]):
$$x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}}$$

**Z-score štandardizácia** (nulový priemer, jednotková odchýlka):
$$x' = \frac{x - \mu}{\sigma}$$

Z-score je preferovaná pri CNN, pretože je robustnejšia voči outlierom a vhodnejšia pre ReLU aktivácie.

### Normalizácia obrazových dát

Pre obrázky sa používa normalizácia po kanáloch pomocou štatistík celého datasetu. Pre ImageNet:
$$\text{mean} = [0.485,\; 0.456,\; 0.406], \quad \text{std} = [0.229,\; 0.224,\; 0.225]$$

Pre každý kanál $c$:
$$\text{pixel}'_{c} = \frac{\text{pixel}_c - \text{mean}_c}{\text{std}_c}$$

### Augmentácia dát

*Data augmentation* umelé rozširuje trénovací dataset aplikovaním transformácií:

| Typ augmentácie | Príklady |
|---|---|
| Geometrická | Horizontálny flip, rotácia ±15°, zoom, crop |
| Farebná | Zmena jasu, kontrastu, saturácie (*color jitter*) |
| Pokročilá | CutMix (zmiešanie oblastí dvoch obrázkov), MixUp (lineárna interpolácia obrázkov a ich labelov) |
| Pre robustnosť | Gaussian noise, rozmazanie, JPEG kompresný artefakt |

Augmentácia sa aplikuje **len na trénovaciu množinu**, nie na validačnú ani testovaciu.

### Nevyvážené triedy

*Class imbalance* (nevyvážené triedy) je časté v medicínskom zobrazovaní, bezpečnostných systémoch a detekcii anomálií.

**Riešenia:**
- **Oversampling** minority triedy: SMOTE (*Synthetic Minority Over-sampling Technique*) generuje syntetické vzorky.
- **Undersampling** majority triedy: náhodné odstraňovanie vzoriek.
- **Váhovaná strata** (*weighted loss*): triede s menším počtom vzoriek sa priradí väčšia váha $w_c = \frac{N}{k \cdot N_c}$.
- **Focal loss**: $\mathcal{L}_{\text{focal}} = -\alpha_t (1-p_t)^\gamma \log(p_t)$, kde $(1-p_t)^\gamma$ znižuje príspevok ľahko klasifikovaných vzoriek.

---

## Matematika — zhrnutie kľúčových vzorcov

**Batch Normalization (tréning):**
$$\hat{x}_i = \frac{x_i - \mu_{\mathcal{B}}}{\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}}, \qquad y_i = \gamma \hat{x}_i + \beta$$

**L2 regularizácia:**
$$\mathcal{L} = \mathcal{L}_{\text{task}} + \frac{\lambda}{2} \sum_i w_i^2$$

**L1 regularizácia:**
$$\mathcal{L} = \mathcal{L}_{\text{task}} + \lambda \sum_i |w_i|$$

**Z-score normalizácia:**
$$x' = \frac{x - \mu}{\sigma}$$

**Focal loss:**
$$\mathcal{L}_{\text{focal}} = -\alpha_t (1-p_t)^\gamma \log(p_t)$$

---

## Porovnania a kompromisy

| Technika | Výhoda | Nevýhoda |
|---|---|---|
| Batch Normalization | Rýchlejší tréning, vyššie LR | Závislosť na veľkosti dávky; problém pri malom batch size |
| L2 regularizácia | Hladká optimalizácia, husté riešenie | Nevykoná selekciu príznakov |
| L1 regularizácia | Riedke riešenie, selekcia príznakov | Nediferencovateľná v 0, nestabilná optimalizácia |
| Dropout | Silná regularizácia, lacné na výpočet | Spomaľuje konvergenciu; nevhodné s BN |
| Data augmentation | Zlepšuje generalizáciu bez nových dát | Môže zaviesť nerealistické vzorky |
| k-fold CV | Spoľahlivejší odhad výkonnosti | k-krát vyššia výpočtová záťaž |

---

## Časté zlyhania a obmedzenia

- **Batch size = 1**: Batch normalizácia zlyhá — štatistika z jednej vzorky je nereliabilná. Riešenie: Group Normalization alebo Layer Normalization.
- **Malý dataset bez augmentácie**: Rýchly overfit bez regularizácie.
- **Data leakage cez normalizáciu**: Ak sa počítajú štatistiky (mean, std) na celom datasete vrátane testovacích dát, testovacia metrika je optimisticky skreslená. Štatistiky sa smú počítať len z trénovacej množiny.
- **Nevyváženosť tried ignorovaná**: Model sa naučí predpovedať vždy majoritnú triedu — vysoká presnosť, nulová sensitivita.
- **Testovacia množina použitá na ladenie**: Výsledky sú optimisticky skreslené a nereprezentujú skutočnú generalizáciu.

---

## Praktické využitie

**PyTorch:**
```python
import torch.nn as nn
import torchvision.transforms as transforms

# Batch Normalization v CNN
model = nn.Sequential(
    nn.Conv2d(3, 64, kernel_size=3, padding=1),
    nn.BatchNorm2d(64),
    nn.ReLU(inplace=True)
)

# Normalizácia obrazov (ImageNet štatistiky)
normalize = transforms.Normalize(
    mean=[0.485, 0.456, 0.406],
    std=[0.229, 0.224, 0.225]
)

# L2 regularizácia cez weight_decay
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3, weight_decay=1e-4)
```

**scikit-learn:**
```python
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import StratifiedKFold

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)  # použiť train štatistiky!

skf = StratifiedKFold(n_splits=5)
```

**Aplikačné oblasti:**
- **Medicínske zobrazovanie**: Silná nevyváženosť tried (napr. nádor vs. zdravé tkanivo) → focal loss, weighted cross-entropy; BN kritická pre stabilitu.
- **Autonómne vozidlá**: ImageNet predtrénované modely + fine-tuning; augmentácia slnečného svetla, dažďa, noci.
- **Priemyselná inšpekcia**: Chybné kusy sú zriedkavé → oversampling, SMOTE.

---

## Súvisiace pojmy

- [[batch normalization]]
- [[L1 and L2 regularization]]
- [[train-validation-test split]]
- [[data normalization and class imbalance]]
- [[skip connections]]

---

*Spýtaj sa sám seba:*
1. Ako sa líši správanie batch normalizácie počas trénovania a počas inferencie? Prečo je tento rozdiel dôležitý?
2. Kedy by si uprednostnil L1 regularizáciu pred L2? Uveď konkrétny príklad.
3. Prečo je potrebná samostatná testovacia množina — nestačí trénovacia + validačná?
4. Čo spraví z-score štandardizácia s distribúciou príznakov? Čo sa stane ak ju neaplikuješ?
5. Čo je stratifikácia pri rozdelení dát a kedy je nevyhnutná?
