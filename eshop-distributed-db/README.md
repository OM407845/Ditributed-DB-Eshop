# Projet Fin module : Bases de données distribuées - Fragmentation, Procédures stockées et Optimisation

Ce dépôt contient l'implémentation du Travail Pratique (TP) sur les bases de données distribuées. Il met en œuvre la fragmentation de données, l'utilisation de procédures stockées pour la manipulation (DML) et l'optimisation des requêtes.

## 🎯 Objectifs du Projet

1. **Fragmentation d’une base de données relationnelle** en respectant un ensemble de critères de sélection.
2. **Utilisation des procédures stockées PL/SQL** pour les requêtes d’écriture SQL.
3. **Utilisation des index** pour optimiser les requêtes de lecture SQL.

## 📝 Énoncé des Scénarios

Soit la base de données relationnelle « EShop » (utilisée dans le TP1).

### 1er Scénario : Fragmentation Dérivée
Soient les requêtes de sélection les plus utilisées dans les deux sites Site1 et Site2 :
* **R1** = σ(idCategorie=50 AND quantite>100) (LigneCommandes)
* **R2** = σ(idCategorie=35 AND quantite>50) (LigneCommandes)

### 2ème Scénario : Fragmentation par Volume de Vente (Gros vs Détail)
On sépare les grosses commandes (Grossistes) des petites (Détail).
* **Logique :** Le Site 1 gère les stocks de gros volumes (Entrepôt central), le Site 2 gère les petits volumes (Magasins de proximité).
* **Table fragmentée :** `LigneCommandes`
* **Critères :**
  * **R1 (Site 1)** = σ(Quantite ≥ 100) (LigneCommandes)
  * **R2 (Site 2)** = σ(Quantite < 100) (LigneCommandes)

---

## 🛠️ Architecture du Projet

L'environnement repose sur Docker Compose pour simuler un réseau de bases de données distribuées :
* **oracle-main** : Base globale centralisant les tables de dimension et interceptant les DML (via triggers).
* **oracle-site1** : Base distante gérant le Fragment 1.
* **oracle-site2** : Base distante gérant le Fragment 2.

```text
eshop-distributed-db/
├── docker-compose.yml
├── RAPPORT_PROJET.md
└── scripts/
    ├── main/
    ├── Site1/
    └── Site2/
```

---

## 🚀 Tutoriel de Déploiement et d'Exécution

### Étape 1 : Démarrer l'environnement Docker
Dans le dossier `eshop-distributed-db`, lancez les conteneurs :
```bash
docker-compose up -d
```
*Patientez quelques minutes le temps que les 3 bases de données Oracle soient prêtes.*

### Étape 2 : Initialisation des Bases de Données
Connectez-vous à chaque base de données (via un client SQL comme SQL*Plus, DBeaver ou SQL Developer) et exécutez les scripts dans l'ordre suivant :

**Sur le serveur Main (`oracle-main`) :**
1. Exécutez `scripts/main/01_init_main.sql` (Création de l'utilisateur, tables globales, DB Links)
2. Exécutez `scripts/main/02_insert_data.sql` (Insertion du jeu de données initial)

**Sur le Site 1 (`oracle-site1`) :**
1. Exécutez `scripts/Site1/01_init_site1.sql` (Création de l'utilisateur et tables du fragment)
2. Exécutez `scripts/Site1/02_procedures_site1.sql` (Création des procédures de manipulation DML)

**Sur le Site 2 (`oracle-site2`) :**
1. Exécutez `scripts/Site2/01_init_site2.sql` (Création de l'utilisateur et tables du fragment)
2. Exécutez `scripts/Site2/02_procedures_site2.sql` (Création des procédures de manipulation DML)

---

## 🔄 Comment tester les Scénarios ?

La logique de routage distribué (fragmentation) est gérée par des **Triggers** déployés sur la base de données principale (Main).

### Le Scénario 2 est actif par défaut
Une fois les conteneurs Docker démarrés et les bases initialisées, **c'est le Scénario 2 (Fragmentation par Volume de Vente) qui est actif par défaut**.
Vous pouvez directement tester vos opérations :
* Un `INSERT` avec `Quantite >= 100` sera envoyé au Site 1.
* Un `INSERT` avec `Quantite < 100` sera envoyé au Site 2.
* Un `UPDATE` modifiant la quantité au-dessus ou en-dessous du seuil de 100 déclenchera la migration de la ligne d'un site à l'autre.

### Passer au Scénario 1
Pour basculer sur la fragmentation du Scénario 1, il vous suffit d'écraser les triggers actuels en exécutant le script du scénario 1 sur la base **Main** :
```sql
@scripts/main/04_scenario1_triggers.sql
```
*(Dans DBeaver/SQL Developer, ouvrez le fichier et exécutez tout le contenu).*
Après cela, les requêtes `INSERT`, `UPDATE` et `DELETE` sur la table `LIGNECOMMANDES` seront routées vers Site1 ou Site2 selon les règles de catégorie et quantité du Scénario 1.

---

## ⚡ Tester l'Optimisation des Requêtes

Pour vérifier l'impact des index et comparer les plans d'exécution (`EXPLAIN PLAN`), connectez-vous à la base **Main** et exécutez :
```sql
@scripts/main/06_queries_and_optimization.sql
```
Vous y trouverez les requêtes analytiques avec et sans utilisation d'un index basé sur une fonction, ainsi que les commandes pour afficher les statistiques d'exécution.
