# 🥈 Silver Layer - Nettoyage et enrichissement

## 📋 Objectif

Cette couche transforme les données brutes de Bronze en données **nettoyées, standardisées et enrichies**. C'est ici qu'on applique les règles de qualité et qu'on ajoute des colonnes calculées.

## 📂 Notebooks

### `bronze_to_silver_pyspark.ipynb`
**Objectif** : Transformer Bronze → Silver avec PySpark

**Transformations appliquées** :
- ✅ **Nettoyage** : Gestion des valeurs nulles
- ✅ **Standardisation** : Correction du format `time` (`:` au lieu de `-`)
- ✅ **Arrondi** : Valeurs numériques à 2 décimales
- ✅ **Enrichissement** : Ajout de colonnes calculées
  - `date` : Date extraite de datetime
  - `time` : Heure extraite de datetime
  - `day`, `month`, `year` : Composantes de la date
  - `period` : Période de la journée (Matin/Après-midi/Soir/Nuit)

**Technologies** : PySpark, Delta Lake

---

### `bronze_to_silver_sql.ipynb` *(optionnel)*
**Objectif** : Version SQL de la transformation Bronze → Silver

**Avantage** : Syntaxe SQL déclarative, plus simple pour certains utilisateurs

**Technologies** : Spark SQL, Delta Lake

## 🔄 Logique de transformation

```python
# Pseudo-code
1. Lire la table Bronze
2. Extraire date et time depuis datetime
3. Créer les colonnes day, month, year
4. Calculer la période (Matin: 6-12h, Après-midi: 12-18h, etc.)
5. Corriger le format time (remplacer '-' par ':')
6. Arrondir wind_speed, wind_direction, energy_kwh à 2 décimales
7. Gérer les valeurs nulles
8. Écrire dans la table Silver (Delta)
```

## 📊 Table Silver

**Nom** : `silver_eolienne_production`

**Format** : Delta Lake

**Mode d'écriture** : `overwrite` (écrasement complet à chaque exécution)

**Nouvelles colonnes ajoutées** :
| Colonne | Type | Description |
|---------|------|-------------|
| `date` | date | Date de production |
| `time` | string | Heure de production (format HH:MM) |
| `day` | int | Jour du mois (1-31) |
| `month` | int | Mois (1-12) |
| `year` | int | Année |
| `period` | string | Matin/Après-midi/Soir/Nuit |

**Colonnes conservées** (avec nettoyage) :
- `turbine_name`, `latitude`, `longitude`
- `region`, `department`
- `wind_speed` (arrondi à 2 décimales)
- `wind_direction` (arrondi à 2 décimales)
- `energy_kwh` (arrondi à 2 décimales)
- `operational_status`

## 🎯 Règles métier

### Calcul de la période
```
Matin       : 06:00 - 11:59
Après-midi  : 12:00 - 17:59
Soir        : 18:00 - 21:59
Nuit        : 22:00 - 05:59
```

### Correction du format time
```
Bronze : "14-30"  →  Silver : "14:30"
```

## 🚀 Exécution

Ce notebook sera exécuté par la **Data Pipeline** en deuxième étape, après l'ingestion Bronze.

## ⚠️ Points d'attention

- Vérifier les valeurs aberrantes (wind_speed négatif, etc.)
- Gérer les fuseaux horaires si nécessaire
- Documenter les règles de calcul de `period`

---

**Étape précédente** : [Bronze Layer](../bronze/README.md)
**Prochaine étape** : [Gold Layer](../gold/README.md)
