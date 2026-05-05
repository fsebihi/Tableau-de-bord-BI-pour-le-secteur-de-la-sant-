# 🏥 Pilotage de la Performance Hospitalière & Parcours de Soins

> **Dashboard Power BI** — Réseau hospitalier de 8 établissements | +10 000 patients | 3 ans d'historique (2022–2024)

---

## 📌 Contexte du Projet

Le système de santé français, piloté par le Ministère de la Santé et les Agences Régionales de Santé (ARS), doit surveiller en continu l'activité hospitalière, analyser les coûts et anticiper les besoins futurs en ressources.

Ce projet vise à **remplacer un reporting manuel chronophage** par une solution Power BI complète permettant aux directeurs d'établissement de piloter leur réseau en temps réel via des dashboards interactifs.

---

## 🎯 Objectifs

- Suivre l'activité hospitalière par hôpital, région et période
- Analyser les pathologies traitées et leur évolution
- Mesurer la performance financière et clinique des établissements
- Identifier les tensions de capacité (taux d'occupation)
- Aider à la planification future du système de santé

---

## 🗂️ Architecture des Données — Schéma en Étoile

```
                    dim_temps
                        |
dim_pathologies ── fait_sejours ── dim_hopital
                        |
                   dim_patients
```

| Table | Type | Colonnes clés |
|-------|------|---------------|
| `fait_sejours` | **Faits** | id_patient, id_hopital, id_pathologie, id_temps, cout_sejour, cout_medicaments, duree_sejour, mode_entree, mode_sortie, statut_sejour, nombre_actes |
| `dim_hopital` | Dimension | nom_hopital, type_etablissement, region, nombre_lits, taux_occupation_cible, latitude, longitude |
| `dim_patients` | Dimension | genre, groupe_age, mutuelle, ville |
| `dim_pathologies` | Dimension | libelle_pathologie, famille_pathologie, niveau_gravite, cout_moyen_reference, duree_moy_reference |
| `dim_temps` | Dimension | annee, mois, mois_libelle, trimestre_libelle, date_complete |

---

## 🔧 Stack Technique

| Outil | Usage |
|-------|-------|
| **Power BI Desktop** | Développement des dashboards |
| **Power Query (M)** | Nettoyage et transformation des données |
| **DAX** | Mesures avancées (Time Intelligence, KPIs cliniques) |
| **Modélisation en étoile** | Architecture décisionnelle |

---

## 📊 Les 4 Vues Analytiques

### 💰 Vue Financière
> Pilotage des coûts du réseau hospitalier

**KPIs :** Coût Actuel · Coût N-1 · YoY Coût · CMS Actuel · CMS N-1 · YoY CMS · Écart CMS vs CMR

**Visuels :**
- Évolution mensuelle des coûts N vs N-1 (line chart)
- Répartition des coûts par type d'établissement (bar chart)
- Performance par établissement — tableau avec Nb Hospitalisations, Coût Total, CMS, Taux Occupation
- Répartition des coûts par service médical (bar chart horizontal)

**Insight clé :** En 2024, le réseau enregistre une hausse des coûts de **+4,34% vs N-1** avec un CMS de **8,92K€**. L'Hôpital de proximité de Muret affiche un taux d'occupation de **132,73%** — signal d'alerte sur la surcharge capacitaire.

---

### 🏥 Vue Pathologies
> Analyse épidémiologique et performance clinique

**KPIs :** Nb Hospitalisations · YoY Hospitalisations · Taux Réadmission · Taux Mortalité · DMS · CMS

**Visuels :**
- Évolution mensuelle des hospitalisations N vs N-1
- Top pathologies par volume de séjours
- DMS par pathologie (Leucémie aiguë : 19j, AVC hémorragique : 18j)
- Répartition par niveau de gravité (donut : 34,95% niveau Élevé)

**Insight clé :** **58,49%** de taux de réadmission et **4,97%** de taux de mortalité — indicateurs construits à partir de `mode_sortie` et de l'analyse des séjours multiples par patient.

---

### 👤 Vue Patients
> Analyse démographique et parcours de soins

**KPIs :** Nb Patients · Nb Patients N-1 · YoY Patients · DMS · DMS N-1 · YoY DMS

**Visuels :**
- Évolution mensuelle du nombre de patients N vs N-1
- Répartition par groupe d'âge (18-40 ans = groupe majoritaire : 2 175 patients)
- Répartition par genre (50,59% F / 49,41% H)
- DMS par groupe d'âge (75+ = 10,6 jours vs 7,9 jours pour 41-60)

**Insight clé :** Les patients **75 ans et plus** ont une DMS **34% supérieure** à la moyenne — signal fort pour la planification des ressources gériatriques.

---

### 🗺️ Vue Carte
> Géolocalisation et vision réseau

**KPIs :** Nb Établissements · Nb Séjours · Nb Patients · Taux Occupation

**Visuels :**
- Carte interactive avec bulles proportionnelles au volume de patients
- Couleurs par type d'établissement (CHU, CHR, Clinique privée, ESPIC, Hôpital de proximité)
- Filtres : Région · Trimestre · Type d'établissement

**Insight clé :** Taux d'occupation réseau de **75,13%** — en dessous de l'objectif de 85%, avec des disparités géographiques importantes entre établissements.

---

## ⚙️ Mesures DAX Clés

### Time Intelligence — Comparaison N vs N-1
```dax
Coût Année Précédente = 
CALCULATE(
    [Coût Total],
    SAMEPERIODLASTYEAR(dim_temps[date_complete])
)

Évolution YoY % = 
VAR Evolution = DIVIDE([Coût Total] - [Coût Année Précédente], [Coût Année Précédente])
RETURN
    IF(Evolution > 0, "▲ " & FORMAT(Evolution, "0.00%"), "▼ " & FORMAT(ABS(Evolution), "0.00%"))
```

### Taux de Mortalité
```dax
Taux Mortalité = 
DIVIDE(
    CALCULATE(COUNTROWS(fait_sejours), fait_sejours[mode_sortie] = "Décès"),
    COUNTROWS(fait_sejours)
)
```

### Taux de Réadmission
```dax
Taux Réadmission = 
VAR PatientsReadmis = 
    CALCULATE(
        DISTINCTCOUNT(fait_sejours[id_patient]),
        FILTER(VALUES(fait_sejours[id_patient]), CALCULATE(COUNTROWS(fait_sejours)) > 1)
    )
RETURN DIVIDE(PatientsReadmis, [Nombre Patients])
```

### Taux d'Occupation
```dax
Taux Occupation réel % = 
DIVIDE(
    SUMX(fait_sejours, fait_sejours[duree_sejour]),
    SUMX(dim_hopital, dim_hopital[nombre_lits]) * 365
)
```

### Couleurs Conditionnelles Dynamiques
```dax
Couleur YoY Coût = 
VAR Evolution = DIVIDE([Coût Total] - [Coût Année Précédente], [Coût Année Précédente])
RETURN IF(Evolution > 0, "#A32D2D", "#3B6D11")
```

### Storytelling Dynamique
```dax
Storytelling Financier = 
"En " & SELECTEDVALUE(dim_temps[annee]) & 
" le réseau enregistre une hausse des coûts de " & [Évolution YoY %] & 
" vs N-1. Taux d'occupation : " & FORMAT([Taux Occupation réel %], "0.0%") & 
IF([Taux Occupation réel %] < 0.85, 
   " — Taux d'occupation en dessous de l'objectif. ⚠️ Signal d'alerte sur la capacité.", 
   " — Taux d'occupation dans l'objectif. ✅")
```

---

## 📈 Résultats & Chiffres Clés (2024)

| Indicateur | Valeur |
|------------|--------|
| Coût total réseau | 139,97 M€ |
| Évolution YoY coûts | ▲ +4,34% |
| Coût Moyen Séjour | 8,92 K€ |
| Nb hospitalisations | 15 684 |
| Nb patients uniques | 7 930 |
| DMS moyenne | 8,48 jours |
| Taux réadmission | 58,49% |
| Taux mortalité | 4,97% |
| Taux occupation réseau | 75,13% |
| Pathologie la plus lourde (DMS) | Leucémie aiguë lymphoblastique — 19 jours |
| Service le plus coûteux | Oncologie — 40 M€ |

---

## 🗃️ Structure du Repository

```
Healthcare-BI-Dashboard/
│
├── Source/
│   ├── fait_sejours.csv
│   ├── dim_hopital.csv
│   ├── dim_patients.csv
│   ├── dim_pathologies.csv
│   └── dim_temps.csv
│
│── Screen/
│   ├── Vue Financiére.png
│   ├── Vue Pathologie.png
│   ├── Vue Patients.png
│   ├── Vue Carte.png
│
├── dashboard/
│   └── HEALTHCARE.pbix
│
├── docs/
│   └── HEALTHCARE.pdf
│
└── README.md
```

---

## 🚀 Comment utiliser ce projet

1. **Cloner le repository**
```bash
git clone https://github.com/Tarek-KHAZEM/Healthcare-BI-Dashboard.git
```

2. **Ouvrir le fichier** `dashboard/HEALTHCARE.pbix` avec Power BI Desktop

3. **Actualiser les sources** si nécessaire en pointant vers le dossier `Source/`

4. **Explorer les 4 vues** via la navigation en haut du dashboard

---

## 👤 Auteur

**Tarek KHAZEM**  
MS Business Intelligence & Analytics — CY Tech Cergy  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Tarek_Khazem-blue?logo=linkedin)](https://www.linkedin.com/in/tarek-khazem-43901139a/)

---

## 📄 Licence

Projet réalisé dans un cadre académique — MS BI & Analytics, CY Tech Cergy.  
Les données sont synthétiques et ne représentent aucun patient réel.
