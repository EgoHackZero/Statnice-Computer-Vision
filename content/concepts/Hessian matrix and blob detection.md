# Hessian Matrix and Blob Detection

*Hessianová matica (Hessian matrix)* obrazu je matica druhých parciálnych derivácií intenzity. Jej determinant sa používa ako efektívny detektor blob-ov — základ algoritmu [[04 SURF — Speeded-Up Robust Features]].

## Detaily

**Hessianová matica:**
Pre obraz $I(x,y)$ konvolvovaný Gaussianom so σ je Hessianová matica:
$$H(x, y, \sigma) = \begin{pmatrix} L_{xx}(x,y,\sigma) & L_{xy}(x,y,\sigma) \\ L_{xy}(x,y,\sigma) & L_{yy}(x,y,\sigma) \end{pmatrix}$$
kde $L_{xx} = \frac{\partial^2 L}{\partial x^2}$, $L_{yy} = \frac{\partial^2 L}{\partial y^2}$, $L_{xy} = \frac{\partial^2 L}{\partial x \partial y}$.

**Prečo determinant deteguje blob-y:**
- Vlastné čísla $\lambda_1, \lambda_2$ Hessianovej matice vyjadrujú hlavné krivosti intenzitnej plochy
- **Blob** (škvrna): obe vlastné čísla veľké a rovnakého znamienka → $\det(H) = \lambda_1 \lambda_2 > 0$, $\det(H)$ veľký
- **Hrana**: jedno veľké, jedno malé vlastné číslo → $\det(H) \approx 0$
- **Ploché miesto**: obe vlastné čísla malé → $\det(H) \approx 0$

$$\det(H) = L_{xx} L_{yy} - L_{xy}^2$$

Maximá $\det(H)$ v priestore mierok odpovedajú centrám blob-ov príslušnej veľkosti.

**Rýchlosť cez box filtre:**
Analytické výpočet $L_{xx}$, $L_{yy}$, $L_{xy}$ vyžaduje konvolúciu s druhými deriváciami Gaussiánu — náročná operácia pri veľkom σ. SURF (Bay et al. 2006) aproximuje tieto deriváty pomocou *box filtrov* (obdĺžnikové filtre s hodnotami ±1) aplikovaných cez [[integral image]]. Výsledok: $\det(H)$ sa vypočíta v konštantnom čase $O(1)$ **nezávisle od mierky** (veľkosti filtra).

**Porovnanie s DoG (SIFT):**
| Vlastnosť | DoG (SIFT) | det(H) (SURF) |
|---|---|---|
| Zachytáva | Blob-y (izotropné) | Blob-y (izotropné) |
| Výpočet | Rozdiel Gaussiánov | Box-filter aproximácia |
| Rýchlosť | Stredná | Rýchla (integrálny obraz) |
| Rotačná invariantnosť | Áno | Áno (det je invariantný) |

**Detekcia v priestore mierok:**
SURF počíta $\det(H_{approx})$ pre box filtre veľkostí 9×9, 15×15, 21×21, ... (zodpovedajúcich rôznym σ). Lokálne maximá v tomto 3D priestore (x, y, mierka) sú kandidátmi na kľúčové body.

## Kde sa používa

- [[04 SURF — Speeded-Up Robust Features]] — primárny detektor kľúčových bodov
- Detekcia rohov: Harrisov detektor používa Hessianovú maticu na odlíšenie hrán, rohov a plôch
- Analýza krivosti povrchov v medicíne (CT skulptúra kosti)
- Optický tok pri detekcii charakteristických bodov

## Súvisiace pojmy

[[04 SURF — Speeded-Up Robust Features]], [[integral image]], [[Difference of Gaussians]], [[01 SIFT — scale space and keypoint detection]]
