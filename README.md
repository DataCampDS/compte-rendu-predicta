# suivi-datacamp
Les points hebdomadaires se feront sur ce README.md


# 17/11/2025

J'ai testé plusieurs méthodes d'amélioration de l'algorithme.
J'ai améliorer la normalisation et j'ai essayé de réduire le bruit. Cela m'a donné une meilleure accuracy.
J'ai essayé d'utiliser une ACP mais le résultat n'était pas bon. Nous allons alors retravailler cette ACP pour trouver un nombre intéressant de composantes principales à utiliser.
J'ai également essayé un algorithme XGBoost qui a donné de meilleurs résultats que le Random Forest. 
Cependant, nous n'avons pas pris le temps d'analyser les données pour être sûr de ce que nous faisons. Donc aujourd'hui nous retournons à la base et allons prendre le temps de bien regarder les données et travailler le pre-processing.

L'objectif du projet est de classer des cellules à partir de données RNA-seq. L'objectif serait de prédire automatiquement le type d’une cellule à partir de son profil de gènes.

Il y a 4 types de cellules : 
- Cancer_cells
- NK_cells
- T_cells_CD4+
- T_cells_CD8+

Chaque point correspond à une cellule, représentée par plusieurs milliers de gènes (chaque colonne est un gène) et un label (dans l'entraînement).
Le jeu de données est de 1000 cellules pour l'entraînement, 500 cellules pour le test, donc 1500 en tout.

On commence à analyser le dataset d'entraînement en regardant ses dimensions qui sont (1000, 13551). Les colonnes correspondent au type de cellule et à la présence des gènes dans la cellules, l'expression des gènes est alors au nombre de 13550, ce qui est très élevé. Nous avons alors trié les données géniques en supprimant les gènes qui ne sont pas pertinants pour différencier les cellules, comme les gènes qui sont toujours constants pour chaque type de cellules.


# 1/12/2025

On a travaillé sur le pre-processsing des données. Les colonnes ayant une valeur identique (ou presque) pour toutes les observations, c'est-à-dire les colonnes dont la variance est inferieur à 0.05, ont été supprimées. Cela a réduit le dataset d'un peu plus de 5000 colonnes. Ensuite, nous avons étudier la corrélation. Certaines colonnes étant fortement corrélées avec d'autres ont été supprimés. 

On a tracé la taux de variance expliqué en fonction du nombre de composantes. Grâce à cela, on a pu contaster qu'avec 25 composantes, plus de 90% de variance était expliquée. On a alors réalisé une ACP avec 25 composantes principales.

Pour les pipelines on a testé :  ACP + régression logistique, ACP + classifieur de Bayes, ACP + XGBOOST
et pour l'instant c'est le dernier qui est le plus précis.


# 22/12/2025

On a testé d'équilibrer strictement les 4 classes pour voir si celle des NK_cells, qui est d'origine très sous représentée, pourrait devenir mieux prédite. Et en effet, en en limitant les données d'entrainement et en affichant la matrice de confusion on remarque que les vrais NK_cells sont 2 fois plus prédites. Avant elles étaient davantage confondues avec les T_cells_CD8+ que prédites positivement ce qui en faisait le problème majeur de la classification. Enfin, bien que l'équilibre stricte des classes n'a pas fait changer la prédiction des Cancer_cells, il a empiré la confusion des T_cells CD4+ et CD8+, qui constitue le deuxième problème de la classification.


# 27/12/2025

On a essayé de gérer la confusion des T_cells avec une méthode hiérarchique. D'abord on a regroupé les T_cells et on a entrainé un classifieur sur les Cancer_cells, les NK_cells, et les T_cells puis ensuite un deuxième classieur pour différencier les T_cells CD4+ et CD8+, avant de recombiner toutes les prédictions intermédiaires en sortie. En affichant la matrice de confusion on voit que les T_cells sont mieux prédites ainsi. Cependant, c'était d'abord léger, car sans la balance des classes, qui devait etre repensée avec ces données séparées, les NK_cells étaient toujours prédites comme des T_cells_CD8+. Ainsi, on a fait une balance stricte des classes pour le classifieur principal (sur les 3 classes), et on a fait l'entrainement du deuxieme classifieur sur toutes les T_cells. C'est donc bien la combinaison des deux techniques qui est satisfaisante car les problèmes sont liés par le fait que les NK_cells (sous représentés) étaient prédites comme des T_cells (dur à différencier).

# 29/12/2025

Finalement, on a décidé de re tester la régression logistique. La régression logistique gère toute seule le déséquilibre des classes grâce au paramètre class_weight='balanced', cela permet de conserver l’ensemble des données d’entraînement. Le gradient Boosting peut peut-être mener à du sur-apprentissage également, et c'est pour cela que l'on a voulu retester la régression logistique. Et en effet, la régression logistique présente un faible écart entre les performances d’entraînements et de tests et avec des performances élevées en termes de balanced accuracy.
De plus, les NK_cells semblent être mieux prédites avec la régression logistique mais les CD8 semblent par contre être moins bien prédites. 
Nous avons aussi remplacé la sélection des gènes par variance brute par une sélection utilisant le coefficient de variation, afin de privilégier des gènes réellement informatifs et d’éviter ceux qui sont très exprimés mais peu discriminants. Les résultats obtenus montrent que cette approche est au moins aussi performante pour la régression logistique mais pas pour le gradient boosting.
L'accuracy obtenue est très bonne (0.85) mais inférieure à celle obtenue avec le gradient boosting (0.86). Mais si cette méthode peut nous éviter du sur-apprentissage, cela peut être une bonne option.

En plus, nous avons regardé l'accuracy pour plusiseurs valeurs de nombre de composants pour la PCA. Trop peu de composantes perdent de l’information, trop de composantes ajoutent du bruit. Entre 5 et 20 composantes peut être considéré comme trop faible, entre 30 et 40 a l'air bien et au dessus de 50 c'est trop.
