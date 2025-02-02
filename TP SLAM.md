
# Questions préliminaires

## **1. Data Simulation**

### **a. Information fournie par l'animation graphique**

Si l’animation est activée (`plot_p = 1`), elle affiche les éléments suivants en temps réel :

1.  **Mouvement du robot** :
    
    -   Le robot suit une trajectoire perturbée autour d’un cercle idéal centré en `CC` (centre du cercle).
    -   La position réelle est marquée par un point bleu.
2.  **Repères (landmarks)** :
    
    -   Les positions des repères sont affichées sous forme de carrés.
    -   Les repères visibles (dans le champ de vision du robot) sont colorés en **vert**.
    -   Les repères non visibles sont colorés en **rouge**.
3.  **Trajectoire du robot** :
    
    -   La trajectoire actuelle du robot est tracée avec une ligne en **pointillés bleus**.
4.  **Champ de vision** :
    
    -   Un domaine triangulaire (en cyan transparent) illustre la zone dans laquelle le robot peut observer les repères.
5.  **Éléments visuels supplémentaires** :
    
    -   Les coordonnées relatives des repères visibles sont représentées avec des flèches.
    -   Les noms des repères visibles (ex. : `A1`, `A2`, etc.) sont affichés pour indiquer leur identification.

Cette animation aide à comprendre comment :

-   Les mesures sont influencées par la visibilité des repères.
-   Le mouvement du robot et son champ de vision affectent la collecte des données.

----------

### **b. Signification des variables de sortie**

#### **Variables principales**

1.  **`N`** : Nombre total de pas de temps simulés.
    
2.  **`T`** : Vecteur des instants de temps (1xN).
    
3.  **`M`** : Nombre de repères fixes dans l’environnement.
    
4.  **`Z`** : Matrice (2MxN) des mesures bruitées des positions relatives des repères visibles.
    
    -   Les lignes contiennent les coordonnées relatives `x` et `y` pour chaque repère.
    -   Les colonnes correspondent aux instants de temps.
    -   Les valeurs `NaN` indiquent qu'un repère est hors du champ de vision.
5.  **`arrayH`** : Matrice 3D des matrices d’observation pour chaque instant (2Mx(2+2M)xN).
    
    -   Ces matrices décrivent la relation entre les mesures et les états cachés.
6.  **`arrayR`** : Matrice 3D des covariances du bruit de mesure (2Mx2MxN).
    
    -   Elle décrit l’incertitude associée aux mesures des repères visibles.
7.  **`in_fov`** : Matrice binaire (2MxN) indiquant la visibilité des repères.
    
    -   `1` si le repère est visible, `NaN` sinon.

#### **Variables dynamiques**

8.  **`F`** : Matrice de transition d’état ((2+2M)x(2+2M)).
    
    -   Elle modélise la dynamique du robot et des repères.
9.  **`Qw`** : Matrice de covariance du bruit dynamique ((2+2M)x(2+2M)).
    
    -   Elle quantifie l'incertitude dans la dynamique de l’état.
10.  **`B`** : Vecteur de contrôle ((2+2M)x1).
    
    -   Contient les termes constants liés à la dynamique (centre du cercle).

#### **Variables liées aux mesures**

11.  **`Hfull`** : Matrice d'observation complète (2Mx(2+2M)).
    
    -   Utilisée si tous les repères sont visibles.
12.  **`Rv`** : Matrice de covariance du bruit de mesure (2Mx2M).
    
    -   Elle quantifie l’incertitude des mesures.

#### **Variables initiales**

13.  **`mX0`** : Espérance de l'état initial ((2+2M)x1).
    
    -   Comprend la position initiale supposée du robot et des repères.
14.  **`PX0`** : Covariance de l'état initial ((2+2M)x(2+2M)).
    
    -   Quantifie l'incertitude initiale.

#### **Variables résultantes**

15.  **`X`** : Matrice (2+2M)xN des états cachés réels (positions réelles du robot et des repères).
    -   Chaque colonne correspond à un instant de temps.


