# MI2025_T03
 
## Introduction 

L'objectif de ce projet était de détecter automatiquement les volumes problématiques dans une série d'images provenant de séquences d’IRM fonctionnelle. Pour réaliser cela, une série de stratégies a été envisagée (machine learning, deep learning, … ) pour arriver finalement à notre code final. Cette version finale se base sur le calcul du Z-score. En effet, l’idée générale de notre code est de calculer à la fois un Z-score “intra volume”, donc pour des coupes adjacentes d’un même volume et un Z-score “cross volume”, donc pour une même coupe pour des volumes différents. En combinant les 2 Z-scores et en fixant une valeur seuil pour le Z-score, cela permet  de mettre en évidence les potentiels volumes problématiques. 

## Notions théoriques 

Comme dit dans l’introduction, notre méthode se base sur le calcul du Z-score. Nous avons choisi cette mesure statistique car c’est une mesure qui permet d’identifier facilement des valeurs aberrantes ou extrêmes. Cette mesure est donc bien adaptée à notre situation. En effet, le Z-score permet de décrire la position d’une valeur donnée par rapport à la moyenne d’un groupe de valeurs  et cela en termes d’écart-types. En d’autres mots, cette méthode nous fournit une échelle standardisée qui permet d’évaluer à quel point la mesure est éloignée de la moyenne. Mathématiquement, le Z-score se définit comme : 
$Z = \frac{x - \mu}{\sigma}$
## Exploitation du Z-score pour la détection d’artefacts
Comme expliqué plus haut, pour notre approche, nous avons décidé de calculer à la fois un Z-score “intra volume” et un Z-score “cross volume”.

Dans le cas du Z-score intra volume, tout d’abord, on calcule la moyenne d’intensité de chaque coupe d’un même volume. Ensuite, la différence absolue entre les moyennes d’intensité des coupes  adjacentes est calculée. Puis, on applique un z-score sur ces différences pour mesurer à quel point chaque variation entre deux coupes est anormale par rapport aux autres variations dans ce même volume. Dans ce cas, un z-score élevé indique un saut d’intensité particulièrement brutal, ce qui peut refléter la présence d’un artefact.

Dans le cas du Z-score cross volume, on commence par calculer la moyenne d’intensité d’une même coupe à travers tous les volumes. Ensuite, on applique le Z-score sur cette série pour détecter les volumes où la moyenne de la coupe s’écarte anormalement de la tendance générale. Si une coupe dans un certain volume contient du bruit ou un artefact, cela génère un Z-score élevé pour la coupe de ce volume.

## Création d’un seuil adaptatif 

Après le calcul du Z-score, on introduit un seuil de détection pour détecter les données aberrantes, c’est-à-dire celles ayant un Z-score élevé et donc celles qui représentent des potentiels artéfacts. Dans notre cas, nous avons décidé de définir un seuil adaptatif basé sur les volumes propres, c'est-à-dire ceux qui ne présentent pas d’artéfacts. Cela a comme avantage de tenir compte de la variation naturelle du signal dans les données propres sans avoir besoin de fixer un seuil arbitraire ou fixe.

Pour définir ce seuil, on va d’abord calculer les Z-scores pour tous ces volumes propres. Ensuite, on calcule le 99.5ᵉ percentile de ces scores, c’est-à-dire qu’on calcule la valeur au-delà de laquelle seulement 0.5 % des volumes propres dépassent ce score. De cette façon, tout volume dont le score dépasse ce seuil est considéré comme potentiellement anormal. On choisit de calculer le 99.5e percentile car c’est un bon compromis pour capturer les anomalies rares tout en évitant de classer trop de volumes normaux comme suspects. Ce choix reflète une volonté de privilégier une spécificité élevée, c’est-à-dire minimiser les faux positifs, même si cela peut réduire légèrement la sensibilité et donc laisser peut-être passer certains artefacts plus subtils.

## Approche intra et cross volume : 
 
Dans notre approche, nous avons décidé de calculer à la fois le Z-score intra volume et le Z-score cross volume. Le choix du calcul de ces 2 Z-score permet alors une double vérification et de rendre notre modèle plus robuste. En effet, des artéfacts plus subtils pourraient passer inaperçus dans le Z-score intra volume mais pas le Z-score cross volume et inversément. Cela a donc pour effet de réduire les faux négatifs et dans ce cas, l'accent est alors mis sur la sensibilité. 

