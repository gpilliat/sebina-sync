# Fiche Technique : Export_ETU

Ce job assure l'extraction et la mise en conformité des données de scolarité des étudiants de l'UHA pour le projet SEBINA.

## 1. Caractéristiques du Flux

- **Source principale** : Oracle 19c, vue `PASSCAMPUS.V_UHA_WSSIGB_ETU_EXPORT` ([voir SQL](../sql/views_etu.sql)).
- **Cible** : PostgreSQL, table `EXPORT_ETU`.
- **Identifiant pivot** : `ETU_NUMBER` (code étudiant).
- **Fréquence** : 3 exécutions quotidiennes.

## 2. Logique d'enrichissement LDAP

Comme pour le flux personnels, une jointure **Inner Join** est effectuée avec l'annuaire LDAP.

- **Critère de jointure** : `ETU_NUMBER` (Oracle) == `supannEtuId` (LDAP).
- **Données LDAP récupérées** : mail institutionnel et identifiants techniques nécessaires à l'authentification.

> **⚠️ Conséquence** : même comportement que pour les personnels — un étudiant absent du LDAP est exclu du flux.

## 3. Détails du Hash (SHA-256)

Le hash est calculé sur une chaîne concaténant **45 champs**, soit un périmètre plus large que pour les personnels afin de capter les changements de diplôme ou d'étape en cours d'année. Pour le mécanisme général, voir [hash-differentiel.md](hash-differentiel.md).

Segments inclus :

- État civil (nom, prénom, date de naissance, civilité)
- Adresse annuelle (scolarité) et adresse permanente
- Identifiants (INE, ID Opaque)
- Détails complets du cursus universitaire (diplôme, niveau, UFR, discipline SISE)

## 4. Règles Métier

- **Niveaux académiques** : mapping des codes niveaux vers des valeurs numériques (`L1` → `1`, `M1` → `4`, `D1` → `6`, etc.).
- **Cycles de formation** : extraction du cycle (`L` → `1`, `M` → `2`, `D` → `3`).
- **Tarification** : fixée à `DU` (Droits Universitaires).
