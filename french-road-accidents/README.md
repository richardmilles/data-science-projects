#  Analyse et Modélisation de la Gravité des Accidents Routiers en France (2005–2021)

## 1. Introduction

Les accidents de la route constituent un enjeu majeur de santé publique, de sécurité et d’économie. Chaque année, des milliers de personnes sont tuées ou gravement blessées sur les routes françaises. Comprendre les facteurs qui influencent la gravité d’un accident est crucial pour améliorer la prévention, orienter les politiques publiques, et optimiser la tarification ou la gestion des sinistres pour les assureurs.

Ce projet s’appuie sur un ensemble de données ouvertes couvrant les accidents corporels de la circulation en France entre 2005 et 2021. L’objectif est de développer un modèle prédictif capable d’estimer la gravité d’un accident en fonction des conditions connues : caractéristiques de la route, du véhicule, du conducteur, des circonstances, etc.


## 2. Compréhension du problème et des données

###  Objectif métier

L’objectif est de prédire la **gravité d’un accident** parmi trois classes :
- **1** : Indemne
- **2** : Tué
- **3** : Blessé hospitalisé

Cette tâche est formulée comme un **problème de classification multi-classes**. Un bon modèle pourrait :
- Aider à identifier les situations à risque élevé
- Appuyer la mise en place de politiques de sécurité ciblées
- Fournir des alertes automatiques dans des outils embarqués
- Mieux comprendre les profils d'accidents graves


### 🗃 Source des données

