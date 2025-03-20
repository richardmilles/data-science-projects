# Prédiction de l'Attrition des Employés

## 1. Introduction

### 1.1 Contexte du Projet
L'attrition des employés est un problème majeur dans de nombreuses entreprises. Une rotation élevée du personnel peut entraîner des coûts significatifs en recrutement, formation et perte de compétences. Comprendre les raisons des départs volontaires et prédire quels employés sont susceptibles de quitter permettrait aux entreprises d'anticiper ces départs et de prendre des mesures correctives.

### 1.2 Objectifs du Projet
L'objectif principal de ce projet est de développer un modèle de machine learning capable de prédire si un employé est susceptible de quitter l'entreprise. Plus précisément, nous cherchons à :
- Identifier les facteurs les plus influents sur l'attrition des employés.
- Construire un modèle prédictif précis et robuste.
- Fournir des recommandations aux équipes RH pour limiter l'attrition.

## 2. Exploration et Analyse des Données

### 2.1 Aperçu du Jeu de Données
Le jeu de données utilisé provient d'une entreprise fictive et contient 1480 observations et 38 variables. Ces variables couvrent différents aspects de l'emploi, notamment :
- Informations démographiques : Âge, genre, état civil.
- Caractéristiques professionnelles : Poste, département, ancienneté.
- Conditions de travail : Salaire, heures supplémentaires, satisfaction au travail.
- Variable cible : Attrition (Oui/Non), indiquant si l'employé a quitté l'entreprise.

### 2.2 Analyse Exploratoire
Avant de construire un modèle, il est essentiel d'explorer les données pour détecter les tendances et anomalies.
- Taux d'attrition global : Environ 16% des employés quittent l'entreprise.
- Influence des heures supplémentaires : Une grande partie des employés ayant effectué des heures supplémentaires ont quitté l'entreprise.
- Impact du salaire : Les employés aux salaires plus faibles semblent plus enclins à partir.
- Ancienneté et départs : Une proportion non négligeable d'employés partent après quelques années dans l'entreprise.

## 3. Modélisation et Expérimentations

### 3.1 Modèles Testés
Nous avons testé trois modèles différents pour comparer leurs performances :

- Régression Logistique : Précision Globale 85%, Recall (classe 1) 23%, Précision (classe 1) 61%
- Random Forest : Précision Globale 84%, Recall (classe 1) 49%, Précision (classe 1) 50%
- XGBoost : Précision Globale 83%, Recall (classe 1) 57%, Précision (classe 1) 47%

### 3.2 Choix du Meilleur Modèle
Le modèle XGBoost a été retenu car il offre le meilleur compromis entre précision et rappel. Son recall de 57% signifie qu'il détecte plus de la moitié des employés qui quitteront l'entreprise, ce qui est un résultat satisfaisant comparé aux autres modèles.

## 4. Analyse des Facteurs Clés de l'Attrition

### 4.1 Variables les Plus Importantes
D'après notre modèle, les variables ayant le plus d'impact sur l'attrition sont :
1. Heures supplémentaires (OverTime) : Une charge de travail excessive est un facteur de départ.
2. État civil - Célibataire : Les employés célibataires sont plus enclins à changer d'emploi.
3. Poste occupé : Les commerciaux et directeurs de production sont plus à risque.
4. Ancienneté (YearsAtCompany) : Un seuil critique semble exister au-delà duquel les employés partent.
5. Satisfaction au travail (JobSatisfaction) : Un faible niveau de satisfaction accroît le risque de départ.

## 5. Recommandations et Applications Pratiques

### 5.1 Actions RH Recommandées
1. Surveiller les heures supplémentaires et mettre en place des politiques pour réduire la surcharge de travail.
2. Offrir des perspectives de carrière claires pour les employés à risque.
3. Revoir les conditions de travail des commerciaux et directeurs de production.
4. Améliorer la satisfaction au travail en adaptant les rémunérations et les avantages sociaux.
5. Suivi individuel des employés à fort risque d'attrition via des entretiens réguliers.

## 6. Implémentation et Utilisation du Modèle

Le modèle final est sauvegardé pour des prédictions futures :

Une interface utilisateur basée sur Streamlit ou Flask pourrait être développée pour permettre une utilisation plus intuitive par les équipes RH.

## 7. Conclusion et Perspectives

Ce projet a permis de construire un outil d'aide à la décision pour anticiper l'attrition des employés. Les résultats montrent que les départs peuvent être partiellement prédits, permettant ainsi aux entreprises de prendre des mesures préventives.

### Perspectives d'Amélioration
- Exploration d'autres modèles plus complexes comme LightGBM.
- Intégration de nouvelles variables comme les évaluations de performance.
- Mise en place d'un système de suivi en temps réel pour anticiper les départs de manière proactive.
