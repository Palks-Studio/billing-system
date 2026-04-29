<p align="center">
  <img src="docs/images/paid_fr.png"
       alt="Interface d’acquittement des factures — consultation des factures en attente et marquage comme payées"
       width="600">
</p>

> 🇫🇷 Français | [🇬🇧 English](./README.md)

![License](https://img.shields.io/badge/License-LICENSE.md-lightgreen.svg)
![Offline First](https://img.shields.io/badge/Mode-Offline%20First-0095b1?style=flat)
![Self Hosted](https://img.shields.io/badge/Hosting-Self%20Hosted-6b7280?style=flat)
![PHP](https://img.shields.io/badge/PHP-8%2B-777BB4?style=flat&logo=php&logoColor=white)
[![Facturation sans SaaS](https://img.shields.io/badge/Facturation%20sans%20SaaS-0095b1?style=flat)](https://palks-studio.com/fr/facturation-sans-saas)

<p align="center">
  <a href="https://palks-studio.com">
    <img src="https://img.shields.io/badge/Palks%20Studio-Website-0095b1?style=for-the-badge" />
  </a>
</p>

# Billing System

> ⚠️ Ce dépôt présente le projet et sa documentation technique.  
> La version de production n’est pas distribuée publiquement.

> L’installation est réalisée directement sur l’hébergement du client.  
> Si vous souhaitez utiliser ce système, merci de contacter **Palks Studio**.

Système de facturation complet, autonome et bilingue (FR/EN), installable sur tout hébergement PHP/Apache. Aucune base de données. Aucune dépendance SaaS. Hébergement autonome avec contrôle total des données.

[![Facturation sans SaaS](https://img.shields.io/badge/Facturation%20sans%20SaaS-0095b1?style=flat)](https://palks-studio.com/fr/facturation-sans-saas)

---

## Présentation générale

Billing System est une suite de trois outils de facturation reliés entre eux, accessibles depuis une interface unifiée. Il couvre l'intégralité du cycle de vie d'une prestation :  de l'émission du devis jusqu'à l'acquittement de la facture, en passant par la signature électronique et l'archivage structuré.

Le système est conçu pour être déployé directement chez le client, sur un hébergement Apache standard avec PHP 8.x et Composer. Il ne nécessite ni base de données, ni service tiers, ni abonnement.

---

## Fonctionnalités

- Server-side PDF quote generation (mPDF)  
- Génération automatique de la facture acquittée au moment de la facturation  
- Signature électronique du devis par le client (canvas tactile/souris)  
- Auto-remplissage client depuis les archives (SIREN, SIRET, TVA, email, nom)  
- Archivage structuré par client et par période  
- Numérotation séquentielle sécurisée (verrou fichier)  
- Export mensuel des factures (archive ZIP) depuis l’interface  
- Export mensuel des recettes (CSV) depuis l’interface  
- Export annuel du journal des recettes (CSV)  
- Envoi email automatique à chaque étape (devis, facture, acquittement)  
- Navigation entre les trois modules depuis une barre commune  
- Interface bilingue FR/EN avec switch en temps réel  
- Mode sombre / mode clair persisté  
- Aucune base de données  
- Aucune dépendance SaaS  
- Sécurité minimale : sessions sécurisées, tokens, anti-brute force  
- Gestion internationale : mentions légales TVA automatiques selon la zone client (FR / UE / hors UE)  
- TVA forcée à 0 % pour les clients UE et hors UE, avec mention légale adaptée  
- 249 pays centralisés dans `countries.php` — code ISO en backend, nom complet en interface  
- Autocomplétion pays sans accents, affichage en nom complet, transmission en ISO  
- Colonne TVA masquée dans les documents si aucune TVA applicable  
- Colonne `zone_tva` dans le journal des recettes CSV (valeurs : `fr`, `eu`, `world`)

---

## Structure du projet

```
billing-system/
│
├── public/
│   │  └── assets/
│   │      ├── logo*              → Logo de l'entreprise si fourni
│   │      ├── signature.png      → Signature de l'utilisateur utilisée sur les devis et les factures (format PNG)
│   │      └── favicon*           → Favicon optionnel affiché dans l’onglet du navigateur
│   │
│   ├── Endpoint a                → Endpoint de génération de devis
│   ├── Endpoint b                → Endpoint de génération de facture
│   ├── Endpoint c                → Endpoint de génération directe de facture
│   ├── Endpoint d                → Endpoint de génération de facture acquittée
│   │ 
│   ├── Interface a               → Interface de génération des devis
│   ├── Interface b               → Interface de génération directe de facture
│   ├── Interface c               → Interface permettant de marquer une facture comme payée
│   ├── sign.php                  → Interface de signature
│   ├── export a                  → Export ZIP des factures archivées
│   ├── export b                  → Export CSV du journal des recettes
│   │
│   ├── countries.php             → Liste de 249 pays au format ISO
│   ├── search.php                → Recherche et auto-remplissage des informations client
│   ├── serve.php                 → Accès sécurisé aux PDF via token
│   └── save.php                  → Sauvegarde des devis générés et archivage
│
├── vendor/                       → Bibliothèques utilisées par le moteur de génération des documents
├── templates/                    → Modèles HTML utilisés pour le rendu des documents
│
├── app.php                       → Configuration centrale de l’émetteur et des coordonnées bancaires
├── mailer.php                    → Script interne d’envoi d’emails avec pièces jointes
├── core.php                      → Moteur principal : logique de génération, calculs et archivage
├── LICENCE.md                    → Licence du projet
│
├── contracts/                    → Archivage des devis signés et non signés
├── counters/                     → Compteurs de numérotation (devis et factures)
├── logs/                         → Journaux système (optionnel)
├── data/                         → Données opérationnelles
│
└── docs/
    ├── GUIDE_UTILISATEUR.md      → Guide utilisateur
    ├── OVERVIEW_FR.md            → Vue d’ensemble du projet et de son fonctionnement
    └── README_FR.md              → Documentation d’installation et d’utilisation (version client)
```


---

## Les trois modules

### 1. Générateur de devis

**Fonctionnement :**

1. L'utilisateur remplit le formulaire : coordonnées émetteur, coordonnées client, lignes de prestation, coordonnées bancaires, paramètres (devise, langue PDF).  
2. Un aperçu des totaux HT / TVA / TTC est calculé en temps réel.  
3. À la soumission, une fenêtre de confirmation s'affiche avant génération.  
4. Le PDF est généré localement et téléchargé. Simultanément, le devis est archivé côté serveur avec un token de signature valable 30 jours.  
5. Un email est envoyé au client avec un lien de consultation et de signature en ligne.

**Détails techniques :**

- La numérotation est automatique et repart de la base existante du client  
- Auto-remplissage client par SIREN, SIRET, TVA, email ou nom  
- Validation Luhn SIRET/SIREN côté navigateur  
- Auto-complétion SIREN depuis SIRET  
- Switch langue FR/EN en temps réel sans rechargement  
- Sélecteur devise : EUR, USD, GBP, CHF, CAD  
- Coordonnées bancaires (IBAN/BIC) conditionnelles dans le PDF  
- Bloc « Bon pour accord » avec image de signature configurable  
- Footer TVA conditionnel (art. 293B CGI si TVA = 0)  
- Pagination PDF automatique

---

### 2. Facturation directe

Interface de génération de facture côté serveur via Dompdf. Produit simultanément deux PDF à chaque génération : la facture normale et la facture acquittée (pré-générée, en attente de paiement).

**Fonctionnement :**

1. L’utilisateur remplit le formulaire : informations émetteur, informations client, lignes de prestation, coordonnées bancaires, paramètres (devise, langue du PDF).  
2. Un aperçu des totaux (HT / TVA / TTC) est calculé en temps réel.  
3. À la soumission, une fenêtre de confirmation s’affiche avant la génération.  
4. Le PDF est généré localement puis téléchargé. En parallèle, le devis est archivé côté serveur avec un jeton de signature valide pendant 30 jours.  
5. Un email est envoyé au client avec un lien pour consulter et signer le devis en ligne.

**Détails techniques :**

- Numérotation annuelle des factures avec mécanisme anti-collision  
- Protection contre la double facturation à partir d’une même référence de devis  
- Validation complète des entrées avec gestion des erreurs en français et en anglais  
- Calculs financiers précis avec gestion des imprécisions liées aux nombres flottants  
- Agrégation de la TVA par taux  
- Traitement automatique du logo pour compatibilité PDF  
- Récupération automatique des informations émetteur depuis les métadonnées existantes  
- Traçabilité des PDF via empreinte cryptographique (SHA-256)  
- Envoi des factures par email avec pièce jointe, sans dépendance externe

---

### 3. Acquittement et suivi des paiements

Interface de suivi des factures en attente et de validation des paiements. Permet d’identifier les factures non réglées, de les marquer comme payées et de suivre les recettes.

**Fonctionnement :**  

1. Le module détecte automatiquement les factures en attente de paiement.  
2. L’utilisateur sélectionne une date de règlement et clique sur « Marquer comme payé ».  
3. Le système archive la facture comme acquittée, met à jour les métadonnées, enregistre la transaction dans le journal des recettes et envoie la facture au client par email.  
4. Un tableau des dernières recettes de l’année est affiché en bas de page.

**Détails techniques :**  

- Détection automatique des factures à traiter  
- Filtrage des factures déjà enregistrées comme payées  
- Prévention des doublons dans le journal des recettes  
- Gestion sécurisée des écritures (verrouillage)  
- Protection contre la double soumission des formulaires  
- Indicateur du nombre de factures en attente  
- Export annuel des recettes au format CSV

---

### Modules transverses

Composants partagés au sein du système, assurant cohérence, sécurité et automatisation.

#### Signature électronique

Page publique accessible via un lien sécurisé. Permet au client de consulter le devis et d’apposer une signature électronique.

- Aperçu PDF multi-pages dans le navigateur  
- Capture de signature (compatible tactile et souris)  
- Validation des entrées (format et taille minimale)  
- Enregistrement de l’image de signature  
- Mise à jour des métadonnées avec horodatage et traçabilité  
- Envoi d’emails de confirmation au client et à l’émetteur (FR/EN)  
- Protection contre la double signature  
- Expiration du lien après 30 jours

#### Auto-remplissage client

Endpoint JSON utilisé lors de la saisie des formulaires. Recherche dans les données existantes via SIREN, SIRET, TVA, email, nom ou numéro de devis.

- Retour des coordonnées complètes, langue, devise et éléments associés  
- Pré-remplissage automatique des factures à partir des devis existants  
- Normalisation des entrées pour garantir la cohérence des correspondances

#### Accès PDF sécurisé

Couche d’accès basée sur un jeton permettant de servir un PDF sans exposer son emplacement physique.

#### Mailer interne

Module d’envoi d’emails avec gestion des pièces jointes, utilisé par l’ensemble du système. Aucune dépendance à un service SMTP externe.

---

## Sécurité

- Gestion sécurisée des sessions (cookies protégés, isolation des accès)  
- Protection contre les tentatives de brute force  
- Régénération des identifiants de session après authentification  
- Utilisation de jetons sécurisés pour l’accès aux ressources sensibles  
- Validation stricte des entrées et des identifiants  
- Traçabilité des actions via empreintes cryptographiques  
- Protection des endpoints par contrôle d’accès  
- Politique de sécurité des réponses HTTP (type, cache, indexation)  
- Interfaces internes non indexées et non exposées publiquement  
- Aucune dépendance à des services externes : toutes les données restent sous contrôle

---

## Emails automatiques

| Événement          | Destinataire(s)   | Contenu                                  |
|--------------------|-------------------|------------------------------------------|
| Génération devis   | Client            | Lien signature + lien téléchargement PDF |
| Signature devis    | Client + Émetteur | Confirmation de signature + lien PDF     |
| Génération facture | Client            | Facture en pièce jointe                  |
| Acquittement       | Client            | Facture acquittée en pièce jointe        |

Tous les emails sont bilingues FR/EN selon la langue du document.

---

## Prérequis techniques

- PHP 8.1 ou supérieur  
- Serveur web compatible (Apache recommandé)  
- Gestion des réécritures d’URL  
- Extension GD (traitement des images)  
- Fonction d’envoi d’emails active côté serveur  
- Droits d’écriture sur les répertoires de stockage et d’archivage

---

## Archivage et données

Toutes les données sont stockées sous forme de fichiers plats. Aucune base de données n’est requise.

| Type                     | Description                                      | Format              |
|--------------------------|--------------------------------------------------|---------------------|
| Devis                    | Archivés avec leurs métadonnées associées        | PDF + métadonnées   |
| Signatures               | Stockées sous forme d’images                     | PNG                 |
| Factures                 | Archivées avec leurs métadonnées associées       | PDF + métadonnées   |
| Factures en attente      | Pré-générées en attente de validation            | PDF                 |
| Factures acquittées      | Archivées après confirmation de paiement         | PDF                 |
| Recettes                 | Export structuré des transactions                | CSV                 |
| Compteurs                | Suivi de la numérotation séquentielle            | Entier              |
| Logs                     | Journal des activités du système                 | Texte horodaté      |

---

© Palks Studio — voir LICENSE.md  
- https://palks-studio.com
