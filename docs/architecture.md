# 🏗️ Architecture du projet - Pipeline Éolienne

## Vue d'ensemble

Ce document décrit l'architecture complète du pipeline de données pour l'analyse de production d'énergie éolienne avec Microsoft Fabric.

## 🎨 Architecture Medallion

L'architecture Medallion organise les données en **3 couches** progressivement enrichies :

### Flux de données

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          MICROSOFT FABRIC WORKSPACE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌─────────┐       ┌─────────┐       ┌─────────┐       ┌─────────┐     │
│  │ GitHub  │──────▶│ BRONZE  │──────▶│ SILVER  │──────▶│  GOLD   │───┐ │
│  │ CSV     │       │Lakehouse│       │Lakehouse│       │Lakehouse│   │ │
│  │ Source  │       └─────────┘       └─────────┘       └─────────┘   │ │
│  └─────────┘            │                  │                 │        │ │
│                         │                  │                 │        │ │
│                    Delta Lake         Delta Lake        Star Schema   │ │
│                    Raw Data          Cleaned Data      Dimensions +   │ │
│                                                         Facts          │ │
│                                                                        │ │
│                                                                        │ │
│  ┌──────────────────────────────────────────────────────────────────┐ │ │
│  │               DATA FACTORY PIPELINE                              │ │ │
│  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐     │ │ │
│  │  │ Ingest   │──▶│ Bronze→  │──▶│ Silver→  │──▶│ Create   │     │ │ │
│  │  │ Bronze   │   │ Silver   │   │ Gold     │   │ Semantic │     │ │ │
│  │  │ Notebook │   │ Notebook │   │ Notebooks│   │ Model    │     │ │ │
│  │  └──────────┘   └──────────┘   └──────────┘   └──────────┘     │ │ │
│  └──────────────────────────────────────────────────────────────────┘ │ │
│                                                                        │ │
│  ┌──────────────────────────────────────────────────────────────────┐ │ │
│  │                    SEMANTIC MODEL                                │ │ │
│  │                    (Star Schema)                                 │ │ │
│  │   ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐               │ │ │
│  │   │dim_date│  │dim_time│  │dim_    │  │dim_    │               │ │ │
│  │   │        │  │        │  │turbine │  │ status │               │ │ │
│  │   └───┬────┘  └───┬────┘  └───┬────┘  └───┬────┘               │ │ │
│  │       │           │           │           │                     │ │ │
│  │       └───────────┴───────────┴───────────┘                     │ │ │
│  │                          │                                       │ │ │
│  │                   ┌──────▼──────┐                               │ │ │
│  │                   │fact_        │                               │ │ │
│  │                   │production   │                               │ │ │
│  │                   └─────────────┘                               │ │ │
│  └──────────────────────────────────────────────────────────────────┘ │ │
│                                                                        │ │
│  ┌──────────────────────────────────────────────────────────────────┐ │ │
│  │                      POWER BI REPORTS                            │◄┘ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │   │
│  │  │ Dashboard   │  │ Geographic  │  │ Temporal    │             │   │
│  │  │ Overview    │  │ Analysis    │  │ Analysis    │             │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘             │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## 🥉 Bronze Layer - Données brutes

### Objectif
Ingérer les données brutes depuis GitHub sans transformation.

### Technologies
- **Lakehouse** : Bronze Lakehouse
- **Format** : Delta Lake
- **Notebook** : `ingestion_bronze.ipynb`

### Table
- **Nom** : `bronze_eolienne_production`
- **Colonnes** : datetime, turbine_name, latitude, longitude, region, department, wind_speed, wind_direction, energy_kwh, operational_status

### Caractéristiques
- ✅ Données brutes, aucune transformation
- ✅ Format source conservé (CSV → Delta)
- ✅ Ingestion incrémentale
- ✅ Historisation complète

---

## 🥈 Silver Layer - Données nettoyées

### Objectif
Nettoyer, standardiser et enrichir les données Bronze.

### Technologies
- **Lakehouse** : Silver Lakehouse
- **Format** : Delta Lake
- **Notebooks** :
  - `bronze_to_silver_pyspark.ipynb` (PySpark)
  - `bronze_to_silver_sql.ipynb` (SQL - optionnel)

### Table
- **Nom** : `silver_eolienne_production`
- **Colonnes originales** : Toutes les colonnes Bronze (nettoyées)
- **Colonnes ajoutées** :
  - `date` : Date extraite
  - `time` : Heure corrigée (format HH:MM)
  - `day`, `month`, `year` : Composantes temporelles
  - `period` : Période de la journée (Matin/Après-midi/Soir/Nuit)

### Transformations appliquées
1. ✅ Extraction de date et time depuis datetime
2. ✅ Correction du format time ("14-30" → "14:30")
3. ✅ Arrondi des valeurs numériques à 2 décimales
4. ✅ Calcul de la période de la journée
5. ✅ Gestion des valeurs nulles
6. ✅ Standardisation des formats

---

## 🥇 Gold Layer - Modèle dimensionnel

### Objectif
Créer un Star Schema optimisé pour l'analytique et Power BI.

### Technologies
- **Lakehouse** : Gold Lakehouse
- **Format** : Delta Lake
- **Notebooks** :
  - `silver_to_gold_dimensions.ipynb` (créer les dimensions)
  - `silver_to_gold_facts.ipynb` (créer la table de faits)

### Star Schema

#### Dimensions (4)

1. **dim_date** : Calendrier
   - PK : `date_id` (int - format YYYYMMDD)
   - Colonnes : date, day, month, year, quarter, day_of_week, is_weekend

2. **dim_time** : Heures
   - PK : `time_id` (int - format HHMM)
   - Colonnes : time, hour, minute, period

