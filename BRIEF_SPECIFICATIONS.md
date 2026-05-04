# ERP Achat Vente Stock (AVS)

ERP web complet pour la gestion integree des achats, ventes, stocks et inventaires. Le systeme centralise les flux, assure une tracabilite de bout en bout, applique des workflows d'approbation multi-niveaux, et expose des tableaux de bord par role pour le pilotage.

---

## 1. Contexte et objectif metier

Les organisations multi-sites font face a des processus fragmentes entre achats, ventes, stock et finance. AVS unifie l'ensemble de la chaine operationnelle afin de :
- reduire les erreurs et les ecarts d'inventaire,
- accelerer les delais de traitement,
- renforcer le controle interne (separation des taches),
- fournir des KPI exploitables par role, site et depot.

---

## 2. Perimetre fonctionnel

### 2.1 Referentiels
- Articles, familles, fournisseurs, clients
- Unites de mesure, taxes, devises
- Methodes de valorisation (FIFO, CUMP)
- Parametrage peremption / lots / series
- Tarifs multi-devises et validites

### 2.2 Achats
- Demande d'achat (DA) avec priorite et workflow
- Approbation multi-niveaux (N1/N2/N3)
- Verification budgetaire finance
- Pro-forma et comparaison fournisseurs
- Bon de commande (BC) et validation legale
- Receptions partielles, controle qualite
- Factures fournisseurs et 3-way match
- Paiements et suivi des echeances

### 2.3 Ventes
- Devis / pro-forma client
- Commandes clients et reservation stock
- Preparation, livraison, facturation
- Avoirs clients avec double validation
- Encaissements multi-modes

### 2.4 Stock
- Lots / series, statut qualite
- Mouvements entree / sortie / transfert / ajustement
- Reservation et allocation FIFO/FEFO
- Stock par depot / emplacement / lot
- Historique non modifiable

### 2.5 Inventaires
- Sessions annuelles, tournantes ou ponctuelles
- Saisie de comptage, recomptage
- Calcul des ecarts et ajustements valides
- Gel optionnel des mouvements

### 2.6 Administration et securite
- RBAC et perimetres d'acces (ABAC)
- Plafonds d'approbation et de remise
- Delegations temporaires
- Audit complet et non modifiable

---

## 3. Logique metier et regles critiques

- Separation des taches :
  - Createur DA != approbateur DA
  - Createur BC != approbateur BC
  - Receptionnaire != validateur facture
  - Createur avoir != validateur != encaisseur
  - Compteur inventaire != validateur ajustement
- Validation budgetaire obligatoire avant engagement BC
- 3-way match obligatoire avant validation facture fournisseur
- Remises plafonnees par role et par utilisateur
- Lots perimes bloques automatiquement
- Stock disponible = stock physique - stock reserve
- Historique de mouvements non modifiable
- Numerotation automatique non reutilisable

---

## 4. Architecture technique

- Backend : Spring Boot (Java 17 ou 11 minimum)
- Base de donnees : PostgreSQL 12+
- Frontend : templates HTML cote serveur
- Organisation en domaines metier : referentiels, achats, ventes, stock, inventaires, admin

Flux principaux :
- Achats : DA -> approbations -> BC -> reception -> facture -> paiement
- Ventes : devis -> commande -> livraison -> facture -> encaissement
- Stock : mouvements liees aux documents et aux utilisateurs
- Inventaires : session -> comptage -> ecarts -> ajustements

## 6. KPI et tableaux de bord

- Direction generale : CA, marge, rotation stock, obsolescence, ecarts inventaire
- Achats : cycle DA -> BC, OTD fournisseurs, ecarts facture
- Stock : precision, peremption, productivite reception / picking
- Ventes : backlog, retards, remises, avoirs
- Finance : factures bloquees, valeur stock comptable vs operationnelle