Le but final est de combiner le Z-score intra-volume et cross-volume. De cette façon, le seuil défini pour le Z-score intra-volume est alors additionné avec le seuil défini pour le Z-score du cross-volume, pour obtenir un seuil global qui correspond à la combinaison des 2 Z-scores. Une idée supplémentaire a été d’effectuer une somme pondérée entre les 2 seuils afin de donner plus d’importance à une certain Z-score par exemple. Cela sera rediscuté dans la section résultat. 

## Codes
La version finale de notre approche correspond au code "``analysis_preliminary.ipynb``" qui reprend l'ensemble des concepts théoriques développés ci-dessus de la manière la plus optimale selon nous. C'est donc sur les résultats obtenus par ce code qui nous baseront notre analyse de ses résultats. 

## Résulats 

Tout d'abord, comme indiqué dans la section "Approche intra et cross volume", une des méthodes possibles consiste à combiner deux scores avec des poids différents. Pour cela, plusieurs simulations ont été réalisées afin de déterminer les poids optimaux à attribuer à chacun des seuils. Nous avons observé, à l’aide des données propres et de la fonction ``run_pipeline2()``, que le score intra-volume reste quasiment constant et n’aide donc pas à discriminer les artefacts. Par conséquent, c’est principalement le score cross-volume qui permet de faire la différence dans la détection, ce qui justifie de lui attribuer un poids plus élevé. Cette pondération repose sur une observation empirique des données, et non sur un fondement théorique strict. 

>Dans la version finale de notre code, nous avons alors considéré un poids de 1 pour le score intra-volume et un poids de 1.5 pour le score cross-volume, le rendant plus important car il rélève plus souvent des défauts.

Sur les graphiques suivants, qui correpond aux trois premiers fichiers des données propres, nous pouvons bien osberver qu'effectivement le score intra-volume reste quasiment constant d'un volume à l'autre comme l'indique la ligne continu jaune sur ces graphiques.
![intra1.png](./Image/intra1.png)
![intra2.png](./Image/intra2.png)
![intra3.png](./Image/intra3.png)

Ensuite, en ce qui cocnerne la détection des artéfacts, pour notre approche où on combine à la fois le Z-score intra-volume et cross-volume de manière pondéré, voici ci-dessous les résultats que nous pouvons observer pour un des fichiers artéfacté : 

![arttt.png](./Image/arttt.png)
Les pics bleus qui dépassent la ligne rouge horizontale (le seuil combiné) correspondent aux volumes dont le Z-score est supérieur au seuil, indiquant ainsi un artéfact potentile. De cette façon, en plus du graphique, notre modèle nous retourne la liste des volumes corrompus par des artéfacts, qui correspond donc aux pics bleus sur le graphique. Pour le graphique ci-dessus, la liste correspondante est la suivante : [42, 97, 148, 195, 196, 328, 377, 378, 521, 655, 734, 735, 780, 781, 782, 783, 784, 785, 788, 794]. On observe plusieurs pics successifs sur les volumes  734, 735, 781-785, 788, 794. Cela peut refléter une perturbation prolongée comme un mouvement brusque ou un pic de bruit par exemple.