3. **dim_turbine** : Turbines
   - PK : `turbine_id` (int)
   - Colonnes : turbine_name, latitude, longitude, region, department

4. **dim_operational_status** : Statuts
   - PK : `status_id` (int)
   - Colonnes : operational_status, status_description

#### Fait (1)

**fact_production** : Production d'énergie
- PK : `production_id` (bigint)
- FK : `date_id`, `time_id`, `turbine_id`, `status_id`
- Mesures : `energy_kwh`, `wind_speed`, `wind_direction`

---

## 🔄 Orchestration - Data Factory Pipeline

### Composants

1. **Activité 1** : Notebook `ingestion_bronze.ipynb`
   - Charge les données depuis GitHub vers Bronze

2. **Activité 2** : Notebook `bronze_to_silver_pyspark.ipynb`
   - Transforme Bronze → Silver
   - Dépend de l'Activité 1

3. **Activité 3** : Notebook `silver_to_gold_dimensions.ipynb`
   - Crée les dimensions dans Gold
   - Dépend de l'Activité 2

4. **Activité 4** : Notebook `silver_to_gold_facts.ipynb`
   - Crée la table de faits dans Gold
   - Dépend de l'Activité 3

### Ordre d'exécution

```
Ingestion Bronze
      ↓
Bronze → Silver
      ↓
Silver → Gold (Dimensions)
      ↓
Silver → Gold (Facts)
```

### Planification
- **Mode** : Automatique (Scheduled)
- **Fréquence** : Quotidienne / Hebdomadaire / Sur demande

---

## 📊 Semantic Model - Power BI

### Relations

```
dim_date (1) ────────── (*) fact_production
dim_time (1) ────────── (*) fact_production
dim_turbine (1) ──────── (*) fact_production
dim_operational_status (1) ── (*) fact_production
```

### Mesures DAX (exemples)

```dax
Total Energy = SUM(fact_production[energy_kwh])

Average Wind Speed = AVERAGE(fact_production[wind_speed])

Number of Turbines = DISTINCTCOUNT(dim_turbine[turbine_id])

Energy YTD = TOTALYTD([Total Energy], dim_date[date])
```

### Hiérarchies

- **Date** : Année → Trimestre → Mois → Jour
- **Time** : Période → Heure → Minute
- **Géographie** : Région → Département → Turbine

---

## 📈 Power BI Reports

### Rapport 1 : Overview Dashboard
- KPIs : Production totale, moyenne par turbine, nombre de turbines actives
- Graphiques : Évolution temporelle, Top turbines, Répartition géographique

### Rapport 2 : Analyse géographique
- Carte des turbines
- Production par région/département
- Analyse des conditions de vent par zone

### Rapport 3 : Analyse temporelle
- Production par période de la journée
- Saisonnalité
- Comparaisons année sur année

---

## 🛠️ Technologies utilisées

| Composant | Technologie |
|-----------|-------------|
| **Plateforme** | Microsoft Fabric |
| **Stockage** | OneLake (Data Lake) |
| **Format** | Delta Lake |
| **Compute** | Apache Spark (PySpark) |
| **Transformation** | PySpark + Spark SQL |
| **Orchestration** | Data Factory (Pipelines) |
| **Modélisation** | Semantic Model (Power BI) |
| **Visualisation** | Power BI |
| **Versioning** | Git / GitHub |

---

## 📂 Organisation dans Fabric Workspace

```
Workspace: Fabric-Eolienne-Project
│
├── 🏠 Lakehouses (3)
│   ├── Bronze_Lakehouse
│   ├── Silver_Lakehouse
│   └── Gold_Lakehouse
│
├── 📓 Notebooks (5-7)
│   ├── ingestion_bronze.ipynb
│   ├── bronze_to_silver_pyspark.ipynb
│   ├── bronze_to_silver_sql.ipynb (optionnel)
│   ├── silver_to_gold_dimensions.ipynb
│   └── silver_to_gold_facts.ipynb
│
├── 🔄 Pipelines (1)
│   └── Eolienne_Master_Pipeline
│
├── 📊 Semantic Models (1)
│   └── Eolienne_Semantic_Model
│
└── 📈 Reports (2-3)
    ├── Eolienne_Auto_Report
    └── Eolienne_Custom_Report
```

---

## ✅ Checklist de réalisation

### Environnement
- [ ] Trial Microsoft Fabric activé
- [ ] Workspace créé
- [ ] Repository GitHub initialisé

### Bronze Layer
- [ ] Bronze Lakehouse créé
- [ ] Notebook d'ingestion développé
- [ ] Données chargées depuis GitHub
- [ ] Table Bronze créée (Delta)

### Silver Layer
- [ ] Silver Lakehouse créé
- [ ] Notebook PySpark Bronze→Silver développé
- [ ] Transformations appliquées (nettoyage, enrichissement)
- [ ] Table Silver créée

### Gold Layer
- [ ] Gold Lakehouse créé
- [ ] Notebook Dimensions développé
- [ ] Notebook Facts développé
- [ ] Star Schema créé (4 dimensions + 1 fait)

### Orchestration
- [ ] Pipeline Data Factory créée
- [ ] Activités Notebook ajoutées
- [ ] Dépendances configurées
- [ ] Planification automatique activée

### Semantic Model
- [ ] Semantic Model créé
- [ ] Relations définies
- [ ] Mesures DAX créées
- [ ] Hiérarchies configurées

### Power BI
- [ ] Rapport auto-généré créé
- [ ] Rapport personnalisé développé
- [ ] Interactivité ajoutée (slicers, drill-down)

### Documentation
- [ ] README complet
- [ ] Captures d'écran ajoutées
- [ ] Schémas d'architecture
- [ ] Vidéo de démonstration

---

**Date de dernière mise à jour** : Février 2026
