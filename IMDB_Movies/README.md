#  Prédiction du succès financier des films IMDB

## 1. Introduction

Ce projet de data science vise à prédire le **revenu brut (Gross)** des films à partir de leurs caractéristiques, en se basant sur le dataset **IMDB Movies Dataset** disponible sur Kaggle.

L’objectif principal est de construire un modèle capable d'estimer le succès commercial d’un film **avant sa sortie**, en exploitant des informations telles que :
- Le nom des **acteurs** et du **réalisateur**
- Le **genre** du film
- Sa **durée**
- Son **année de sortie**
- Son **certificat de classification**
- Le **nombre de votes IMDB**, etc.

###  Contexte
Le dataset contient 1 000 films parmi les mieux notés sur IMDB, avec des variables comme :
- `Series_Title`, `Director`, `Stars`, `Genre`, `Runtime`, `Certificate`
- `IMDB_Rating`, `Meta_score`, `No_of_votes`, `Gross` (la cible à prédire)

Ce projet a été réalisé dans une logique **d’apprentissage par la pratique**, en appliquant toutes les étapes clés d’un cycle de data science :
1. Exploration et nettoyage des données
2. Préparation des variables
3. Modélisation et évaluation
4. Interprétation des résultats

Le but final : **comprendre les facteurs qui influencent réellement le revenu d’un film** et construire un modèle robuste pour en faire la prédiction.

## 2.  Exploration des données (EDA)

Avant toute modélisation, une étape d'exploration et de compréhension des données a été réalisée.

###  Description du dataset

Le dataset contient **1 000 films** avec les variables suivantes :

| Variable         | Description |
|------------------|-------------|
| `Series_Title`   | Titre du film |
| `Released_Year`  | Année de sortie |
| `Certificate`    | Classification du film (ex : PG-13, R, U) |
| `Runtime`        | Durée du film (ex : "142 min") |
| `Genre`          | Genres du film (ex : "Action, Drama, Sci-Fi") |
| `IMDB_Rating`    | Note moyenne sur IMDB |
| `Meta_score`     | Score Metacritic |
| `Director`       | Réalisateur |
| `Star1` à `Star4`| Acteurs principaux |
| `No_of_Votes`    | Nombre de votes reçus sur IMDB |
| `Gross`          | Revenu brut généré par le film (en $) |

###  Données manquantes

- `Certificate` : 101 valeurs manquantes  
- `Meta_score` : 157 valeurs manquantes  
- `Gross` : 169 valeurs manquantes (notre **variable cible**)

Des décisions ont été prises :
- Remplacer les valeurs manquantes de `Meta_score` par sa **moyenne**
- Remplacer `Certificate` par sa **valeur la plus fréquente**
- Supprimer les lignes où `Gross` est manquant (car non prédictible)

### 📊 Visualisations exploratoires

Plusieurs visualisations ont été réalisées pour mieux comprendre les données :

- **Distribution de `Gross`** : très asymétrique, avec quelques blockbusters extrêmes
- **Corrélation entre variables** : `No_of_Votes` est la plus corrélée à `Gross` (0.57)
- **Genres les plus rentables** : `Adventure`, `Sci-Fi`, `Action` dominent
- **Réalisateurs les plus bankables** : Anthony Russo, James Cameron, J.J. Abrams...
- **Tendance temporelle** : les films récents ont tendance à rapporter davantage

---

## 3.  Préparation des données

###  Traitement des variables catégoriques

Les colonnes comme `Genre`, `Certificate`, `Director`, `Star1–4` ont été encodées :

- `Genre`, `Certificate` : **One-Hot Encoding**
- `Director`, `Stars` : **Target Encoding** basé sur le revenu moyen par personne
  - Les 4 colonnes `Star1–4` ont été combinées et remplacées par un score moyen `Stars_encoded`

###  Transformation de la variable cible

Le revenu (`Gross`) ayant une distribution très asymétrique, on a appliqué une **transformation logarithmique** :
```python
df['log_Gross'] = np.log(df['Gross'] + 1)
```
Cela rend la distribution plus normale et améliore l’apprentissage des modèles de régression.