### **c.  Pourquoi le filtre de Kalman fournit une solution exacte à l'estimation bayésienne du vecteur d'état caché ?**
Le filtre de Kalman fournit une solution exacte parce qu'il exploite parfaitement les hypothèses de linéarité et de bruit gaussien pour calculer analytiquement les moments de la distribution a posteriori. Cela garantit que l’estimation de l’état est optimale dans ce cadre spécifique.


## **2. Probability density function**

### **a. PDF Initiale $p(x_0)$**

Le vecteur d'état initial $x_0$ inclut la position initiale du robot et les positions des landmarks. Cet état est modélisé comme une distribution gaussienne multivariée :

$$
x_0 \sim \mathcal{N}(m_{x_0}, P_{x_0}),
$$

où :
- $m_{x_0}$ est le vecteur moyen de l'état initial (par exemple, la position initiale du robot et les estimations antérieures des positions des landmarks).
- $P_{x_0}$ est la matrice de covariance de l'état initial, capturant les incertitudes dans les positions initiales du robot et des landmarks.

La PDF est donnée par :

$$
p(x_0) = \frac{1}{\sqrt{(2\pi)^{d_x} \det(P_{x_0})}} \exp\left(-\frac{1}{2}(x_0 - m_{x_0})^\top P_{x_0}^{-1} (x_0 - m_{x_0})\right),
$$

où $d_x$ est la dimension du vecteur d'état $x_0$.

---

### **b. PDF de Dynamique Prioritaire $p(x_k | x_{k-1})$**

Le modèle de transition d'état décrit comment l'état évolue de $k-1$ à $k$ :

$$
x_k = F x_{k-1} + B + w_k,
$$

où :
- $F$ est la matrice de transition d'état.
- $B$ est un vecteur de contrôle (par exemple, prenant en compte la dynamique circulaire du mouvement du robot).
- $w_k \sim \mathcal{N}(0, Q_w)$ représente le bruit de processus.

La PDF de la dynamique prioritaire est :

$$
p(x_k | x_{k-1}) = \frac{1}{\sqrt{(2\pi)^{d_x} \det(Q_w)}} \exp\left(-\frac{1}{2}(x_k - F x_{k-1} - B)^\top Q_w^{-1} (x_k - F x_{k-1} - B)\right)
$$

---

### **c. PDF de Mesure $p(z_k | x_k)$**

Le modèle de mesure relie l'état aux observations :

$$
z_k = H x_k + v_k,
$$

où :
- $H$ est la matrice d'observation, reliant l'espace d'état à l'espace de mesure.
- $v_k \sim \mathcal{N}(0, R)$ est le bruit de mesure.

La PDF est :

$$
p(z_k | x_k) = \frac{1}{\sqrt{(2\pi)^{d_z} \det(R)}} \exp\left(-\frac{1}{2}(z_k - H x_k)^\top R^{-1} (z_k - H x_k)\right),
$$

où :
- $d_z$ est la dimension du vecteur de mesure $z_k$.
- $R$ est la matrice de covariance du bruit de mesure.

#### **Visibilité des Landmarks**

Vu que seulement une sous-partie des landmarks est visible au temps $k$, la matrice d'observation $H$ et le vecteur de mesure $z_k$ seront adaptés pour inclure uniquement les landmarks visibles.

La taille de $H$ et $R$ sera donc ajustée dynamiquement :

$$
H_k = [H_{\text{visible}}], \quad R_k = \text{blkdiag}(R_{\text{visible}}).
$$

----



# Rapport : TP Filtres à particules
## **Introduction**
L'objectif de ce TP est d'étudier et de mettre en œuvre des filtres à particules pour l'estimation d'état dans un système dynamique. Deux variantes principales du filtre ont été développées :
1. **Filtre à Particules SIS (Sequential Importance Sampling)** : Utilise un modèle probabiliste pour pondérer les particules en fonction des observations disponibles.
2. **Filtre à Particules SIR (Sampling Importance Resampling)** : Ajoute une étape de rééchantillonnage pour résoudre le problème de dégénérescence des particules.

Les algorithmes ont été appliqués à une simulation de robot mobile évoluant dans un environnement contenant des repères fixes.

---

## **Simulation : Modèle du Système**

