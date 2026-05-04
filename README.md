# ERP Achat Vente Stock (AVS)

Application web ERP pour la gestion integree des achats, ventes, stocks et inventaires, avec workflows d'approbation, gestion multi-sites et tableaux de bord.

## Prerequis

- Java 17 (ou 11 minimum)
- Maven 3.8+
- PostgreSQL 12+

## Configuration base de donnees

1. Creer la base de donnees `avs_db` dans PostgreSQL.
2. Verifier les identifiants dans [src/main/resources/application.properties](src/main/resources/application.properties).
   - `spring.datasource.url=jdbc:postgresql://localhost:5432/avs_db`
   - `spring.datasource.username=postgres`
   - `spring.datasource.password=root`
3. Charger le schema et les donnees de test :
   - Schema: [database/new_database.sql](database/new_database.sql)
   - Donnees: [database/new_data.sql](database/new_data.sql)

## Lancer l'application

1. Compiler le projet :
   - `mvn clean package`
2. Demarrer l'application (au choix) :
   - `mvn spring-boot:run`
   - ou `java -jar target/avs-0.0.1-SNAPSHOT.war`
3. Ouvrir l'application :
   - http://localhost:8080/login

## Comptes de test (par defaut)

- admin / admin (acces total)

## Structure fonctionnelle (resume)

- Referentiels (articles, familles, fournisseurs, clients, tarifs)
- Achats (DA, BC, receptions, factures, paiements)
- Ventes (devis, commandes, livraisons, factures, encaissements)
- Stock (lots, mouvements, transferts, reservations)
- Inventaires (sessions, comptages, ajustements)
- Administration et securite (utilisateurs, roles, audit)