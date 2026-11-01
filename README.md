# Analyse des importations alimentaires des Comores

Ce projet propose une étude statistique, exploratoire et structurelle des importations alimentaires de l'Union des Comores à partir des données de la FAO (FAOSTAT).

L'objectif est d'analyser la dépendance alimentaire du pays, d'identifier les principaux produits importés et pays fournisseurs, d'évaluer la concentration des approvisionnements à l'aide d'indicateurs dédiés (CR1, CR3, HHI), et de formaliser les contraintes méthodologiques liées à la discontinuité des séries temporelles douanières.

---

## Sommaire

- [Contexte](#contexte)
- [Structure du projet](#structure-du-projet)
- [Données et périmètre](#données-et-périmètre)
- [Principes méthodologiques](#principes-méthodologiques)
- [Pipeline d'analyse](#pipeline-danalyse)
- [Résultats principaux](#résultats-principaux)
- [Limites méthodologiques](#limites-méthodologiques)
- [Installation et exécution](#installation-et-exécution)
- [Dépendances](#dépendances)

---

## Contexte

En tant que petit État insulaire en développement (PEID), l'Union des Comores dépend fortement des importations maritimes pour subvenir aux besoins alimentaires de sa population. Cette situation expose l'archipel aux fluctuations des marchés internationaux, aux coûts de transport et aux chocs d'approvisionnement.

Ce projet documente de manière empirique :
1. Les denrées prépondérantes en volume et en valeur.
2. Les pays partenaires et la structure géographique des approvisionnements.
3. Le niveau de concentration des importations produit par produit.
4. L'évolution des volumes, des montants et de la valeur unitaire moyenne (USD/tonne).
5. La pertinence (et les limites) d'une modélisation prédictive face aux discontinuités de déclaration.

---

## Structure du projet

```text
importations-alimentaires-comores/
├── data/
│   ├── raw/
│   │   └── FAOSTAT_data.csv            # Données brutes FAO (non modifiées)
│   └── processed/
│       ├── imports_comoros_processed.csv   # Données nettoyées (produit-année-pays)
│       ├── product_catalog.csv             # Référentiel des 344 produits CPC
│       ├── product_classification.csv      # Classification par type d'usage
│       └── supplier_concentration.csv      # Indicateurs de concentration (CR1, CR3, HHI)
├── figures/                            # Emplacement pour les figures générées
├── notebooks/
│   ├── 01_preparation_donnees.ipynb    # Ingestion, filtrage et mise en forme
│   ├── 02_analyse_exploratoire.ipynb   # Statistiques descriptives et classements
│   ├── 03_analyse_statistique.ipynb    # Distributions, corrélations et tests
│   ├── 04_analyse_concentration.ipynb  # Analyse de concentration des fournisseurs
│   └── 05_modelisation_predictive.ipynb# Diagnostic de série temporelle et modèle de test
├── requirements.txt                    # Liste des dépendances
└── README.md                           # Documentation du projet
```

---

## Données et périmètre

- **Source** : FAOSTAT, matrice détaillée du commerce (*Detailed Trade Matrix - TM*).
- **Pays déclarant** : Comores (code M49 : 174).
- **Classification** : Central Product Classification (CPC).
- **Périodes disponibles** : 14 années au total :
  - 1995 à 2002 (8 années).
  - 2016 à 2021 (6 années).
  - Période non renseignée dans la base détaillée : 2003 à 2015 (13 années de rupture).
- **Unités retenues** :
  - Quantités en tonnes (`t`).
  - Valeurs en milliers de dollars US (`1000 USD`).
  - Les flux exprimés en têtes de bétail (`An`, `1000 An`) sont écartés des sommes physiques pour préserver la cohérence des tonnages.

---

## Principes méthodologiques

1. **Intégrité des données brutes** : Le fichier source dans `data/raw/` est conservé dans son état d'origine. Les transformations sont écrites exclusivement dans `data/processed/`.
2. **Traitement des valeurs manquantes** : Les données absentes ne sont pas automatiquement remplacées par des zéros. Une cellule vide indique une absence d'enregistrement, distincte d'un volume nul.
3. **Classification des produits** : Le fichier `data/processed/product_classification.csv` fournit un modèle répertoriant les 344 produits. Les catégories par défaut sont positionnées sur `À vérifier` afin de permettre une validation manuelle progressive (alimentation humaine, animale, usage mixte, non alimentaire).
4. **Continuité temporelle** : Compte tenu de l'interruption de 13 années (2003–2015), aucune interpolation artificielle n'est appliquée entre les deux périodes.

---

## Pipeline d'analyse

Les notebooks sont organisés de manière séquentielle :

### 1. 01_preparation_donnees.ipynb
- Chargement de `data/raw/FAOSTAT_data.csv` et vérification des types.
- Filtrage sur le pays déclarant (Comores) et harmonisation des flux d'importation en tonnes et en milliers d'USD.
- Vérification des drapeaux de qualité (*flags* `A`, `E`, `I`, `X`).
- Extraction du catalogue des 344 produits CPC.
- Génération de la table produit-année-pays dans `data/processed/imports_comoros_processed.csv`.

### 2. 02_analyse_exploratoire.ipynb
- Synthèse globale : 344 produits, 124 pays partenaires, 8 162 flux enregistrés.
- Classement des produits les plus importés en tonnage et en valeur monétaire.
- Classement des principaux partenaires commerciaux.
- Évolution temporelle des quantités globales et des montants en devises.
- Calcul de la valeur unitaire moyenne pondérée annuelle (USD/tonne).

### 3. 03_analyse_statistique.ipynb
- Étude des distributions des tonnages et des valeurs (forte asymétrie à droite, médiane basse par rapport aux valeurs maximales).
- Corrélations de Pearson et de Spearman entre quantités et valeurs.
- Analyse des valeurs unitaires par couple produit-année.
- Test non paramétrique de Mann-Whitney comparant les distributions des valeurs de la période 1995–2002 à celles de la période 2016–2021 ($U = 7\,000\,614.5$, $p < 10^{-25}$). Ce test est présenté à titre purement descriptif, les observations successives n'étant pas indépendantes.

### 4. 04_analyse_concentration.ipynb
- Calcul annuel des indicateurs de concentration par produit :
  - **CR1** : Part de marché du principal pays fournisseur.
  - **CR3** : Part cumulée des trois premiers fournisseurs.
  - **HHI** : Indice d'Herfindahl-Hirschman ($\sum s_i^2$).
- Identification des denrées soumises à un fournisseur unique ($CR1 = 1.0$) ou à un marché très concentré ($HHI > 0.5$).
- Possibilité d'examiner en détail la décomposition d'un code CPC spécifique.
- Sauvegarde des métriques dans `data/processed/supplier_concentration.csv`.

### 5. 05_modelisation_predictive.ipynb
- Diagnostic de la série temporelle et mise en évidence de la coupure de 13 ans (2003–2015).
- Extraction de la plus longue sous-série continue (8 ans, de 1995 à 2002).
- Évaluation d'une régression linéaire avec découpage train/test chronologique.
- Conclusion méthodologique : la régression linéaire sur ce segment produit un $R^2$ négatif ($-14.29$), démontrant qu'une projection prédictive simple sur des séries aussi courtes et discontinues n'est pas fiable. Le notebook formalise cette limite et oriente les travaux vers l'analyse de concentration et la détection de ruptures plutôt que la prévision pure.

---

## Résultats principaux

| Indicateur | Valeur observée | Commentaire |
| :--- | :--- | :--- |
| **Volume cumulé** | ~1,70 million de tonnes | Cumul sur les 14 années disponibles. |
| **Valeur cumulée** | ~951,2 millions USD | Montant total des importations enregistrées. |
| **Denrée dominante** | Riz (paddy, brisé, blanchi) | Représente près de la moitié de la valeur totale importée (> 440 millions USD). |
| **Source de protéines** | Viande et abats de volaille | Produits figurant parmi les 5 premiers postes d'importation en valeur (> 125 millions USD cumulés). |
| **Premier partenaire** | Pakistan | Plus de 354 millions USD sur la période (fournisseur majeur de riz). |
| **Autres partenaires clés** | France, Inde, EAU, Afrique du Sud | Fournisseurs diversifiés en produits transformés, céréales et oléagineux. |
| **Évolution du prix moyen** | De ~400 USD/t à ~745 USD/t | Tendance à la hausse de la valeur unitaire moyenne entre 1995 et 2021. |
| **Concentration** | Élevée sur les denrées de base | Nombreux produits stratégiques approvisionnés auprès d'un unique pays au cours d'une année donnée. |

---

## Limites méthodologiques

1. **Rupture de série (2003–2015)** : L'absence de données détaillées sur 13 années consécutives interdit l'utilisation de modèles de prévision temporelle classiques (ARIMA, exponentiel lissé) sur l'ensemble de la période.
2. **Dépendance des observations** : Les lignes de flux douaniers ne sont pas statistiquement indépendantes (effets pays, saisonnalité, accords commerciaux). Les tests d'hypothèses doivent être lus comme des descripteurs de contraste et non comme des preuves causales.
3. **Hétérogénéité des agrégats CPC** : Certains codes regroupent des sous-catégories variées dont les valeurs unitaires diffèrent sensiblement selon les années.
4. **Validation de la classification** : La colonne `Product_Category` dans `data/processed/product_classification.csv` doit être renseignée manuellement pour permettre une analyse catégorielle complète.

---

## Installation et exécution

### 1. Cloner le projet
```bash
git clone https://github.com/Nayer878/importations-alimentaires-comores.git
cd importations-alimentaires-comores
```

### 2. Prérequis
Python 3.10 ou supérieur.

### 3. Environnement virtuel
```bash
python -m venv .venv
# Sous Windows PowerShell :
.venv\Scripts\Activate.ps1
# Sous Linux / macOS :
source .venv/bin/activate
```

### 4. Dépendances
```bash
pip install -r requirements.txt
```

### 5. Fichier source
Vérifier la présence du fichier brut dans :
```text
data/raw/FAOSTAT_data.csv
```

### 6. Ordre d'exécution des notebooks
1. `notebooks/01_preparation_donnees.ipynb`
2. *(Facultatif)* Ajuster `data/processed/product_classification.csv`.
3. `notebooks/02_analyse_exploratoire.ipynb`
4. `notebooks/03_analyse_statistique.ipynb`
5. `notebooks/04_analyse_concentration.ipynb`
6. `notebooks/05_modelisation_predictive.ipynb`

---

## Dépendances

Les bibliothèques utilisées figurent dans `requirements.txt` :

- `pandas` : Traitement et agrégation des données tabulaires.
- `numpy` : Opérations numériques.
- `matplotlib` et `seaborn` : Graphiques et visualisations.
- `scipy` : Tests statistiques non paramétriques.
- `scikit-learn` : Métriques d'évaluation et régression.
- `jupyter` et `nbformat` : Exécution des notebooks.
