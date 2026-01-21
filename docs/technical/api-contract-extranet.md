# Contrat API - Extranet Symfony

**Date** : 21 janvier 2026  
**Version** : 1.0  
**Objectif** : Definir les endpoints API necessaires pour l'integration Email2Extranet

---

## Vue d'ensemble

L'Email Parser doit pouvoir :
1. Rechercher si un bien existe deja
2. Creer un nouveau bien si necessaire
3. Creer une demande liee a un bien
4. Uploader des pieces jointes

```
Email Parser  ───►  API Extranet  ───►  Base de donnees Symfony
```

---

## Authentification

| Element | Valeur |
|---------|--------|
| Methode | API Key ou Bearer Token |
| Header | `Authorization: Bearer <token>` |
| Securite | HTTPS obligatoire |

> L'API Key sera stockee comme secret dans l'Email Parser.

---

## Endpoints

### 1. Rechercher un bien

**`GET /api/biens`**

Recherche un bien existant par ses criteres d'adresse.

#### Parametres de requete

| Parametre | Type | Obligatoire | Description |
|-----------|------|-------------|-------------|
| `adresse` | string | Non | Numero et nom de rue |
| `code_postal` | string | Non | Code postal |
| `ville` | string | Non | Ville |

#### Reponse (200 OK)

```json
{
  "total": 1,
  "biens": [
    {
      "id": 107,
      "adresse": "Bien de test",
      "complement_adresse": null,
      "code_postal": "95300",
      "ville": "test",
      "gestionnaire": {
        "id": 1,
        "nom": "Quentin Cunat"
      },
      "client": {
        "id": 1,
        "nom": "QCC"
      }
    }
  ]
}
```

#### Reponse si aucun resultat (200 OK)

```json
{
  "total": 0,
  "biens": []
}
```

---

### 2. Creer un bien

**`POST /api/biens`**

Cree un nouveau bien dans l'Extranet.

#### Corps de la requete

```json
{
  "adresse": "12 rue de la Paix",
  "complement_adresse": "Batiment A",
  "code_postal": "75002",
  "ville": "Paris",
  "gestionnaire_id": 1,
  "information": "Copropriete Les Jardins - Digicode 1234",
  "conseil_syndical": [
    {
      "prenom": "Jean",
      "nom": "Dupont",
      "email": "jean.dupont@email.com",
      "telephone": "0612345678",
      "president": true
    }
  ]
}
```

#### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `adresse` | string | Oui | Numero et nom de rue |
| `complement_adresse` | string | Non | Batiment, escalier, etc. |
| `code_postal` | string | Oui | Code postal |
| `ville` | string | Oui | Ville |
| `gestionnaire_id` | integer | Oui | ID du gestionnaire interne |
| `information` | string | Oui | Infos complementaires (codes, copro) |
| `conseil_syndical` | array | Non | Membres du conseil syndical |

#### Reponse (201 Created)

```json
{
  "id": 108,
  "adresse": "12 rue de la Paix",
  "code_postal": "75002",
  "ville": "Paris",
  "created_at": "2026-01-21T10:30:00Z"
}
```

---

### 3. Creer une demande

**`POST /api/demandes`**

Cree une nouvelle demande liee a un bien.

#### Corps de la requete

```json
{
  "bien_id": 107,
  "date_demande": "2026-01-21",
  "objet": "Infiltration terrasse - fuite importante",
  "metier": "Etancheite",
  "statut": "En attente de traitement",
  "urgence": "Urgent",
  "details": {
    "description": "Fuite d'eau importante au niveau de la terrasse du 5eme etage",
    "ref_syndic": "OS-2026-001234",
    "contact_nom": "Mme Martin",
    "contact_telephone": "0698765432",
    "contact_email": "martin@foncia.fr",
    "codes_acces": "Digicode 1234, Interphone Martin"
  },
  "source": {
    "type": "email",
    "email_id": "msg-123456",
    "date_reception": "2026-01-21T08:30:00Z",
    "expediteur": "gestionnaire@foncia.fr"
  }
}
```

#### Champs principaux

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `bien_id` | integer | Oui | ID du bien concerne |
| `date_demande` | date | Oui | Date de la demande |
| `objet` | string | Oui | Description courte |
| `metier` | string | Oui | Etancheite, Plomberie, Couverture, Autre |
| `statut` | string | Oui | Statut initial |
| `urgence` | string | Non | Urgent, Normal, Faible |

#### Champs details (objet `details`)

| Champ | Type | Description |
|-------|------|-------------|
| `description` | string | Description complete |
| `ref_syndic` | string | Reference OS/devis du syndic |
| `contact_nom` | string | Nom du contact |
| `contact_telephone` | string | Telephone du contact |
| `contact_email` | string | Email du contact |
| `codes_acces` | string | Codes d'acces, digicodes |

