# Scale Space and Gaussian Pyramid

*Priestor škál (scale space)* je matematický rámec, ktorý umožňuje analyzovať obraz súčasne na viacerých úrovniach rozlíšenia. Gaussova pyramída je jeho diskrétnou aproximáciou.

## Detaily

**Prečo priestor škál?**
Skutočné objekty existujú v rôznych veľkostiach. Chceme detekovať čiernu škvrnu bez ohľadu na to, či má priemer 3 px alebo 30 px. Kľúčová myšlienka: obraz filtrovaný Gaussianom so smerodajnou odchýlkou σ „nevidí" detaily menšie ako ~σ. Zväčšovaním σ teda postupne eliminujeme jemné štruktúry a zostávajú len veľké.

**Kontinuálna definícia:**
Priestor škál $L(x, y, \sigma)$ je konvolúcia vstupného obrazu $I(x,y)$ s Gaussianom:
$$L(x, y, \sigma) = G(x, y, \sigma) * I(x, y)$$
$$G(x, y, \sigma) = \frac{1}{2\pi\sigma^2} e^{-(x^2 + y^2)/(2\sigma^2)}$$

**Gaussova pyramída (diskrétna aproximácia):**
SIFT buduje pyramídu tvorenou *oktávami* (*octave*). V každej oktáve je obraz postupne konvolvovaný Gaussianmi s rastúcimi σ, potom sa obraz zmenší na polovicu (subsampling 2×) a postup sa opakuje.

- Počet oktáv: zvyčajne 4 (kým obraz nie je príliš malý)
- Počet mieriek (intervalov) v oktáve: $s = 3$; celkovo $s + 3 = 6$ obrázkov na oktávu (kvôli DoG výpočtu)
- Počiatočné σ: $\sigma_0 = 1{,}6$; medzi susednými vrstvami: $\sigma_k = \sigma_0 \cdot k^{1/s}$
- Subsampling 2× medzi oktávami zodpovedá zdvojeniu σ

Výsledkom je sada vyhladzovaných obrazov — základ pre výpočet [[Difference of Gaussians]].

**Biologická motivácia:**
Priestor škál zodpovedá tomu, ako ľudský vizuálny systém integruje informácie na rôznych priestorových frekvenciách (V1 → V4 → IT cortex v hierarchii od jemných po hrubé črty).

## Matematika

Gaussovská konvolúcia splňa **semi-group property**: $G(\sigma_1) * G(\sigma_2) = G(\sqrt{\sigma_1^2 + \sigma_2^2})$, čo umožňuje budovať pyramídu inkrementálne.

Obraz v oktáve $o$ a mierke $s$:
$$L(x, y, o, s) = G(x, y, \sigma(o,s)) * I_o(x, y)$$
kde $I_o$ je obraz subsamplovaný $2^o$-krát.

## Kde sa používa

- [[01 SIFT — scale space and keypoint detection]] — základ detekcie kľúčových bodov
- [[04 SURF — Speeded-Up Robust Features]] — SURF tiež buduje multi-scale priestor, ale pomocou Hessian filtrov
- Detekcia blob-ov na rôznych mierkach (astronómia, medicína)
- Multiscale analýza obrazov v satelitnom zobrazovaní

## Súvisiace pojmy

[[Difference of Gaussians]], [[01 SIFT — scale space and keypoint detection]], [[04 SURF — Speeded-Up Robust Features]]
