# 🥇 Gold Layer - Modèle dimensionnel (Star Schema)

## 📋 Objectif

Cette couche transforme les données Silver en un **modèle dimensionnel** optimisé pour l'analytique et la consommation par Power BI. On implémente un **Star Schema** avec des tables de dimension et une table de faits.

## 📂 Notebooks

### `silver_to_gold_dimensions.ipynb`
**Objectif** : Créer les 4 tables de dimension

**Dimensions créées** :
1. ✅ `dim_date` : Calendrier complet
2. ✅ `dim_time` : Découpage horaire
3. ✅ `dim_turbine` : Caractéristiques des turbines
4. ✅ `dim_operational_status` : Statuts opérationnels

---

### `silver_to_gold_facts.ipynb`
**Objectif** : Créer la table de faits avec les métriques

**Fait créé** :
- ✅ `fact_production` : Production d'énergie avec clés étrangères vers les dimensions

## 🎯 Modèle dimensionnel

```
         ┌──────────────┐
         │   dim_date   │
         │──────────────│
         │ date_id (PK) │◄─┐
         │ date         │  │
         │ day, month   │  │
         │ year, quarter│  │
         └──────────────┘  │
                           │
         ┌──────────────┐  │     ┌───────────────────┐
         │   dim_time   │  │     │  fact_production  │
         │──────────────│  │     │───────────────────│
         │ time_id (PK) │◄─┼─────│ production_id (PK)│
         │ time         │  │     │ date_id (FK)      │
         │ hour, minute │  │     │ time_id (FK)      │
         │ period       │  │     │ turbine_id (FK)   │
         └──────────────┘  │     │ status_id (FK)    │
                           │     │                   │
         ┌──────────────┐  │     │ energy_kwh        │
         │ dim_turbine  │  │     │ wind_speed        │
         │──────────────│  │     │ wind_direction    │
         │ turbine_id   │◄─┤     └───────────────────┘
         │    (PK)      │  │
         │ turbine_name │  │
         │ latitude     │  │
         │ longitude    │  │
         │ region       │  │
         │ department   │  │
         └──────────────┘  │
                           │
         ┌──────────────┐  │
         │   dim_       │  │
         │ operational_ │  │
         │   status     │  │
         │──────────────│  │
         │ status_id    │◄─┘
         │    (PK)      │
         │ operational_ │
         │   status     │
         └──────────────┘
```

## 📊 Tables de dimension

### 📅 dim_date
**Description** : Calendrier complet avec hiérarchies temporelles

| Colonne | Type | Description |
|---------|------|-------------|
| `date_id` | int | Clé primaire (format YYYYMMDD) |
| `date` | date | Date complète |
| `day` | int | Jour du mois (1-31) |
| `month` | int | Mois (1-12) |
| `month_name` | string | Nom du mois (Janvier, Février...) |
| `year` | int | Année |
| `quarter` | int | Trimestre (1-4) |
| `day_of_week` | int | Jour de la semaine (1-7) |
| `day_name` | string | Nom du jour (Lundi, Mardi...) |
| `is_weekend` | boolean | Weekend ou non |

---

### 🕐 dim_time
**Description** : Découpage horaire de la journée

| Colonne | Type | Description |
|---------|------|-------------|
| `time_id` | int | Clé primaire (format HHMM) |
| `time` | string | Heure (format HH:MM) |
| `hour` | int | Heure (0-23) |
| `minute` | int | Minute (0-59) |
| `period` | string | Matin/Après-midi/Soir/Nuit |

---

### 🌀 dim_turbine
**Description** : Caractéristiques des turbines éoliennes

| Colonne | Type | Description |
|---------|------|-------------|
| `turbine_id` | int | Clé primaire |
| `turbine_name` | string | Nom de la turbine |
| `latitude` | decimal | Latitude GPS |
| `longitude` | decimal | Longitude GPS |
| `region` | string | Région |
| `department` | string | Département |

---

### 🔧 dim_operational_status
**Description** : Statuts opérationnels des turbines

| Colonne | Type | Description |
|---------|------|-------------|
| `status_id` | int | Clé primaire |
| `operational_status` | string | Statut (Active, Maintenance, etc.) |
| `status_description` | string | Description détaillée |

## 📊 Table de faits

### ⚡ fact_production
**Description** : Faits de production d'énergie

| Colonne | Type | Description |
|---------|------|-------------|
| `production_id` | bigint | Clé primaire |
| `date_id` | int | FK vers dim_date |
| `time_id` | int | FK vers dim_time |
| `turbine_id` | int | FK vers dim_turbine |
| `status_id` | int | FK vers dim_operational_status |
| **Mesures** | | |
| `energy_kwh` | decimal | Production d'énergie (kWh) |
| `wind_speed` | decimal | Vitesse du vent |
| `wind_direction` | decimal | Direction du vent |

## 🔄 Logique de transformation

```python
# Pseudo-code

# 1. Créer dim_date
1. Extraire les dates uniques de Silver
2. Générer date_id (YYYYMMDD)
3. Calculer quarter, day_of_week, is_weekend, etc.
4. Écrire dans dim_date

# 2. Créer dim_time
1. Extraire les heures uniques de Silver
2. Générer time_id (HHMM)
3. Extraire hour, minute, period
4. Écrire dans dim_time

# 3. Créer dim_turbine
1. Extraire les turbines uniques de Silver
2. Générer turbine_id (auto-increment)
3. Conserver name, lat, long, region, department
4. Écrire dans dim_turbine

# 4. Créer dim_operational_status
1. Extraire les statuts uniques de Silver
2. Générer status_id
3. Ajouter descriptions
4. Écrire dans dim_operational_status

# 5. Créer fact_production
1. Lire Silver
2. Joindre avec les dimensions pour récupérer les IDs
3. Générer production_id unique
4. Conserver les mesures (energy_kwh, wind_speed, wind_direction)
5. Écrire dans fact_production
```

## 🎯 Avantages du Star Schema

- ✅ **Performance** : Requêtes optimisées avec moins de jointures
- ✅ **Simplicité** : Modèle facile à comprendre
- ✅ **Flexibilité** : Facile d'ajouter de nouvelles dimensions
- ✅ **Power BI** : Modèle idéal pour les cubes OLAP

## 🚀 Exécution

Ces notebooks seront exécutés par la **Data Pipeline** en troisième étape, après la transformation Silver.

Ordre d'exécution :
1. `silver_to_gold_dimensions.ipynb` (créer les dimensions d'abord)
2. `silver_to_gold_facts.ipynb` (créer la table de faits ensuite)

## ⚠️ Points d'attention

- Les dimensions doivent être créées AVANT la table de faits
- Vérifier l'unicité des clés primaires
- Gérer les clés de substitution (surrogate keys)
- Documenter les relations dans Power BI

---

**Étape précédente** : [Silver Layer](../silver/README.md)
**Prochaine étape** : Semantic Model + Power BI