### **Données Simulées**
La fonction `dataSimulation` génère les données nécessaires :
- **Dynamique du système** : Modèle de transition $x_k = F x_{k-1} + B + w$, avec $w \sim \mathcal{N}(0, Q_w)$.
- **Mesures simulées** : $z_k = H x_k + v$, avec $v \sim \mathcal{N}(0, R_v)$.
- **Configuration** :
  - 5 repères fixes dans l'environnement.
  - 50 pas de temps pour la simulation.

---

## **Filtre à Particules SIS**

### **Étapes de l'algorithme SIS**
1. **Initialisation** :
   - $N_p = 100$ particules échantillonnées depuis la distribution initiale $p(x_0)$.
   - Poids initiaux égaux $w_0^{(i)} = 1/N_p$.
   
2. **Propagation** :
   - Les particules sont propagées à chaque instant selon le modèle dynamique du système:
   $$
   x_k^{(i)} \sim p(x_k | x_{k-1}^{(i)}), \quad i = 1, 2, \ldots, N_p
   $$

3. **Mise à Jour des Poids** :
   - Les poids des particules sont ajustés en fonction des observations disponibles :
     $$
     w_k^{(i)} \propto w_{k-1}^{(i)} \cdot p(z_k | x_k^{(i)})
     $$

4. **Estimation** :
   - L'état estimé $\hat{x}_k$ est calculé comme la moyenne pondérée des particules.
  $$
  \hat{x}_k = \sum_{i=1}^{N_p} w_k^{(i)} x_k^{(i)}
  $$

5. **Calcul de la Covariance** :
   - La covariance $P_k$ est calculée pour représenter l'incertitude sur $\hat{x}_k$:
  $$
  P_k = \sum_{i=1}^{N_p} w_k^{(i)} \cdot \big(x_k^{(i)} - \hat{x}_k\big) \big(x_k^{(i)} - \hat{x}_k\big)^\top
  $$

### **Limites du SIS**
Le SIS souffre de **dégénérescence des particules** : après plusieurs étapes, quelques particules seulement portent des poids significatifs.

---

## **Filtre à Particules SIR**