Les données proviennent du portail [Open Data](https://www.data.gouv.fr/fr/datasets/bases-de-donnees-annuelles-des-accidents-corporels-de-la-circulation-routiere-annees-de-2005-a-2023/). Elles regroupent des informations détaillées sur les accidents, véhicules et usagers impliqués.

Les jeux de données ont été :
- Nettoyés
- Fusionnés
- Encodés
- Et enrichis avec des variables cycliques (`TimeOfDay`, `DayOfWeek`, `Month`) via leurs composantes `sin` et `cos`.


###  Prétraitement spécifique

La variable cible initiale `grav` contenait 4 classes :
- **1 : Indemne**
- **2 : Tué**
- **3 : Blessé hospitalisé**
- **4 : Blessé léger**

La classe 4 (blessé léger) représentant seulement **2.7 % des cas**, elle a été **supprimée** pour :
- Réduire le déséquilibre
- Stabiliser l’apprentissage
- Simplifier l’analyse

La classification cible finale ne comporte donc que **3 classes significatives**.


### Variables disponibles

Le dataset final comprend des variables de type :
- **Numérique normalisée** (`age`, `nbv`)
- **Catégorielle encodée** (`catu`, `catv`, `manv`, `prof`, etc.)
- **Cyclique** (sinus et cosinus de l’heure, du jour de la semaine, du mois)
- **Variables de contexte** (luminosité, agglomération, obstacle, etc.)

Le tout sur **2.3 millions de lignes** couvrant la France entière entre 2005 et 2021.


## 3. Analyse exploratoire des données (EDA)

### Objectif

L’objectif de cette analyse exploratoire est d’étudier la distribution des variables disponibles, de comprendre la structure des données, et d’identifier les relations potentielles entre les variables explicatives et la variable cible `grav`, représentant la gravité des accidents. Les données ayant déjà été nettoyées, la classe 4 (blessé léger) a été supprimée en raison de sa très faible représentativité (2.7 %), afin d’améliorer la stabilité et l’efficacité des futurs modèles de classification.


### Distribution de la variable cible

Après traitement, la variable `grav` contient trois classes :

- **1 : Indemne** — 42.05 %
- **2 : Tué** — 20.51 %
- **3 : Blessé hospitalisé** — 37.44 %

Cette distribution relativement équilibrée permet de travailler dans un contexte de classification multi-classes sans déséquilibre majeur. La suppression de la classe 4 a permis de réduire un déséquilibre initial important.


### Analyse univariée et bivariée

#### Âge (`age`)

L’âge est normalisé dans le jeu de données. L’analyse par boîte à moustaches (boxplot) indique que les personnes tuées ou blessées hospitalisées sont, en moyenne, légèrement plus âgées que les indemnes. Cette variable semble donc avoir un lien avec la gravité de l'accident.

#### Catégorie d’usager (`catu`)

La majorité des usagers impliqués appartiennent à la catégorie 1 (probablement les conducteurs). Cependant, les autres catégories (passagers, piétons, cyclistes, etc.) sont plus représentées proportionnellement dans les classes graves, suggérant une plus grande vulnérabilité.

#### Luminosité (`lum`)

Bien que la plupart des accidents surviennent en plein jour, une part importante des accidents graves a lieu dans des conditions de faible visibilité (nuit sans éclairage, nuit avec éclairage insuffisant), ce qui indique l’influence de la luminosité sur la gravité.

#### Agglomération (`agg`)

Les accidents sont plus nombreux en agglomération. Cependant, la gravité des accidents est significativement plus élevée hors agglomération, probablement à cause de vitesses plus élevées et d’un environnement routier moins contrôlé.

#### Mois, jour de la semaine, heure de la journée

- **Mois** : Une légère hausse des accidents graves est observée durant les mois d’été et d’automne, en lien avec l’augmentation du trafic pendant les vacances.
- **Jour de la semaine** : Les accidents sont plus fréquents en semaine, avec une concentration sur les jours ouvrés (lundi à vendredi).
- **Heure** : Deux pics journaliers apparaissent : en matinée (7h–9h) et en fin de journée (16h–20h), correspondant aux heures de pointe. Ces pics sont aussi présents pour les accidents graves.

---

### Analyse des variables routières et contextuelles

#### Type de choc (`choc`)

Les chocs frontaux ou latéraux sont plus souvent associés aux accidents graves, tandis que d'autres types (arrière, stationnement, etc.) apparaissent davantage dans les cas sans blessure.

#### Manœuvre (`manv`)

Une manœuvre prédominante (code 9) apparaît massivement dans toutes les classes, mais certaines manœuvres spécifiques (ex. dépassement, demi-tour) sont sur-représentées dans les accidents mortels, suggérant l’importance du comportement du conducteur.

#### Catégorie de véhicule (`catv`)

Une catégorie de véhicule est ultra-dominante (probablement les voitures particulières). D'autres catégories comme les deux-roues motorisés ou les poids lourds sont, proportionnellement, plus souvent impliquées dans des accidents graves.

#### Profil et plan de la route (`prof`, `plan`)

Les routes plates et droites sont les plus fréquentes. Cependant, les routes présentant des pentes ou des virages sont davantage associées à des accidents graves, indiquant un impact du relief et de la géométrie routière sur la gravité.

#### Catégorie de route (`catr`)

Certaines catégories de routes (grandes routes hors agglomération, autoroutes, etc.) sont plus susceptibles de causer des accidents graves, confirmant les observations faites sur l’agglomération.

#### Obstacle (`obs`) et situation particulière (`situ`)

La présence d’obstacles fixes (arbres, murs, etc.) ou de situations spéciales (travaux, stationnement, circulation inhabituelle) sont des facteurs aggravants dans la survenue d’un accident grave.

Cette analyse exploratoire a permis d’identifier plusieurs variables fortement corrélées à la gravité des accidents, notamment :

- Le profil de l’usager (âge, sexe, catégorie)
- Les conditions environnementales (luminosité, agglomération, heure)
- Le type de véhicule et de manœuvre
- Les caractéristiques de la route (profil, plan, catégorie)
- Les circonstances spécifiques (type de choc, obstacles)

Ces variables seront privilégiées dans la phase de modélisation. Par ailleurs, l’analyse temporelle a mis en évidence des périodes à risque (heures de pointe, périodes estivales), ce qui peut orienter des actions de prévention ciblées.  
Cette EDA constitue une base solide pour formuler des hypothèses métier et développer un modèle prédictif fiable et interprétable.

## 4. Préparation des données pour la modélisation

### Séparation du jeu de données

Le dataset final contient plus de 2,3 millions d’observations. Afin de garantir une évaluation robuste du modèle, les données ont été divisées en trois sous-ensembles :

- **Jeu d’entraînement (train)** : 70 %
- **Jeu de validation (val)** : 15 %
- **Jeu de test (test)** : 15 %

La séparation a été réalisée de manière stratifiée afin de préserver la distribution des classes de la variable cible `grav`. Cette approche permet d'entraîner le modèle sur le jeu `train`, d'ajuster les hyperparamètres sur `val`, puis d'évaluer la performance finale sur `test`, qui reste totalement indépendant.

### Traitement de la variable cible

La variable `grav` a été recodée en trois classes pour la modélisation :

- 0 : Indemne
- 1 : Tué
- 2 : Blessé hospitalisé

Cette transformation permet une compatibilité avec les algorithmes de classification, comme XGBoost, qui attend des classes entières commençant à 0.

### Encodage et normalisation

L’ensemble des variables catégorielles avait déjà été encodé. Les variables numériques (`age`, `nbv`) avaient été normalisées. De plus, les variables cycliques (mois, heure, jour de la semaine) avaient été transformées en `sin` et `cos`, pour respecter leur nature périodique.

Aucun traitement supplémentaire n’a été nécessaire à cette étape.

### Réduction de la variable cible

Comme mentionné précédemment, la classe 4 (blessé léger) a été supprimée pour des raisons de représentativité (moins de 3 % des cas). Le modèle a donc été entraîné uniquement sur les trois classes restantes, afin d'assurer une meilleure stabilité d'apprentissage et une meilleure précision sur les cas graves.

### Rééquilibrage des classes

Même après suppression de la classe 4, la classe "Tué" restait sous-représentée. Pour compenser ce déséquilibre, la méthode SMOTE (Synthetic Minority Over-sampling Technique) a été utilisée. Cette technique permet de créer des exemples synthétiques pour la classe minoritaire à partir des voisins les plus proches, améliorant ainsi l’apprentissage sans perte d’information.

SMOTE a été appliqué uniquement sur le jeu d'entraînement, dans le cadre d’un pipeline de modélisation.

### Réduction des variables

Après une première phase de modélisation avec toutes les variables, l’importance des features a été évaluée à l’aide de XGBoost. Les 10 variables les plus importantes ont été sélectionnées pour la version finale du modèle :

- `agg` : type de zone (agglomération ou non)
- `catv` : catégorie du véhicule
- `catr` : catégorie de la route
- `obs` : type d’obstacle
- `catu` : type d’usager
- `plan` : plan de la route
- `sexe` : sexe de l’usager
- `nbv` : nombre de voies
- `circ` : régime de circulation
- `choc` : type de choc

Cette sélection permet de simplifier le modèle, de réduire les temps d’entraînement et de maintenir des performances comparables, tout en améliorant la sensibilité sur les cas critiques.

## 5. Modélisation

### Choix du modèle

Le modèle principal utilisé dans ce projet est **XGBoost (Extreme Gradient Boosting)**, un algorithme d’ensemble basé sur des arbres de décision. Ce choix repose sur plusieurs critères :

- Excellentes performances sur des données tabulaires
- Gestion native du déséquilibre via des pondérations et la régularisation
- Possibilité d’intégrer un pipeline complet (prétraitement + modèle)
- Interprétabilité via les scores d’importance des variables
- Support du multi-class nativement (`objective = "multi:softmax"`)

D'autres modèles (comme Random Forest ou Logistic Regression) n’ont pas été testés, car l’objectif du projet n’était pas la comparaison de modèles, mais l’optimisation complète d’un modèle unique.

### Mise en place du pipeline de modélisation

Un **pipeline complet** a été construit à l’aide de `imblearn.pipeline.Pipeline`, intégrant :

- **SMOTE** (pour le rééquilibrage automatique des classes)
- **XGBoost** (entraînement du modèle)

Ce pipeline permet de garantir que le rééchantillonnage (SMOTE) ne se fait **que sur le jeu d’entraînement**, évitant ainsi les fuites de données entre les phases d’apprentissage et d’évaluation.

### Optimisation des hyperparamètres

L’optimisation des hyperparamètres a été réalisée avec `RandomizedSearchCV`, en effectuant 20 combinaisons aléatoires de paramètres sur un ensemble de validation croisée à 3 folds. Les paramètres explorés incluaient :

- `n_estimators` : nombre d’arbres
- `max_depth` : profondeur maximale des arbres
- `learning_rate` : taux d’apprentissage
- `subsample` : fraction d’échantillons utilisés pour chaque arbre
- `colsample_bytree` : fraction des variables utilisées à chaque split
- `gamma` : régularisation (complexité minimale pour effectuer un split)

L'entraînement a été effectué avec **early stopping**, permettant d’arrêter automatiquement l’optimisation si les performances sur l’ensemble de validation ne s’amélioraient plus après 10 itérations.

### Métrique d’évaluation personnalisée

L’objectif métier étant de **mieux détecter les accidents mortels**, la métrique utilisée pour guider l’optimisation n’était pas l’accuracy globale, mais le **rappel (recall) sur la classe “Tué”** (encodée 1).

Une fonction personnalisée a été définie pour extraire le rappel de cette classe, et utilisée comme `scoring` dans le `RandomizedSearchCV`.

Cela a permis d’orienter l’optimisation vers une meilleure sensibilité aux cas les plus critiques, même au prix d’un léger sacrifice sur les autres classes.

### Deux approches testées

Deux versions du modèle ont été testées :

- **Modèle complet** : entraîné avec toutes les variables du dataset
- **Modèle réduit** : entraîné uniquement avec les 10 variables les plus importantes

La comparaison des deux modèles a permis d’évaluer l’impact de la réduction de dimension sur les performances du modèle, notamment sur la détection des cas graves.
## 5. Modélisation (suite)

### Entraînement du modèle

L’entraînement a été réalisé sur le jeu `X_train` (70 % des données), avec validation croisée 3-fold sur ce même sous-ensemble. Le jeu de validation (`X_val`, 15 %) a été utilisé pour l’évaluation intermédiaire, et le jeu de test (`X_test`, 15 %) a été conservé jusqu’à l’évaluation finale.

Le modèle final a été entraîné à la fois sur :

- Un **jeu complet avec toutes les variables disponibles**
- Un **jeu réduit** contenant uniquement les **10 variables les plus importantes**, identifiées via l'attribut `feature_importances_` de XGBoost

Ce second modèle visait à tester si la réduction de complexité permettait de meilleures performances sur les cas graves.


## 6. Résultats

### Résultats du modèle complet

Sur le jeu de validation, le modèle complet a obtenu :

| Classe           | Precision | Recall | F1-score |
|------------------|-----------|--------|----------|
| Indemne          | 0.70      | 0.80   | 0.75     |
| Tué              | 0.47      | 0.49   | 0.48     |
| Blessé hospitalisé | 0.63    | 0.52   | 0.57     |

- Accuracy globale : **63 %**
- Rappel pour la classe "Tué" : **49 %**

Ce modèle détectait correctement environ **1 accident mortel sur 2**, ce qui représentait déjà une nette amélioration par rapport à la baseline initiale (42 %).

### Résultats du modèle réduit (10 variables)

Avec seulement les 10 variables les plus importantes, le modèle a obtenu sur le jeu de validation :

| Classe           | Precision | Recall | F1-score |
|------------------|-----------|--------|----------|
| Indemne          | 0.75      | 0.74   | 0.75     |
| Tué              | 0.46      | 0.58   | 0.51     |
| Blessé hospitalisé | 0.63    | 0.53   | 0.58     |

- Accuracy globale : **63 %**
- Rappel pour la classe "Tué" : **58 %**

Ce modèle, bien que plus simple, a permis **d’améliorer significativement la détection des cas les plus graves**, ce qui correspondait à l’objectif métier du projet.

### Variables les plus importantes

Le graphe des importances montre que les variables les plus déterminantes dans la prédiction sont :

1. `agg` : type de zone (agglomération)
2. `catv` : type de véhicule
3. `catr` : type de route
4. `obs` : obstacle rencontré
5. `catu` : type d’usager
6. `plan` : profil de la route
7. `sexe` : sexe de l’usager
8. `nbv` : nombre de voies
9. `circ` : régime de circulation
10. `choc` : type de choc


## 7. Interprétation métier

L’analyse du modèle final permet de mettre en évidence plusieurs facteurs critiques influençant la gravité d’un accident :

- Les **zones hors agglomération** sont plus dangereuses, probablement à cause de vitesses plus élevées.
- Les **véhicules à deux-roues**, les **poids lourds**, ou les **piétons** sont plus exposés aux cas graves.
- La présence d’un **obstacle** fixe ou mobile (mur, arbre, autre véhicule) aggrave fortement la situation.
- Les **manœuvres dangereuses** (dépassements, demi-tours, etc.) sont liées aux accidents mortels.
- Les accidents sur **routes sinueuses ou pentues** sont plus souvent associés à des issues graves.
- Certaines caractéristiques personnelles comme **le sexe** ou **le type d’usager** (piéton, passager...) influencent également la gravité.

Ces observations peuvent être utilisées pour :

- Prioriser des **zones d'intervention routière** ou de sécurité
- Développer des **politiques de prévention ciblées**
- Orienter la **tarification du risque** pour les assurances
- Intégrer des **alertes prédictives dans les systèmes embarqués**

## 8. Conclusion et perspectives

Ce projet a permis de construire un modèle de classification multi-classes visant à prédire la gravité d’un accident de la route en France, à partir de données issues de la période 2005–2021. À travers une démarche rigoureuse (exploration des données, nettoyage, rééquilibrage, optimisation), nous avons obtenu un modèle performant, à la fois sur le plan global (accuracy de 63 %) et, surtout, sur la détection des cas les plus graves (rappel de 58 % pour les accidents mortels).

Le recours à XGBoost, combiné à SMOTE et à une optimisation guidée par une métrique métier (le rappel sur la classe “Tué”), a permis de construire un modèle à la fois robuste, ciblé et relativement simple grâce à une sélection finale de 10 variables.

### Limites du modèle

Malgré ces bons résultats, plusieurs limitations doivent être soulignées :

- **Manque de données comportementales** : les données ne prennent pas en compte l’état du conducteur (alcool, téléphone, fatigue…), pourtant essentiel dans la réalité.
- **Approche statique** : le modèle ne prend en compte qu’une photographie de l’accident, sans données dynamiques ou séquentielles.
- **Rappel partiel sur les cas mortels** : malgré une nette amélioration, 42 % des accidents mortels restent non détectés par le modèle.

### Perspectives

Plusieurs pistes peuvent être explorées pour améliorer ce modèle ou l’adapter à d'autres contextes :

- **Ajout de nouvelles variables** issues d'autres sources (conditions météo précises, état du conducteur, densité de trafic, limitations de vitesse).
- **Approche hiérarchique ou séquentielle** : prédire d’abord la gravité globale, puis affiner entre les cas graves (hospitalisation vs décès).
- **Évaluation dans un cadre temps réel** pour des applications embarquées (voitures connectées, alertes GPS...).
- **Déploiement d’un outil interactif** permettant de simuler la gravité d’un accident selon des paramètres choisis (dashboard, application web...).

Ce projet démontre qu’un algorithme d’apprentissage automatique peut efficacement contribuer à mieux comprendre et anticiper la gravité des accidents routiers, à condition de rester conscient de ses limites et de continuer à l’enrichir avec des données plus fines et pertinentes.

