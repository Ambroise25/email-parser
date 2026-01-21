# Architecture Technique - Email2Extranet

**Mise a jour** : 21 janvier 2026

---

## Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              FLUX GLOBAL                                 │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────┐      ┌─────────────────┐      ┌─────────────────┐
    │  EMAILS  │─────▶│  EMAIL-PARSER   │─────▶│ API EXTRANET    │
    │  (IMAP)  │      │                 │      │ (Symfony)       │
    └──────────┘      │ - Lecture IMAP  │      │                 │
                      │ - Parsing LLM   │      │ - REST API      │
                      │   (Mistral)     │      │ - CRUD Biens    │
                      │ - Pieces jointes│      │ - CRUD Demandes │
                      │ - Archivage     │      │ - Fichiers      │
                      └─────────────────┘      └────────┬────────┘
                                                        │
                      ┌─────────────────┐               │
                      │    EXTRANET     │◀──────────────┘
                      │    (Symfony)    │
                      │                 │
                      │ - Tableau bord  │
                      │ - Gestion biens │
                      │ - Suivi demandes│
                      └─────────────────┘
```

---

## Composants

### 1. EMAIL-PARSER (Nouvelle app Replit)

**Responsabilites :**
- Connexion IMAP a la boite email
- Lecture des emails non lus
- Extraction des donnees via LLM (Mistral)
- Detection du metier (Etancheite, Plomberie, Couverture, etc.)
- Evaluation de l'urgence
- Envoi vers API Extranet
- Archivage dans dossier "Email traite"
- Conservation des pieces jointes

**Stack technique :**
| Element | Choix | Justification |
|---------|-------|---------------|
| Langage | Python 3.11 | Bibliotheques email/NLP |
| Email | imaplib | Standard Python |
| LLM | Mistral via OpenRouter | Integration Replit |
| HTTP Client | requests | Appels vers API |

### 2. API EXTRANET (A developper dans Symfony)

**Responsabilites :**
- Exposer les endpoints REST pour l'integration
- Gerer les entites Bien et Demande
- Recherche et matching d'adresses
- Upload des pieces jointes

**Endpoints a creer :**
| Endpoint | Description |
|----------|-------------|
| `GET /api/biens` | Rechercher un bien |
| `POST /api/biens` | Creer un bien |
| `POST /api/demandes` | Creer une demande |
| `POST /api/fichiers` | Uploader pieces jointes |
| `GET /api/gestionnaires` | Lister gestionnaires |
| `GET /api/clients` | Lister syndics |

> Voir `docs/technical/api-contract-extranet.md` pour le detail complet.

### 3. EXTRANET EXISTANT (Symfony)

L'Extranet existant sur `extranet.enerpur.fr` continue de fonctionner.
Le tableau de bord Email2Extranet sera integre a terme.

**Entites existantes :**
- Bien (adresse, code postal, ville, gestionnaire)
- Demande (objet, metier, statut, lie a un bien)
- Intervention (date, responsable, compte-rendu)
- Client (syndic)
- Gestionnaire (interne Enerpur)

---

## Modele de donnees

### Bien

| Champ | Type | Description |
|-------|------|-------------|
| id | int | Identifiant unique |
| adresse | string | Numero et rue |
| complement_adresse | string | Batiment, escalier |
| code_postal | string | Code postal |
| ville | string | Ville |
| gestionnaire_id | int | Gestionnaire interne |
| information | string | Codes, copropriete |

### Demande

| Champ | Type | Description |
|-------|------|-------------|
| id | int | Identifiant unique |
| bien_id | int | Bien concerne |
| date_demande | date | Date de la demande |
| objet | string | Description courte |
| metier | enum | Etancheite, Plomberie, Couverture, Autre |
| statut | enum | En attente, A contacter, Rdv programme... |
| urgence | enum | Urgent, Normal, Faible |
| ref_syndic | string | Reference OS/devis |
| details | json | Codes acces, contacts, etc. |
| source_email | json | ID email, expediteur, date |

### Metiers

| Valeur | Description |
|--------|-------------|
| Etancheite | Toiture-terrasse, infiltrations |
| Plomberie | Canalisations, fuites interieures |
| Couverture | Tuiles, ardoises, gouttieres |
| Autre | A preciser |

### Niveaux d'urgence

| Valeur | Critere |
|--------|---------|
| Urgent | Fuite active, degats en cours |
| Normal | Demande de devis, pas de fuite |
| Faible | Preventif, controle |

### Statuts

| Valeur | Description |
|--------|-------------|
| En attente de traitement | Nouveau, a traiter |
| A contacter | Premier contact a faire |
| A relancer | Attente reponse |
| Rdv non pris | Discussion en cours |
| Rendez-vous programme | RDV fixe |

---

## Flux de traitement complet

```
1. Email recu dans INBOX
       │
       ▼
2. Email Parser lit l'email (IMAP)
       │
       ▼
3. LLM (Mistral) extrait les donnees :
   - Syndic, adresse du bien
   - Objet, metier, urgence
   - Contacts, codes d'acces
   - Pieces jointes
       │
       ▼
4. GET /api/biens?adresse=...&code_postal=...
       │
       ├── Bien trouve ──► Recuperer bien_id
       │
       └── Bien non trouve ──► POST /api/biens ──► bien_id
       │
       ▼
5. POST /api/demandes (avec bien_id)
       │
       ▼
6. Pour chaque piece jointe :
       POST /api/fichiers
       │
       ▼
7. Email deplace vers "Email traite" (IMAP)
       │
       ▼
8. Demande visible dans l'Extranet
```

---

## Schema de sortie du parser LLM

```json
{
  "email_id": "msg-12345",
  "date_reception": "2026-01-21T08:30:00Z",
  "syndic_detecte": "Foncia",
  "confiance_globale": 0.92,
  "bien": {
    "adresse": "12 rue de la Paix",
    "complement": "Batiment A",
    "code_postal": "75002",
    "ville": "Paris",
    "copropriete": "Les Jardins"
  },
  "demande": {
    "objet": "Infiltration terrasse niveau 5",
    "metier": "Etancheite",
    "urgence": "Urgent",
    "ref_syndic": "OS-2026-001234",
    "codes_acces": "Digicode 1234, Interphone Martin",
    "contacts": [
      {"nom": "Mme Martin", "tel": "0698765432", "email": "martin@foncia.fr"}
    ]
  },
  "gestionnaire": {
    "nom": "Marine ZOZAYA",
    "email": "marine.zozaya@foncia.com",
    "agence": "Foncia Vaucelles"
  },
  "pieces_jointes": [
    {"nom": "photo_fuite.jpg", "type": "image/jpeg", "taille": 245000}
  ]
}
```

---

## Securite

### Authentification API
- API Key ou Bearer Token
- Header : `Authorization: Bearer <token>`
- HTTPS obligatoire

### Stockage des secrets
- Token API dans secrets Replit (Email Parser)
- Pas d'exposition des credentials IMAP

---

## Deploiement

| Composant | Hebergement |
|-----------|-------------|
| Email Parser | Nouvelle app Replit |
| API Extranet | Serveur OVH (avec Symfony existant) |
| Extranet | Serveur OVH (extranet.enerpur.fr) |

---

## Prochaines etapes

1. [x] Documenter les exigences metier (requirements-v2.md)
2. [x] Definir le contrat API (api-contract-extranet.md)
3. [ ] Creer l'app Email Parser (nouvelle Replit)
4. [ ] Developper l'API dans Symfony
5. [ ] Integrer et tester
