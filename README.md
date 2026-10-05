# Outil d’analyse et d’évaluation de CV

Projet réalisé dans le cadre d’un stage au sein d’un cabinet de conseil spécialisé SAP.

L’objectif est de développer un outil permettant d’effectuer un **premier tri de candidatures** en analysant automatiquement un CV et en le comparant à une fiche de poste paramétrable.

Le projet repose sur un système de **parsing et de scoring basé sur des règles métier explicables**, sans recours à l’intelligence artificielle.

---

## Objectifs

L’application permet de :

- importer un CV au format PDF, DOC ou DOCX ;
- extraire automatiquement plusieurs informations du candidat ;
- comparer le profil avec une fiche de poste ;
- détecter les compétences présentes et manquantes ;
- estimer l’expérience professionnelle ;
- détecter le niveau d’études ;
- calculer un score de pertinence sur 100 ;
- attribuer un verdict RH ;
- afficher les résultats dans une interface web.

Le but est d’aider le recruteur lors du premier tri des candidatures.  
Le score constitue une **aide à la décision** et ne remplace pas la décision finale du recruteur.

---

## Technologies utilisées

- Python
- Flask
- pdfplumber
- Expressions régulières (`re`)
- JSON
- HTML
- CSS
- LibreOffice pour la conversion DOC/DOCX → PDF

---

## Structure du projet

```text
cv-ranking-ui/
│
├── app.py
│
├── backend/
│   ├── parser.py
│   └── scoring.py
│
├── templates/
│   ├── index.html
│   ├── result.html
│   └── details.html
│
├── static/
│   └── style.css
│
├── data/
│   ├── cvs/
│   └── offre.json
│
└── README.md
```

### `app.py`

Fichier principal de l’application.

Il permet notamment de :

- lancer le serveur Flask ;
- gérer l’import des CV ;
- appeler le module de parsing ;
- appeler le module de scoring ;
- transmettre les résultats aux différentes pages HTML.

### `backend/parser.py`

Ce module analyse le contenu du CV afin d’en extraire les informations utiles.

Il permet notamment de détecter :

- le nom et le prénom ;
- l’adresse e-mail ;
- le numéro de téléphone ;
- le poste ou titre du profil ;
- les compétences techniques ;
- les soft skills ;
- le niveau d’études ;
- l’expérience professionnelle estimée ;
- les langues ;
- la localisation.

L’analyse repose principalement sur des **expressions régulières**, des recherches de mots-clés et différentes règles de détection.

### `backend/scoring.py`

Ce module compare les informations extraites du CV avec les critères définis dans la fiche de poste.

Il calcule les différents blocs du score et produit un résultat global sur 100.

### `data/offre.json`

La fiche de poste constitue le référentiel utilisé pour évaluer le CV.

Elle est stockée dans un fichier JSON afin de pouvoir être modifiée **sans changer le code Python**.

Exemple simplifié :

```json
{
  "poste": "Consultant SAP BC S/4HANA",
  "niveau": "confirmé",
  "experience_min": 5,
  "niveau_etude_min": "bac+5",

  "competences": {
    "obligatoires": [],
    "appreciees": [],
    "bonus": []
  },

  "soft_skills": [],

  "ponderation": {
    "obligatoires": 50,
    "appreciees": 20,
    "bonus": 10,
    "experience": 15,
    "formation": 5
  }
}
```

La fiche de poste peut contenir :

- les compétences obligatoires ;
- les compétences appréciées ;
- les compétences bonus ;
- les soft skills ;
- l’expérience minimale ;
- le niveau d’études minimum ;
- les langues demandées ;
- les pondérations utilisées pour le scoring.

Le système permet également de regrouper différentes écritures d’une même compétence afin d’éviter les doublons, par exemple :

```text
S/4HANA
S4HANA
S/4 hana
```

---

## Système de scoring

Le scoring est volontairement simple et explicable.

Exemple de répartition :

| Critère | Score maximum |
|---|---:|
| Compétences obligatoires | 50 |
| Compétences appréciées | 20 |
| Compétences bonus | 10 |
| Expérience | 15 |
| Formation | 5 |
| **Total** | **100** |

Les compétences obligatoires ont le poids le plus important.

Une compétence bonus ne doit pas permettre de compenser complètement l’absence d’une compétence obligatoire.

---

## Verdict

À partir du score obtenu, un verdict est attribué :

| Score | Verdict |
|---|---|
| ≥ 75 | Profil pertinent |
| 55 à 74 | À étudier |
| < 55 | Peu pertinent |

