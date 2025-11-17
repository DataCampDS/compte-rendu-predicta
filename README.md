# suivi-datacamp
Les points hebdomadaires se feront sur ce README.md


# 17/11/2025

(Flavie) 
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