##  Normalisation des variables numériques

- Les variables numériques (No_of_Votes, Runtime, Stars_encoded, etc.) ont été standardisées avec StandardScaler
- La variable cible Gross n’a pas été normalisée, mais seulement transformée en log.

## Split du dataset

Le dataset a été séparé en 3 parties :

- 70% Entraînement (X_train, y_train)
- 15% Validation (X_val, y_val)
- 15% Test final (X_test, y_test)

## 4.  Modélisation

L’objectif est de prédire le revenu brut (`Gross`) d’un film à partir de ses caractéristiques.  
Plusieurs modèles de régression ont été testés, en commençant par un modèle simple, puis en allant vers des modèles plus puissants.

---

###  Régression linéaire (baseline)

Un premier modèle **de régression linéaire** a été entraîné sur la variable transformée `log(Gross)`.

#### Résultat :
Les performances obtenues étaient **extrêmement mauvaises** :

- **MAE :** valeurs astronomiques
- **R² :** fortement négatif (pire qu’un modèle qui prédit la moyenne)

 **Interprétation** : la relation entre les variables et le revenu n’est **pas linéaire**, donc ce modèle est **inadapté**.

---

###  Random Forest Regressor

Un modèle **Random Forest**, basé sur un ensemble d’arbres de décision, a ensuite été utilisé.

####  Avantages :
- Capte les **relations non linéaires**
- Gère bien les **interactions entre variables**
- Résistant aux outliers
- Aucun besoin de normalisation des variables

####  Résultats (validation) :
- **MAE :** 14,4 M $
- **RMSE :** 34,8 M $
- **R² :** 0.8700

Ces résultats montrent que le modèle est **très performant**, expliquant plus de 87% de la variance des revenus sur des films jamais vus à l'entraînement.

---

### ⚙️ Optimisation avec GridSearchCV

Une recherche d'hyperparamètres (`n_estimators`, `max_depth`, etc.) a été menée pour affiner le modèle.

Résultat : les performances sont restées **quasiment identiques**, ce qui confirme que le modèle initial était déjà très bien ajusté.

---

###  Test d’un modèle XGBoost

Un modèle **XGBoost** (Extreme Gradient Boosting) a également été testé.  
Ce modèle est très performant sur de nombreux problèmes, mais dans ce cas précis, il a légèrement **sous-performé** comparé à Random Forest.

####  Résultats XGBoost (validation) :
- **MAE :** 15,5 M $
- **RMSE :** 38,2 M $
- **R² :** 0.8438

> Conclusion : le **Random Forest** reste le **modèle optimal** pour ce projet.

---

###  Évaluation finale sur le jeu de test

Le modèle Random Forest a ensuite été testé sur les **données totalement inédites** (`X_test`, `y_test`), avec la transformation inverse du logarithme appliquée aux prédictions.

####  Résultats sur test final :
- **MAE :** 18,1 M $
- **RMSE :** 40,0 M $
- **R² :** 0.8642 

 Cela confirme que le modèle est **robuste, fiable et généralisable**.


## 5.  Évaluation des performances et importance des variables

###  Métriques utilisées

Trois métriques ont été utilisées pour évaluer la performance des modèles :

| Métrique | Description |
|---------|-------------|
| **MAE (Mean Absolute Error)** | Erreur moyenne absolue : donne une idée directe de l'écart moyen entre la prédiction et la réalité |
| **RMSE (Root Mean Squared Error)** | Pénalise davantage les grosses erreurs que le MAE |
| **R² (coefficient de détermination)** | Mesure la proportion de la variance expliquée par le modèle (entre 0 et 1 idéalement) |

---

###  Résumé des performances finales du modèle Random Forest

| Jeu de données | MAE (en $) | RMSE (en $) | R² |
|----------------|-------------|-------------|-----|
| **Validation** | 14,455,614 | 34,857,261 | 0.8700 |
| **Test final** | 18,135,556 | 40,056,388 | 0.8642  |

