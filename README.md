# 🌬️ Pipeline de Données - Analyse Production Éolienne

> Projet de pipeline de données end-to-end avec **Microsoft Fabric** utilisant l'architecture **Medallion** (Bronze/Silver/Gold)

[![Microsoft Fabric](https://img.shields.io/badge/Microsoft_Fabric-0078D4?style=flat&logo=microsoft&logoColor=white)](https://fabric.microsoft.com)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apache-spark&logoColor=white)](https://spark.apache.org/)
[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat&logo=power-bi&logoColor=black)](https://powerbi.microsoft.com/)

---

## 📋 Vue d'ensemble

Ce projet implémente un **pipeline de données complet** pour analyser la production d'énergie de turbines éoliennes. Il utilise l'architecture **Medallion** (Bronze → Silver → Gold) et l'écosystème Microsoft Fabric.

### 🎯 Objectifs du projet

- ✅ Ingérer des données de production éolienne depuis GitHub
- ✅ Nettoyer et transformer les données avec PySpark/SQL
- ✅ Créer un modèle dimensionnel (Star Schema)
- ✅ Orchestrer le pipeline avec Data Factory
- ✅ Visualiser les résultats dans Power BI

### 📊 Données analysées

Les données proviennent de plusieurs turbines éoliennes et incluent :
- 🕐 Date et heure de production
- 🌀 Caractéristiques des turbines
- 📍 Localisation géographique
- 💨 Conditions de vent (vitesse, direction)
- ⚡ Production d'énergie (kWh)
- 🔧 Statut opérationnel

**Source des données** : [GitHub - data-training-fabric](https://github.com/gsoulat/data-training-fabric/tree/main/eolienne)

---

## 🏗️ Architecture

### Architecture Medallion

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   GitHub    │─────▶│   BRONZE    │─────▶│   SILVER    │─────▶│    GOLD     │─────▶│  Power BI   │
│  (CSV Data) │      │ (Raw Data)  │      │  (Cleaned)  │      │(Dimensional)│      │  (Reports)  │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
                            ↓                     ↓                     ↓
                     Delta Tables          Delta Tables          Star Schema
```

### 🥉 Bronze Layer - Données brutes
- Ingestion des fichiers CSV depuis GitHub
- Stockage au format Delta Lake
- Aucune transformation, données brutes conservées

### 🥈 Silver Layer - Données nettoyées
- Nettoyage et standardisation
- Enrichissement avec colonnes calculées (jour, mois, année, période)
- Correction des formats
- Arrondi des valeurs numériques

### 🥇 Gold Layer - Modèle dimensionnel
- **Star Schema** optimisé pour l'analytique
- **4 Tables de dimension** :
  - `dim_date` : Calendrier complet
  - `dim_time` : Découpage horaire
  - `dim_turbine` : Caractéristiques des turbines
  - `dim_operational_status` : Statuts opérationnels
- **1 Table de faits** :
  - `fact_production` : Mesures de production

---

## 📁 Structure du projet

```
fabric-eolienne-project/
├── README.md                      # 📖 Documentation principale (ce fichier)
├── .gitignore                     # 🚫 Fichiers à ignorer
│
├── notebooks/                     # 📓 Notebooks Jupyter
│   ├── bronze/                    # Ingestion des données
│   │   ├── ingestion_bronze.ipynb
│   │   └── README.md
│   ├── silver/                    # Transformations Silver
│   │   ├── bronze_to_silver_pyspark.ipynb
│   │   ├── bronze_to_silver_sql.ipynb
│   │   └── README.md
│   └── gold/                      # Transformations Gold
│       ├── silver_to_gold_dimensions.ipynb
│       ├── silver_to_gold_facts.ipynb
│       └── README.md
│
├── docs/                          # 📚 Documentation
│   ├── architecture.md            # Schémas d'architecture
│   └── screenshots/               # Captures d'écran
│
├── data/                          # 📊 Données locales
│   └── sample/                    # Échantillons pour tests
│
└── pipeline/                      # 🔄 Configuration Pipeline
    └── pipeline_config.json
```

---

## 🚀 Étapes de réalisation

### ✅ Étape 0 : Préparation de l'environnement
- [ ] Créer un compte Microsoft Fabric (trial)
- [ ] Créer le Workspace Fabric
- [ ] Configurer Git/GitHub

### ✅ Étape 1 : Création des Lakehouses
- [ ] Lakehouse Bronze
- [ ] Lakehouse Silver
- [ ] Lakehouse Gold

### ✅ Étape 2 : Ingestion Bronze
- [ ] Créer le notebook d'ingestion
- [ ] Implémenter la logique incrémentale
- [ ] Charger les données depuis GitHub

### ✅ Étape 3 : Transformation Bronze → Silver
- [ ] Notebook PySpark pour nettoyage
- [ ] Notebook SQL (alternative)
- [ ] Enrichissement des données

### ✅ Étape 4 : Transformation Silver → Gold
- [ ] Créer les 4 tables de dimension
- [ ] Créer la table de faits
- [ ] Implémenter le Star Schema

### ✅ Étape 5 : Orchestration avec Data Factory
- [ ] Créer la Data Pipeline
- [ ] Ajouter les activités Notebook
- [ ] Planifier l'exécution automatique

### ✅ Étape 6 : Semantic Model
- [ ] Créer le modèle sémantique
- [ ] Définir les relations
- [ ] Créer les mesures DAX

### ✅ Étape 7 : Rapports Power BI
- [ ] Rapport auto-généré
- [ ] Rapport personnalisé
- [ ] Ajouter interactivité

### ✅ Étape 8 : Documentation finale
- [ ] Compléter ce README
- [ ] Ajouter captures d'écran
- [ ] Créer la vidéo de démonstration

---

## 🛠️ Technologies utilisées

| Technologie | Usage |
|-------------|-------|
| **Microsoft Fabric** | Plateforme unifiée (Lakehouse, Pipeline, Power BI) |
| **Delta Lake** | Format de stockage transactionnel |
| **PySpark** | Transformations de données distribuées |
| **SQL** | Requêtes déclaratives |
| **Power BI** | Visualisation et rapports |
| **DAX** | Mesures et calculs analytiques |
| **Python** | Scripting et automatisation |
| **Git/GitHub** | Versioning du code |

---

## 📊 Modèle de données (Star Schema)

### Tables de dimension

#### 📅 dim_date
- `date_id` (PK)
- `date`, `day`, `month`, `year`, `quarter`, `day_of_week`, etc.

#### 🕐 dim_time
- `time_id` (PK)
- `time`, `hour`, `minute`, `period` (Matin/Après-midi/Soir/Nuit)

#### 🌀 dim_turbine
- `turbine_id` (PK)
- `turbine_name`, `latitude`, `longitude`, `region`, `department`

#### 🔧 dim_operational_status
- `status_id` (PK)
- `operational_status`, `status_description`

### Table de faits

#### ⚡ fact_production
- `production_id` (PK)
- `date_id` (FK)
- `time_id` (FK)
- `turbine_id` (FK)
- `status_id` (FK)
- **Mesures** : `energy_kwh`, `wind_speed`, `wind_direction`

---

## 📈 Métriques et KPIs

Les rapports Power BI affichent :

- 📊 **Production totale** par turbine
- 📅 **Évolution temporelle** de la production
- 🗺️ **Carte géographique** des turbines
- 💨 **Analyse du vent** (vitesse, direction)
- 🔧 **Statut opérationnel** des turbines
- ⏰ **Analyse par période** (jour/nuit, saisons)

---

## 🎓 Compétences développées

- ✅ **Architecture Medallion** (Bronze/Silver/Gold)
- ✅ **Modélisation dimensionnelle** (Star Schema)
- ✅ **PySpark** pour transformations distribuées
- ✅ **SQL** pour requêtes analytiques
- ✅ **Orchestration** de pipelines de données
- ✅ **Power BI** pour visualisation
- ✅ **DAX** pour mesures calculées
- ✅ **Delta Lake** pour data lakes transactionnels
- ✅ **Git/GitHub** pour versioning

---

## 📚 Ressources

### Documentation officielle
- [Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/)
- [Delta Lake](https://delta.io/)
- [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)
- [Power BI](https://learn.microsoft.com/en-us/power-bi/)
- [DAX](https://dax.guide/)

### Communautés
- [Microsoft Fabric Community](https://community.fabric.microsoft.com/)
- [Power BI Community](https://community.powerbi.com/)
- [Stack Overflow - microsoft-fabric](https://stackoverflow.com/questions/tagged/microsoft-fabric)

---

## 👤 Auteur

**Jawad Berrhili**

📧 Contact : jawadberrhili@hotmail.fr
🐙 GitHub : [@jawadberrhili](https://github.com/djak75)

---

## 📝 Licence

Ce projet est réalisé dans un cadre pédagogique.

---

## 🙏 Remerciements

- Formation Microsoft Fabric
- Guillaume Soulat pour les données d'entraînement
- Benjamin notre super formateur
- Communauté Microsoft Fabric

---

*Dernière mise à jour : Février 2026*
