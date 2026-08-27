# Analyse de données d'assurance auto & télématique

DU Big Data, Data Science et Analyse des Risques sous Python — Université de Montpellier, Faculté d'économie (2023-2024)

**Auteurs :** Babacar Gueye, Amadou Sakho Ndiaye, Kobenan Gboko

## Contexte

Projet réalisé pour une insurtech fictive, visant à exploiter des bases de données d'assurance auto (sinistres, contrats, télématique de conduite) afin d'en extraire des informations utiles à la compréhension des assurés et à l'amélioration des décisions de tarification.

## Données

Trois sources sont combinées :

| Fichier | Contenu | Dimensions |
|---|---|---|
| `DB_SIN.txt` | Sinistres : `Id_pol`, `NB_Claim`, `AMT_Claim` | 4 337 lignes × 3 colonnes |
| `DB_CNT.xlsx` | Contrats : âge, sexe, statut marital, véhicule, crédit score, région, territoire, ancienneté sans sinistre | 100 399 lignes × 12 colonnes |
| `DB_TELEMATICS.csv` | Données de conduite télématique (accélération, freinage, virages, % de conduite par jour/heure) | 100 332 lignes |

> `DB_TELEMATICS.csv` est fourni compressé (`.csv.gz`) pour respecter la limite d'upload de GitHub. Décompressez-le avec `gzip -d DB_TELEMATICS.csv.gz` avant utilisation.

## Démarche

1. **Prétraitement** : gestion des valeurs manquantes, doublons, formats de types, cohérence des bornes (ex. `Duration` ∈ [22, 366], `Insured.age` ∈ [16, 103], sommes de `Pct.drive.*` = 100 %).
2. **Visualisation** : boxplots, histogrammes, heatmap de corrélation après fusion des bases.
3. **Feature engineering** : régularisation des variables télématiques redondantes, création d'un score composite **Driving Style** (Safe / Aggressive) à partir des moyennes d'accélération, freinage et virages.
4. **Jointures** : `fusion_df1` (contrats + télématique), puis `fusion_df2` (+ sinistres, via left join).
5. **Modélisation** (Random Forest, le plus performant) :
   - Prédiction du **Driving Style** à partir du profil de l'assuré
   - Prédiction du **nombre de sinistres** (`NB_Claim`)
   - Tentative de prédiction du **montant des sinistres** (`AMT_Claim`) — abandonnée (accuracy ~15 %, temps de calcul élevé) : cette variable relève davantage d'un exercice de tarification que de classification.
6. **Tarification** : clustering K-means (méthode du coude) sur un jeu de variables pertinentes (âge, sexe, sinistres, kilométrage, driving style, région, véhicule, crédit score) pour définir des classes tarifaires.
7. **Dashboard Shiny (Python)** : interface interactive affichant les indicateurs clés (nombre d'assurés, âge moyen, montant moyen des réclamations, kilométrage) et des visualisations filtrables par profil d'assuré.

## Résultats clés

- Le **Random Forest** est le meilleur algorithme testé pour la classification du style de conduite et du nombre de sinistres.
- Le montant des sinistres (`AMT_Claim`) n'est pas prédictible avec une précision satisfaisante en classification directe ; une approche de tarification actuarielle est plus appropriée.
- La segmentation par K-means aboutit à des classes tarifaires simples mais robustes, dont le prix moyen couvre l'espérance mathématique du risque par cluster.

Le détail complet de la méthodologie, des choix de traitement et des résultats est disponible dans [`report/Rapport_GB.pdf`](report/Rapport_GB.pdf).

## Structure du dépôt

```
.
├── notebooks/
│   └── data_cleaning.ipynb     # Prétraitement, visualisation, feature engineering, modélisation
├── data/
│   ├── DB_SIN.txt
│   ├── DB_CNT.xlsx
│   └── DB_TELEMATICS.csv.gz    # à décompresser avant usage
├── report/
│   └── Rapport_GB.pdf          # Rapport complet du projet
└── README.md
```

## Reproduire l'analyse

```bash
git clone <url-du-repo>
cd <repo>
gzip -d data/DB_TELEMATICS.csv.gz
pip install -r requirements.txt
jupyter notebook notebooks/data_cleaning.ipynb
```

## Limites et perspectives

- La prédiction du montant des sinistres reste un point faible ; une modélisation actuarielle dédiée (GLM Gamma/Tweedie) serait plus adaptée qu'une classification.
- Le clustering K-means avec la méthode du coude n'est pas la segmentation la plus dynamique ; des méthodes de tarification plus fines (GBM, tarification par variable continue) pourraient affiner les classes de risque.