Dans le cas des fichiers médiums, voici ci-dessous les résultats que nous pouvons observer pour un de ce type de fichier : 
![medd.png](./Image/medd.png) 
On observe alors la présence de différents pics qui correspondent à la liste de volumes suivants qui est renvoyée par notre modèle : [164, 233, 241, 242, 338, 434, 610, 611, 612].                     Etant donné que les artéfacts présents dans ce type de fichier sont plus subtiles, nous aimerions bien vérifier si notre modèle nous renvoie une liste correcte et cohérente en visualisant par exemple les artéfacts directement sur les volumes pour confirmer leur présence. Le problème est que les artéfacts sont souvent peu visible à l'oeil nu. Pour contrer ce problème, il est alors préférable de prendre l’image de la différence absolue entre une image moyenne des volumes propres et le volume supposé artéfacté. En effet, l'image ci-dessous illustre ce processus pour le volume supposé artéfacté 164 de notre liste. La colonne de gauche illustre une image moyenne des volumes propres (sur les `n=3` volumes suivants et précédents), la colonne centrale illustre le volume 164 (avec une valeur minimale au $10^e$ percentile pour éliminer une partie du bruit et une valeur maximale au $97.5^e$ percentile pour mieux faire resortir l'artifact) et la colonne de droite la différence absolue entre les deux qui met en lumière les différences importantes entre les deux volumes :
![164.png](./Image/164.png) 
On observe un certain nombre de différences mise en lumière par les points plus clairs sur les images de la colonne de droite et nous permet de confirmer que le volume 164 est artéfacté de manière significative.  

Une autre manière de visualiser ces défauts est de prendre le ratio entre les bonnes coupes moyennes et le volume artifacté comme sur la figure ci dessous, où la valeur d'un élément a été multiplié par $3.10^9$.

![164.png](./Image/164_ratio.png) 


## Recherches littératures 
Lorsqu'on se penche sur la littétuature disponible sur ce domaine, on se rend compte que ce projet ne fait que "gratter la surface" du sujet. On peut facilement trouver une grande quantité d'améliorations et de concepts à intégrer à notre méthode. Voici cependant 3 pistes d'amélioration qui peuvent être directement envisagées afin d’optimiser la robustesse et la sensibilité de notre méthode de détection d’artefacts.

1. Approche plus rigoureuse pour le calcul des seuils : 

Actuellement, le seuil de détection des volumes problématiques est défini de manière empirique comme le 99.5ᵉ percentile des Z-scores issus des volumes jugés propres. Bien que cette approche soit simple et efficace, elle pourrait être améliorée par l'utilisation de méthodes statistiques plus robustes. Par exemple, l’article de Saluja et al. (2025), "Robust outlier detection for heterogeneous distributions applicable to censoring in functional MRI" propose d'utiliser une transformation non paramétrique (sinh-arcsinh) pour ajuster les distributions asymétriques et mieux détecter les valeurs extrêmes. Intégrer ce type d’approche renforcerait la fiabilité du seuil, en particulier dans des jeux de données contenant des outliers subtils.

2. Pondération optimisée par apprentissage supervisé : 

Dans notre méthode actuelle, les Z-scores intra-volume et cross-volume sont combinés à l’aide de pondérations empiriques (respectivement 1 et 1.5). Bien que ce choix soit basé sur une observation pertinente du comportement des scores dans nos données, il reste arbitraire et potentiellement non généralisable à d’autres ensembles de données.
Une amélioration serait de déterminer ces pondérations de manière automatique, en s’inspirant de l’approche proposée par Mejia et al. (2015) dans leur article "PCA leverage: outlier detection for high-dimensional functional magnetic resonance imaging data". Les auteurs y démontrent qu’il est plus efficace de combiner plusieurs indicateurs statistiques en pondérant leur contribution via un modèle supervisé, tel qu’une régression logistique ou une analyse discriminante. Ces modèles apprennent, à partir de données annotées (volumes avec ou sans artefact), à attribuer aux indicateurs les poids qui maximisent leur pouvoir prédictif.

3. Ajout d’une étape de décomposition en composantes indépendantes (ICA) : 

Certaines approches de détection d’artefacts telles que ICA-AROMA (Pruim et al., 2015) exploitent une décomposition ICA pour isoler les composantes associées au bruit (mouvement, respiration, etc.). Une autre amélioration potentielle serait donc d’extraire des composantes indépendantes des volumes IRMf et d’appliquer notre système de scoring (Z-score ou autre) à ces composantes. Cette approche pourrait améliorer la détection des artefacts subtils en séparant mieux les sources de signal pertinentes des sources de bruit.


## Conclusion 
Pour conclure, vu les résultats obtenus et des principes théoriques pris en compte dans notre approche, notre implémentation constitue une approche pertinente pour la détection d'artéfacts. Elle présente l’avantage d’être flexible, grâce à la possibilité d’ajuster les poids attribués aux scores intra et inter-volume. Cela permet d’adapter la sensibilité de la détection en fonction des besoins spécifiques : par exemple, privilégier une détection plus stricte dans des contextes cliniques, ou plus souple dans d’autres cas.

Néanmoins, plusieurs pistes d’amélioration restent possibles. Notamment, des ajustements dans la manière dont les seuils sont définis et les poids qui leur sont associés. Une approche plus théorique de ce dernier point pourrait renforcer notre méthode.





