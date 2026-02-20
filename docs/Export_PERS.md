# Fiche Technique : Export_PERS

Ce job assure l'extraction et la mise en conformité des données du personnel de l'UHA pour le projet SEBINA.

## 1. Caractéristiques du Flux

- **Source principale** : Oracle 19c, vue `GRHUM.V_UHA_WSSIGB_PERS_EXPORT` ([voir SQL](../sql/views_pers.sql)).
- **Cible** : PostgreSQL, table `EXPORT_PERS`.
- **Identifiant pivot** : `supannEmpId`.
- **Fréquence** : 3 exécutions quotidiennes.

## 2. Logique d'enrichissement LDAP

Le job réalise une jointure **Inner Join** entre la vue Oracle et l'annuaire LDAP.

- **Critère de jointure** : `SUPANNEMPID` (Oracle) == `supannEmpId` (LDAP).
- **Données LDAP récupérées** : adresse mail institutionnelle, téléphones professionnel et mobile, login (`uid`), CSN de la carte PassCampus (`uhaPasscampusCsnDec`).

> **⚠️ Conséquence** : si un agent est absent du LDAP, il est exclu du flux et de la table cible en raison de la jointure stricte.

## 3. Détails du Hash (SHA-256)

Le hash est calculé sur une chaîne concaténant **39 champs**. Pour le mécanisme général, voir [hash-differentiel.md](hash-differentiel.md).

Segments inclus :

- État civil (nom, prénom, date de naissance, civilité)
- Adresses professionnelles et personnelles complètes
- Situation administrative (affectation, établissement)
- Attributs LDAP (mail, téléphones, login, CSN)

## 4. Règles Métier

- **Tarification** : forcée à `G` (Gratuit) pour l'ensemble du personnel.
- **Date de suppression** : calculée par la vue source à **+30 jours** après la fin de contrat, offrant un délai de grâce d'accès aux services.
