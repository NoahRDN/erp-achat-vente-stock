# Specifications fonctionnelles et techniques — erp-achat-vente-stock

## 1. Objectif de l'application

L'application erp-achat-vente-stock est un ERP web qui centralise l'ensemble des flux Achats, Ventes, Stock et Inventaires. Elle vise a unifier le cycle complet depuis l'expression du besoin (demande d'achat) jusqu'au paiement fournisseur, et depuis le devis client jusqu'a l'encaissement. Le systeme apporte une tracabilite complete des mouvements, une gouvernance par workflows d'approbation multi-niveaux, et des tableaux de bord par role pour le pilotage.

Le perimetre est multi-entites, multi-sites et multi-depots. Une organisation est structuree par groupe, societes, sites et depots. Chaque site regroupe plusieurs depots et chaque depot contient des emplacements. L'application doit garantir la separation des taches et le controle interne sur les etapes critiques (approbations, validations, ajustements).

## 2. Profils utilisateurs

L'application cible des profils metier distincts avec des droits et des responsabilites explicites. Les profils et leurs attentes sont les suivants :

- Direction generale (DG) : supervision globale, approbations haut niveau, lecture de KPI globaux, validation d'ecarts d'inventaire majeurs.
- Direction financiere (DAF/Finance) : verification budgetaire, validation factures et paiements, suivi des rapprochements et des ecarts.
- Responsables achats : creation et validation des DA et BC selon seuils, negotiation fournisseurs, suivi des receptions.
- Operateurs achats : creation des demandes et des commandes, sans auto-approbation.
- Responsable stock / magasin : gestion des receptions, mouvements, transferts et inventaires, pilotage des ecarts.
- Magasiniers / operateurs stock : execution des mouvements, comptages, preparation des commandes.
- Commerciaux / responsables ventes : devis, commandes, remises, livraison, facturation et suivi encaissements.
- Administrateurs : parametrage, gestion des utilisateurs, roles, permissions et audit.

Chaque profil est contraint par des regles de separation des taches et des plafonds d'autorisation, notamment sur les approbations et les remises.

## 3. Architecture generale

L'application est une application web Spring Boot avec base de donnees PostgreSQL. Les ecrans front-end sont rendus via templates. Le noyau fonctionnel est organise par domaines metier : referentiels, achats, ventes, stock, inventaires, administration et securite.

Les entites principales incluent :

- Organisation : Groupe, Societe, Site, Depot, Emplacement.
- Referentiels : Article, FamilleArticle, Fournisseur, Client, Unite de mesure, Taxe, Devise.
- Achats : DemandeAchat, BonCommande, BonReception, FactureFournisseur, Paiement.
- Ventes : Devis, CommandeClient, BonLivraison, FactureClient, Encaissement, Avoir.
- Stock : Stock, Lot, MouvementStock, ReservationStock, Transfert.
- Inventaires : Inventaire, LigneInventaire, SaisieInventaire.
- Securite : Utilisateur, Role, Audit.

Le systeme doit maintenir une tracabilite de bout en bout et lier chaque mouvement a un document origine et un utilisateur.

## 4. Tableau de bord

Le tableau de bord est oriente pilotage par role et doit presenter des indicateurs operationnels et financiers. Les KPI incluent :

- Direction generale : chiffre d'affaires, marge, valeur stock globale, rotation, obsolescence, ecarts d'inventaire.
- Achats / Supply Chain : delais DA vers BC, respect des delais fournisseurs, taux de receptions conformes, evolution des prix d'achat.
- Stock / Magasin : precision stock, lots perimes, productivite de preparation, temps de traitement reception.
- Ventes : commandes en cours, retards, annulations, remises accordees, backlog non servi.
- Finance : factures bloquees par mismatch, valeur stock comptable, variation de marge.

Chaque KPI doit etre calculable par site, depot et periode afin de permettre un pilotage local et global.

## 5. Organisation metier principale

L'organisation metier est structuree autour des entites Groupe, Societe, Site et Depot. Un groupe possede plusieurs societes, chaque societe possede plusieurs sites et chaque site possede plusieurs depots. Les depots contiennent des emplacements physiques ou sont stockes les articles et lots.