### **Amélioration avec Rééchantillonnage**
Le SIR inclut une étape de rééchantillonnage lorsque le **nombre effectif de particules** $N_{\text{eff}}$ devient trop faible :
$$
N_{\text{eff}} = \frac{1}{\sum_{i=1}^{N_p} w_k^{(i)2}}
$$
- $N_{\text{eff}}$​ diminue lorsque la distribution des poids devient très inégale (c'est-à-dire que peu de particules portent des poids significatifs).
- Si $N_{\text{eff}} < N_{\text{seuil}}$, les particules sont redistribuées en fonction de leurs poids.

### **Rééchantillonnage Systématique**
La méthode de utilisé est le rééchantillonnage systématique. Cette méthode garantit une redistribution efficace des particules en maintenant leur diversité.
- La somme cumulative des poids des particules associe chaque particule à une plage proportionnelle à son poids. 
- Ensuite, des positions uniformément réparties entre 0 et 1 sont générées avec un décalage aléatoire. 
$$
\text{positions } = \frac{1}{N_p} \big( \text{rand} + [0, 1, \ldots, N_p-1] \big)
$$
- Enfin, une particule est sélectionnée à chaque position en fonction de cette somme cumulative, ce qui garantit que les particules avec des poids plus élevés ont plus de chances d'être choisies.

4. **Réinitialisation des poids après rééchantillonnage** :
$$
w_k^{(i)} = \frac{1}{N_p}, \quad i = 1, 2, \ldots, N_p
$$
---

## **Résultats et Visualisations**

### **1. Filtre SIS**
![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIS%20K1.png?raw=true)
![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIS%20K10.png?raw=true)

![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIS%20K25.png?raw=true)

#### **Analyse des Résultats du Filtre SIS**
On observe que, dès les premières itérations, les **ellipses de confiance** disparaissent progressivement. Ce phénomène est dû à la **dégénérescence des particules**. 
1. **Concentration des Poids** :
   - À mesure que le filtre progresse, une seule particule finit par accumuler un poids proche de $1$, tandis que toutes les autres ont des poids proches de $0$.
   - Cela signifie que le filtre repose presque entièrement sur une seule particule, réduisant considérablement la diversité des hypothèses d'état.

2. **Effet sur la Covariance** :
   - La covariance $P_k$, qui représente l'incertitude sur l'estimation, devient rapidement nulle.
   - En effet, avec des poids concentrés sur une seule particule, la contribution des autres particules à $P_k$ s'annule, rendant impossible l'estimation de l'incertitude.

3. **Impact Visuel** :
   - Les **ellipses de confiance**, qui dépendent directement de $P_k$, disparaissent car $P_k$ ne peut plus être calculée de manière significative.



### **2. Filtre SIR**

![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIR%20K1.png?raw=true)
![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIR%20K10.png?raw=true)
![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIR%20K25.png?raw=true)
![DFimage description here](https://github.com/Axelado/mes_markdows/blob/images/SIR%20K50.png?raw=true)

#### **Analyse des Résultats du Filtre SIR**

1. **Disparition des Ellipses de Confiance des Repères** :

- Les ellipses de confiance associées aux **repères immobiles (amers)** disparaissent progressivement.

- Cela s'explique par l'absence de bruit dans la dynamique des repères : leur position est considérée comme fixe et parfaitement connue dans le modèle.

- Par conséquent, la variance associée aux repères diminue au fil du temps, jusqu'à s'annuler complètement, reflétant une incertitude nulle sur leur position.

2. **Évolution des Ellipses de Confiance pour le Robot** :

- Contrairement aux repères, l'ellipse de confiance correspondant à la **position du robot** évolue au cours du temps.

- Cette évolution reflète l'incertitude dynamique liée au mouvement du robot et à la qualité des observations.

- Le rééchantillonnage implémenté dans le SIR permet de résoudre efficacement le problème de **dégénérescence des particules** observé dans le SIS. Cela garantit une diversité suffisante des particules, permettant une estimation robuste de la position du robot et de son incertitude.


## **Conclusion**

Ce TP a permis d’explorer en profondeur les **filtres à particules** et leur application à l’estimation d’état pour un système dynamique. En implémentant et comparant les variantes **SIS** (Sequential Importance Sampling) et **SIR** (Sampling Importance Resampling), plusieurs aspects essentiels ont été étudiés, notamment :
- La gestion de l’incertitude,
- La diversité des particules,
- La robustesse des estimations.

Le **filtre SIS** a montré ses limites, notamment en raison de la **dégénérescence des particules**, où une seule particule finit par porter la quasi-totalité du poids. En revanche, le **filtre SIR**, grâce à l’étape de rééchantillonnage, a permis de préserver la diversité des particules et d’améliorer la robustesse des estimations.

En parallèle il serait pertinant de comparer le filtre à particule au filtre de Kalman que nous avons étudier l'an dernier :

1. **Hypothèses et Modèles** :

- Le filtre de Kalman repose sur des hypothèses fortes de **linéarité** et de **distributions gaussiennes**, ce qui le rend optimal dans ces conditions. Cependant, ses performances diminuent rapidement en présence de non-linéarités ou de distributions non gaussiennes.

- Les **filtres à particules**, quant à eux, ne nécessitent pas ces hypothèses, ce qui les rend adaptés à des systèmes plus complexes, mais au prix d’un **coût computationnel plus élevé**.

2. **Représentation de l’Incertitude** :

- Le **filtre de Kalman** représente l’incertitude à travers une **matrice de covariance**, efficace pour des modèles simples et linéaires.

- Les **filtres à particules** utilisent une approximation par des particules, capable de capturer des incertitudes complexes, mais nécessitant un grand nombre de particules pour atteindre une bonne précision.
<!--stackedit_data:
eyJoaXN0b3J5IjpbODM0NTQzMjEzLC0xOTQxNjM1MzksMTUwNT
M2MDgxMywxNTMzODk2MzE4LC02Njk4NTAzMjAsMTg2NzgwNzU4
LDEyMzQ1MzMwMjUsLTUwNjY0NzQ0OCw2MDE0MTU0MDksMjA1OT
M3Mzk3NCwtNTUzNzYxODQwLDE4NDM4MTY2NzgsLTE5ODQ2MjAy
MjMsNzg1NzU2NTE1LDEwNDczMTk5NjVdfQ==
-->