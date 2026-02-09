# 🥉 Bronze Layer - Ingestion des données brutes

## 📋 Objectif

Cette couche est responsable de l'**ingestion des données brutes** depuis la source (GitHub) vers le Lakehouse Bronze. Les données sont stockées au format **Delta Lake** sans transformation.

## 📂 Notebooks

### `ingestion_bronze.ipynb`
**Objectif** : Charger les fichiers CSV depuis GitHub vers la table Bronze

**Fonctionnalités** :
- ✅ Lecture des fichiers CSV depuis l'URL GitHub
- ✅ Ingestion incrémentale (ne pas recharger les données déjà présentes)
- ✅ Écriture au format Delta Lake
- ✅ Gestion des erreurs

**Colonnes chargées** :
- `datetime` : Date et heure de production
- `turbine_name` : Nom de la turbine
- `latitude`, `longitude` : Coordonnées GPS
- `region`, `department` : Localisation administrative
- `wind_speed`, `wind_direction` : Conditions de vent
- `energy_kwh` : Production d'énergie
- `operational_status` : Statut de la turbine

## 🔄 Logique d'ingestion

```python
# Pseudo-code
1. Lire le CSV depuis GitHub
2. Vérifier si des données existent déjà dans Bronze
3. Si oui : charger uniquement les nouvelles lignes (delta)
4. Si non : charger toutes les données (initial load)
5. Écrire dans la table Bronze au format Delta
```

## 📊 Table Bronze

**Nom** : `bronze_eolienne_production`

**Format** : Delta Lake

**Mode d'écriture** : `append` (ajout incrémental)

**Particularités** :
- Aucune transformation appliquée
- Conservation du format et structure d'origine
- Données brutes telles quelles

## 🚀 Exécution

Ce notebook sera exécuté par la **Data Pipeline** en première étape.

## ⚠️ Points d'attention

- Vérifier la connectivité à GitHub
- Gérer les timeouts potentiels
- Vérifier les duplicatas
- Logger les erreurs d'ingestion

---

**Prochaine étape** : [Silver Layer](../silver/README.md)