Les utilisateurs sont rattaches a une organisation et un perimetre (site, depot, departement). Les droits d'acces et les workflows sont conditionnes par ce perimetre afin d'assurer la segregation des responsabilites.

## 6. Modules principaux

### 6.1 Referentiels

Le module referentiels permet de definir les donnees de base : articles, familles, fournisseurs, clients, unites de mesure, taxes et devises. Il doit assurer la coherence des informations de stock et de facturation.

Fonctionnalites attendues :

- Creation, modification et desactivation des articles et familles.
- Parametrage des methodes de valorisation par famille (FIFO, CUMP) et des regles de lot/peremption.
- Gestion des fournisseurs avec conditions de paiement, historique et qualite.
- Gestion des clients avec plafond de credit et remises.
- Gestion des tarifs multi-devises et validites.

Regles metier :

- Une famille d'articles peut imposer un lot obligatoire ou des dates de peremption.
- Les articles desactives ne peuvent pas etre commandes, vendus ou transferes.
- Les plafonds de credit client bloquent la validation des commandes au-dela du seuil.

Interactions : les referentiels sont utilises par les modules achats, ventes, stock et inventaires.

### 6.2 Achats

Le module achats couvre le processus complet DA vers paiement. Il garantit l'approbation multi-niveaux, la verification budgetaire et la tracabilite des documents.

Fonctionnalites attendues :

- Demande d'achat (DA) avec priorite, statut et workflow.
- Approbations multi-niveaux N1/N2/N3 selon seuils de montant.
- Verification budgetaire par la finance.
- Pro-forma fournisseur et comparaison multi-fournisseurs.
- Bon de commande fournisseur (BC) avec workflow d'approbation legale.
- Receptions partielles ou totales, controle qualite et generation des mouvements d'entree.
- Factures fournisseur avec rapprochement 3-way match (BC, reception, facture).
- Paiements fournisseurs avec suivi des echeances.

Regles metier :

- Le createur d'une DA ou d'un BC ne peut pas etre l'approbateur final.
- La validation budgetaire est obligatoire avant engagement d'un BC.
- Les receptions partielles mettent a jour le reste a recevoir.
- Une facture fournisseur ne peut etre validee si un ecart majeur est detecte au 3-way match.

Interactions : achat alimente le stock, la finance et le reporting.

### 6.3 Ventes

Le module ventes gere le cycle devis vers encaissement et pilote la reservation de stock.

Fonctionnalites attendues :

- Devis / pro-forma avec validite et remises plafonnees.
- Commandes clients avec verification de disponibilite stock.
- Reservation de stock configurable lors de la validation de commande.
- Preparation et livraison avec mouvements de sortie.
- Facturation automatique depuis les livraisons.
- Avoirs clients avec double validation et separation des roles.
- Encaissements multi-modes avec affectation aux factures.

Regles metier :

- Une remise exceptionnelle doit etre validee par un responsable.
- Une livraison est bloquee si le stock disponible est insuffisant.
- Le createur d'un avoir ne peut pas etre le validateur ni l'encaisseur.

Interactions : ventes consomme le stock, alimente la facturation et la comptabilite.

### 6.4 Stock

Le module stock est le coeur de la tracabilite. Il gere les lots, les mouvements, la reservation et les transferts.

Fonctionnalites attendues :

- Gestion des lots et series avec statut qualite (disponible, quarantaine, expire).
- Visualisation stock par article, depot, emplacement, lot.
- Mouvements types : entree, sortie, transfert, ajustement.
- Reservation de stock a la commande client et allocation FIFO/FEFO.
- Transferts inter-depots avec validation et double mouvement.

Regles metier :

- Un lot expire est automatiquement bloque et ne peut pas sortir.
- Le stock disponible est egal au stock physique moins le stock reserve.
- Les mouvements sont historises et non modifiables.

Interactions : stock alimente inventaires, ventes, achats et tableaux de bord.

### 6.5 Inventaires

Le module inventaires assure la fiabilite entre stock theorique et stock physique.

Fonctionnalites attendues :

