

# Rapport : Filtre à Particules (SIS et SIR)

## **Introduction**
L'objectif de ce TP est d'étudier et de mettre en œuvre des filtres à particules pour l'estimation d'état dans un système dynamique. Deux variantes principales du filtre ont été développées :
1. **Filtre à Particules SIS (Sequential Importance Sampling)** : Utilise un modèle probabiliste pour pondérer les particules en fonction des observations disponibles.
2. **Filtre à Particules SIR (Sampling Importance Resampling)** : Ajoute une étape de rééchantillonnage pour résoudre le problème de dégénérescence des particules.

Les algorithmes ont été appliqués à une simulation de robot mobile évoluant dans un environnement contenant des repères fixes.

---

## **Simulation : Modèle du Système**

### **Données Simulées**
La fonction `dataSimulation` gènère les données nécessaires :
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
- Si $N_{\text{eff}} < N_{\text{seuil}}$, les particules sont redistribuées en fonction de leurs poids.

### **Rééchantillonnage Systématique**
La méthode de rééchantillonnage systématique garantit une redistribution efficace des particules en maintenant leur diversité.
- Positions équidistantes :
$$
\text{positions } = \frac{1}{N_p} \big( \text{rand} + [0, 1, \ldots, N_p-1] \big)
$$

- Redistribuer les particules selon leurs poids $w_k^{(i)}$.

4. **Réinitialisation des poids après rééchantillonnage** :
$$
w_k^{(i)} = \frac{1}{N_p}, \quad i = 1, 2, \ldots, N_p
$$
---

## **Résultats et Visualisations**

### **1. Simulation**
- **Trajectoire réelle** : Tracée en bleu.
- **Positions estimées par le filtre** : Tracées en rouge.
- **Ellipses de confiance** : Illustrent l'incertitude sur $\hat{x}_k$ à chaque instant.

### **2. Comparaison SIS vs SIR**
| Critère                     | SIS                                  | SIR                                      |
|-----------------------------|--------------------------------------|------------------------------------------|
| **Performance**              | Dégradée sur de longues périodes.   | Maintenue grâce au rééchantillonnage.   |
| **Diversité des Particules** | Diminue rapidement.                 | Rétablie périodiquement.                |
| **Complexité**               | Plus simple à implémenter.          | Rééchantillonnage ajoute un coût.       |

### **Visualisation Exemple** :
1. Trajectoires (réelle vs estimée).
2. Evolution des particules.
3. Ellipses de confiance représentant l'incertitude.

---

## **Conclusion**
Le TP a permis de comprendre les concepts fondamentaux des filtres à particules et leurs applications en estimation d'état. Le passage du SIS au SIR montre clairement l'importance du rééchantillonnage pour maintenir la performance sur le long terme.

### **Perspectives**
- Intégrer des modèles plus complexes (non linéaires).
- Comparer avec d'autres méthodes d'estimation, comme le filtre de Kalman étendu (EKF).
- Optimiser les performances en augmentant le nombre de particules ou en utilisant des techniques parallèles.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU1Mzc2MTg0MCwxODQzODE2Njc4LC0xOT
g0NjIwMjIzLDc4NTc1NjUxNSwxMDQ3MzE5OTY1XX0=
-->