>  Le modèle est capable de prédire les revenus d’un film avec **moins de 20 millions d’erreur moyenne**, ce qui est excellent compte tenu de la variabilité des films (certains rapportant quelques millions, d'autres plusieurs centaines).

---

###  Importance des variables

Le modèle Random Forest permet d’analyser **l’importance relative de chaque variable** dans la prédiction du revenu.

####  Top 10 des variables les plus importantes :

1. **Stars_encoded** → Niveau "bankable" des acteurs
2. **Director_encoded** → Réputation du réalisateur
3. **Released_Year** → Tendance des films récents à rapporter plus
4. **Genre_Crime, Thriller**
5. **No_of_Votes** → Popularité sur IMDB
6. **Runtime** → Longueur du film
7. **Genre_Drama, History**
8. **Genre_Drama**
9. **Genre_Comedy, Drama, War**
10. **Certificate_Passed**

####  Interprétation :

- Les variables **Stars_encoded** et **Director_encoded** dominent clairement, montrant que **le casting et le réalisateur sont les meilleurs prédicteurs du succès**.
- Le **genre**, le **nombre de votes** et la **durée** du film ont une influence plus marginale.
- Le **certificat de classification** (ex : PG-13, R...) semble peu déterminant.

 Ces résultats sont cohérents avec l’intuition métier : les **grands noms attirent l’audience**, et donc les revenus.

## 6.  Conclusion et interprétation métier

###  Objectif atteint

L’objectif de ce projet était de **prédire le revenu brut (`Gross`) d’un film** à partir de ses caractéristiques disponibles **avant sa sortie**.  
Grâce à un traitement rigoureux des données, une modélisation adaptée et des tests approfondis, un **modèle performant et généralisable** a été construit.

###  Résumé des étapes clés

- **Exploration du dataset IMDB** pour comprendre les tendances générales
- **Nettoyage et transformation** des données, y compris la transformation log de la cible
- **Encodage intelligent** des variables catégoriques comme `Stars`, `Director`, `Genre`
- **Test de plusieurs modèles**, dont Linear Regression (baseline), Random Forest (final), et XGBoost (comparatif)
- **Optimisation des hyperparamètres** avec GridSearch
- **Évaluation finale sur données inédites** avec des métriques solides (R² ≈ 0.86)

---

###  Ce que nous apprend le modèle

L’analyse des variables les plus importantes révèle des **enseignements très clairs** :

- Les **acteurs principaux** (`Stars_encoded`) et le **réalisateur** (`Director_encoded`) sont **les deux facteurs les plus prédictifs** du succès commercial d’un film.
- Les variables comme le **genre**, le **nombre de votes IMDB**, ou la **durée du film** ont un **impact plus limité**.
- Le **certificat de classification** (PG-13, R...) a une influence marginale.

 En d'autres termes, le **casting** et le **talent derrière la caméra** sont les éléments déterminants pour maximiser les revenus d’un film.


### Perspectives et limites

#### Points forts :
- Modèle performant, explicable et reproductible
- Pipeline complet de bout en bout
- Résultats interprétables grâce à l’analyse des importances

####  Limites :
- Données limitées à 1 000 films → pas représentatif de toute l'industrie
- Certaines variables comme `Overview` ou les budgets n’étaient pas exploitées
- Le modèle ne prend pas en compte les effets de campagne marketing, date de sortie, etc.

---

###  Améliorations possibles

- Intégrer des techniques NLP sur `Overview` (résumé du film)
- Utiliser des données supplémentaires (budget, studio, date exacte de sortie)
- Tester d'autres algorithmes de boosting (CatBoost, LightGBM)
- Déployer le modèle dans une application interactive (API ou dashboard)

---


Ce projet montre qu’il est possible de **prédire avec une bonne précision le succès financier d’un film** à partir de variables simples, **en particulier liées au casting et au réalisateur**.  
C’est une démonstration concrète de la **valeur prédictive des données** dans un contexte artistique et économique comme le cinéma.

> Merci de votre lecture   













