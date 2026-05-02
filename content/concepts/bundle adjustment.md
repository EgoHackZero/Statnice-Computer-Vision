# Bundle Adjustment

*Bundle adjustment (BA)* je nelineárna optimalizácia, ktorá súčasne spresňuje polohy všetkých kamier aj 3D súradnice všetkých bodov minimalizáciou celkovej reprojekčnej chyby.

## Motivácia

Inkrementálny SfM postupne pridáva kamery. Každý odhad kamery má malú chybu → chyby sa kumulujú (*drift*). Po 50+ kamerách môže byť model výrazne deformovaný. Bundle adjustment eliminuje tento drift globálnou optimalizáciou.

## Matematická formulácia

$$\min_{P, M} E(P, M) = \sum_{j} \sum_{i \in V(j)} \rho\!\left(\left\| \pi(P_i, M_j) - \mathbf{m}_i^j \right\|^2\right)$$

kde:
- $P_i = (K_i, R_i, \mathbf{t}_i)$: parametre kamery $i$ (intrinsics + extrinsics)
- $M_j = (X_j, Y_j, Z_j)$: 3D súradnice bodu $j$
- $\pi(P_i, M_j)$: projekcia 3D bodu $M_j$ do obrazu kamery $i$
- $\mathbf{m}_i^j$: skutočne pozorovaná 2D poloha bodu $j$ v obraze $i$
- $V(j)$: množina kamier, z ktorých je bod $j$ viditeľný
- $\rho(\cdot)$: robustifikačná funkcia (Huber loss) pre odolnosť voči outlierom

## Reprojekčná chyba

Pre jeden bod v jednom obraze:
$$e_{ij} = \left\| \pi(P_i, M_j) - \mathbf{m}_i^j \right\| \quad [\text{px}]$$

Dobrá BA: stredná reprojekčná chyba < 1 px; typicky 0.3–0.8 px po konvergencii.

## Riešenie: Levenberg-Marquardt

BA sa rieši iteratívnou optimalizáciou **Levenberg-Marquardt (LM)** algoritmom:

$$\left(J^T W J + \lambda I\right) \delta = J^T W r$$

kde:
- $J$: Jakobiánova matica (parciálne derivácie reprojekcie podľa parametrov)
- $r$: vektor reziduálov (reprojekčné chyby)
- $\lambda$: damping parameter (interpoluje medzi gradient descent a Gauss-Newton)

### Sparse BA

BA má špeciálnu *sparsity* štruktúru: kamera $i$ a bod $j$ sú prepojené iba ak $j \in V(i)$. Jakobián $J$ je riedky → Normal equations $(J^T J)$ majú blokovú štruktúru → *Schur complement trick* umožňuje efektívne riešenie.

## Lokálny vs. globálny BA

| | Lokálny BA | Globálny BA |
|---|---|---|
| Rozsah | Posledných $k$ kamier + ich body | Všetky kamery + všetky body |
| Rýchlosť | Rýchly (real-time) | Pomalý (minuty–hodiny) |
| Presnosť | Akumuluje drift | Eliminuje drift |
| Použitie | SLAM (online) | SfM/MVS (offline) |

## Veľkosť problému

Pre $m$ kamier s 6 parametrami a $n$ bodov s 3 súradnicami:
- Celkovo: $6m + 3n$ parametrov
- Napr. COLMAP na 1000 fotiek: $\approx 6000 + 3 \times 10^6$ parametrov

## Implementácie

- **Ceres Solver** (Google): C++ BA knižnica; COLMAP, ORB-SLAM3 ju používajú
- **g2o** (Graph Optimization): populárna v SLAM komunitách
- **gtsam**: Georgia Tech Smoothing and Mapping; faktor-grafová BA
- **scipy.optimize.least_squares** (Python): pre malé problémy

## Súvisiace pojmy

[[10 Structure-from-Motion a Multi-View Stereo]] · [[triangulation]] · [[camera intrinsics and Zhang calibration]] · [[dense MVS and depthmap fusion]]