- Sessions d'inventaire annuelles, tournantes ou ponctuelles.
- Saisie des comptages, multi-tours si ecarts significatifs.
- Calcul automatique des ecarts et demande d'ajustement.
- Validation des ecarts par un role distinct.

Regles metier :

- Le compteur ne peut pas valider l'ajustement.
- Les ecarts importants exigent une validation renforcee.
- Le gel des mouvements pendant inventaire est configurable.

Interactions : inventaires generent des mouvements correctifs et alimentent les KPI.

## 7. Gestion des workflows

Les workflows structurent les validations et assurent la separation des taches. Les principaux workflows sont :

- DA : Brouillon -> Soumise -> En approbation -> Approuvee ou Rejetee.
- BC : Brouillon -> Approuve -> Envoye -> Receptionne.
- Reception : Saisie -> Controle qualite -> Validation -> Mouvement d'entree.
- Commande client : Brouillon -> Confirmee -> Preparation -> Livree -> Facturee.
- Inventaire : Planifie -> En cours -> Analyse -> Valide -> Cloture.

Chaque transition genere un historique d'audit avec l'utilisateur, la date et le commentaire eventuel. Les seuils d'approbation sont parametrables par role et montant.

## 8. Regles metier detaillees

Les regles suivantes sont critiques et doivent etre enforcees par l'application :

- Separation des taches :
  - Createur DA ≠ Approbat. DA
  - Createur BC ≠ Approbat. BC
  - Receptionnaire ≠ Validateur facture
  - Createur avoir ≠ Validateur ≠ Encaisseur
  - Compteur inventaire ≠ Validateur ajustement
- Numerotation automatique non reutilisable pour les documents et mouvements.
- Traçabilite complete des mouvements avec lien au document source et a l'utilisateur.
- Blocage des lots expires et gestion des statuts qualite.
- Respect des plafonds de remise et de credit client.
- Interdiction de modifier l'historique des mouvements de stock.
- Validation budgetaire obligatoire avant engagement d'achats.
- Rapprochement 3-way match obligatoire avant validation facture fournisseur.

## 9. Modules avances

Les modules avances couvrent la valorisation, les KPI avances et la traçabilite de lot de bout en bout.

- Valorisation : support FIFO et CUMP, calcul de couts unitaires, gel mensuel, rapports d'ecarts.
- Traçabilite lot : lien fournisseur -> lot -> client pour les audits qualite.
- KPIs avances : rotation, stock dormant, obsolescence, precision stock.

## 10. Parametres et configuration

Les parametres doivent etre accessibles par les administrateurs et par role :

- Seuils d'approbation par role et montant.
- Parametres de reservation automatique du stock.
- Methodes de valorisation par famille.
- Regles de gel de stock et de cloture mensuelle.
- Parametrage des taxes, devises, unites et conversions.

## 11. Securite, audit et permissions

Le systeme applique un RBAC avec roles predefinis et restrictions par site ou depot. Les actions sensibles exigent des permissions explicites et la journalisation dans un audit non modifiable.

Controles attendus :

- Gestion complete des utilisateurs et roles, activation/desactivation.
- Delegations temporaires avec date de debut et fin.
- Journal d'audit complet : action, utilisateur, date, avant/apres, IP.
- Restriction d'acces par perimetre (site, depot, famille).

## 13. Regles transverses importantes

- Toute action generant un impact stock doit creer un mouvement de stock correspondant.
- Les documents doivent toujours etre lies a une organisation (societe, site, depot).
- Les validations doivent etre traçables et non supprimables.
- Les statuts doivent etre coherents avec les etapes du workflow et interdits de saut.
- Les modifications critiques doivent etre journalisees.

## 14. Perimetre fonctionnel couvert

Le perimetre fonctionnel couvre :

- Achats : DA, BC, receptions, factures fournisseurs, paiements, litiges.
- Ventes : devis, commandes, livraisons, factures clients, encaissements, avoirs.
- Stock : lots, mouvements, transferts, reservations, inventaires.
- Referentiels : articles, familles, clients, fournisseurs, taxes, devises, unites.
- Administration : utilisateurs, roles, audit et parametres.

Le perimetre multi-sites et multi-depots est obligatoire, avec traçabilite et workflows completes pour les operations critiques.
