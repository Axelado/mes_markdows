

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
eyJoaXN0b3J5IjpbLTE5NDE2MzUzOSwxNTA1MzYwODEzLDE1Mz
M4OTYzMTgsLTY2OTg1MDMyMCwxODY3ODA3NTgsMTIzNDUzMzAy
NSwtNTA2NjQ3NDQ4LDYwMTQxNTQwOSwyMDU5MzczOTc0LC01NT
M3NjE4NDAsMTg0MzgxNjY3OCwtMTk4NDYyMDIyMyw3ODU3NTY1
MTUsMTA0NzMxOTk2NV19
-->