#### Champs source (objet `source`)

| Champ | Type | Description |
|-------|------|-------------|
| `type` | string | "email" |
| `email_id` | string | ID unique de l'email |
| `date_reception` | datetime | Date/heure de reception |
| `expediteur` | string | Email de l'expediteur |

#### Statuts possibles

| Statut | Description |
|--------|-------------|
| `En attente de traitement` | Demande recue, a traiter |
| `A contacter` | Premier contact a faire |
| `A relancer` | Attente reponse client |
| `Rdv non pris` | Discussion en cours |
| `Rendez-vous programme` | RDV fixe |

#### Metiers possibles

| Metier | Description |
|--------|-------------|
| `Etancheite` | Toiture-terrasse, infiltrations |
| `Plomberie` | Canalisations, fuites |
| `Couverture` | Tuiles, ardoises, gouttieres |
| `Autre` | A preciser |

#### Reponse (201 Created)

```json
{
  "id": 11950,
  "bien_id": 107,
  "objet": "Infiltration terrasse - fuite importante",
  "statut": "En attente de traitement",
  "created_at": "2026-01-21T10:35:00Z"
}
```

---

### 4. Uploader une piece jointe

**`POST /api/fichiers`**

Upload un fichier (image ou PDF) lie a une demande.

#### Corps de la requete (multipart/form-data)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `demande_id` | integer | Oui | ID de la demande |
| `fichier` | file | Oui | Fichier a uploader |
| `type` | string | Non | "image" ou "document" |
| `description` | string | Non | Description du fichier |

#### Reponse (201 Created)

```json
{
  "id": 456,
  "demande_id": 11950,
  "nom": "photo_fuite.jpg",
  "type": "image",
  "taille": 245678,
  "url": "/uploads/demandes/11950/photo_fuite.jpg",
  "created_at": "2026-01-21T10:36:00Z"
}
```

---

### 5. Lister les gestionnaires

**`GET /api/gestionnaires`**

Liste les gestionnaires disponibles (pour associer un bien).

#### Reponse (200 OK)

```json
{
  "gestionnaires": [
    {
      "id": 1,
      "nom": "Quentin Cunat",
      "email": "quentin@enerpur.fr"
    },
    {
      "id": 2,
      "nom": "Guillaume Oyer",
      "email": "guillaume@enerpur.fr"
    }
  ]
}
```

---

### 6. Lister les clients (syndics)

**`GET /api/clients`**

Liste les clients/syndics enregistres.

#### Reponse (200 OK)

```json
{
  "clients": [
    {
      "id": 1,
      "nom": "QCC",
      "adresse": "..."
    },
    {
      "id": 2,
      "nom": "RBH Scholer",
      "adresse": "..."
    }
  ]
}
```

---

## Codes d'erreur

| Code | Signification |
|------|---------------|
| 200 | OK |
| 201 | Created |
| 400 | Bad Request - Donnees invalides |
| 401 | Unauthorized - Token manquant/invalide |
| 404 | Not Found - Ressource inexistante |
| 422 | Unprocessable Entity - Validation echouee |
| 500 | Internal Server Error |

#### Exemple d'erreur

```json
{
  "error": {
    "code": 422,
    "message": "Validation failed",
    "details": {
      "code_postal": "Le code postal est obligatoire"
    }
  }
}
```

---

## Workflow d'integration

```
1. Email recu
      │
      ▼
2. Parser LLM extrait les donnees
      │
      ▼
3. GET /api/biens?adresse=...&code_postal=...
      │
      ├── Bien trouve ──► Recuperer bien_id
      │
      └── Bien non trouve ──► POST /api/biens ──► Recuperer bien_id
      │
      ▼
4. POST /api/demandes (avec bien_id)
      │
      ▼
5. Pour chaque piece jointe:
      POST /api/fichiers
      │
      ▼
6. Email deplace vers "Email traite"
```

---

## Notes pour l'implementation Symfony

1. **Bundle recommande** : API Platform ou FOSRestBundle
2. **Serialisation** : JSON uniquement
3. **Validation** : Symfony Validator avec contraintes
4. **Documentation** : OpenAPI/Swagger genere automatiquement
5. **Rate limiting** : A considerer (50 emails/jour = faible charge)

---

## Prochaines etapes

1. [ ] Valider ce contrat avec l'equipe Symfony
2. [ ] Implementer les endpoints dans l'Extranet
3. [ ] Tester avec Postman/Insomnia
4. [ ] Connecter l'Email Parser