Ce verdict reste uniquement une aide au premier tri.

---

## Interface web

L’application dispose de trois pages principales.

### `index.html`

Permet à l’utilisateur de déposer un CV.

Les formats acceptés sont :

- PDF
- DOC
- DOCX

Un message d’erreur est affiché lorsqu’un format non autorisé est envoyé.

### `result.html`

Affiche notamment :

- le score global ;
- le verdict ;
- un résumé du scoring ;
- les principaux éléments détectés.

### `details.html`

Permet de consulter plus précisément les informations extraites du CV :

- compétences trouvées ;
- compétences manquantes ;
- soft skills ;
- expérience ;
- formation ;
- langues ;
- localisation ;
- autres informations détectées.

---

# Installation

## Prérequis

### macOS

- Python 3.x
- pip
- Flask
- pdfplumber
- LibreOffice
- environnement virtuel Python recommandé

### Windows

- Python 3.x
- pip
- Flask
- pdfplumber
- LibreOffice
- environnement virtuel Python recommandé

Lors de l’installation de Python sous Windows, il est recommandé de cocher :

```text
Add Python to PATH
```

---

## Installation des dépendances

Créer un environnement virtuel :

```bash
python -m venv .venv
```

### macOS / Linux

```bash
source .venv/bin/activate
```

### Windows – Invite de commandes

```bash
.venv\Scripts\activate
```

### Windows – PowerShell

```powershell
.venv\Scripts\Activate.ps1
```

Installer ensuite les dépendances :

```bash
pip install flask pdfplumber
```

---

## Lancement de l’application

Dans le dossier du projet :

```bash
python app.py
```

Sous Windows, il peut également être nécessaire d’utiliser :

```bash
py app.py
```

Une fois le serveur lancé, ouvrir l’adresse affichée dans le terminal, généralement :

```text
http://127.0.0.1:5000
```

---

## Conversion des fichiers DOC / DOCX

Les fichiers PDF peuvent être analysés directement.

Pour les fichiers DOC et DOCX, l’application utilise **LibreOffice en mode headless** afin de les convertir automatiquement en PDF avant l’analyse.

LibreOffice doit donc être installé sur l’ordinateur pour utiliser cette fonctionnalité.

---

## Fonctionnement général

```text
CV
 │
 ▼
Extraction du texte
 │
 ▼
Parsing
 │
 ├── Identité
 ├── Compétences
 ├── Expérience
 ├── Formation
 ├── Langues
 └── Localisation
 │
 ▼
Comparaison avec offre.json
 │
 ▼
Scoring
 │
 ▼
Verdict
 │
 ▼
Affichage dans l’interface web
```

---

## Première approche du projet

Avant le début du stage, une première version expérimentale du projet avait été développée en utilisant des techniques de **traitement automatique du langage naturel (NLP)** afin de comparer plusieurs CV avec une offre et de calculer leur pertinence.

Cette première approche utilisait notamment Python pour l’analyse textuelle et C++ pour l’exécution du script, la récupération des scores et le classement des CV.

Après échange avec l’entreprise, une seconde approche a été retenue pour le projet final : un système basé sur des **règles métier, du parsing et un scoring explicable**, mieux adapté aux besoins du recruteur.

---

## Limites actuelles

Le projet reste un prototype.

Certaines limites sont notamment liées à :

- la diversité des mises en page des CV ;
- la détection automatique des noms et titres ;
- l’estimation de certaines périodes d’expérience ;
- les différentes formulations possibles d’une même compétence ;
- la qualité du texte extrait depuis certains PDF.

---

## Améliorations possibles

Plusieurs évolutions peuvent être envisagées :

- gérer plusieurs fiches de poste ;
- permettre de modifier la fiche de poste directement depuis l’interface ;
- ajouter une base de données ;
- conserver un historique des candidats analysés ;
- améliorer la gestion des synonymes ;
- améliorer la détection des informations personnelles ;
- permettre l’analyse de plusieurs CV simultanément ;
- améliorer l’interface utilisateur ;
- générer automatiquement un rapport de synthèse.

---

## Contexte

Ce projet a été réalisé dans le cadre d’un stage étudiant au sein de **Dinexus Conseil**, cabinets spécialisés dans le domaine SAP.

Il avait pour objectif de répondre à un besoin concret : faciliter le premier tri des candidatures tout en conservant un fonctionnement simple, transparent et justifiable.
