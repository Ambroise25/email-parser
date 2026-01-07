# Email Parser App - Structure technique

## Vue d'ensemble

Application Python qui lit automatiquement les emails de syndics via IMAP et les parse avec un LLM (Mistral) pour extraire les données structurées.

## Structure des fichiers

```
email-parser-app/
├── main.py              # Point d'entrée - orchestration
├── imap_client.py       # Connexion et lecture des emails IMAP
├── llm_parser.py        # Parser LLM (Mistral via OpenRouter)
├── requirements.txt     # Dépendances Python
└── .env                 # Variables d'environnement (secrets)
```

## Fichiers détaillés

### main.py
- Se connecte à la boîte mail via `imap_client.py`
- Récupère les emails non lus
- Les passe au `llm_parser.py`
- Affiche/retourne le JSON résultat
- Marque les emails comme lus

### imap_client.py
- Connexion IMAP avec SSL/TLS
- Lecture des emails non lus (UNSEEN)
- Extraction des headers et du body
- Gestion des erreurs de connexion

### llm_parser.py
- Envoi de l'email au LLM Mistral via OpenRouter
- Extraction des données structurées (JSON)
- Métier fixé à "Etanchéité"

## Variables d'environnement

| Variable | Description | Obligatoire |
|----------|-------------|-------------|
| `IMAP_HOST` | Serveur IMAP (ex: imap.gmail.com) | Oui |
| `IMAP_USER` | Adresse email | Oui |
| `IMAP_PASSWORD` | Mot de passe ou App Password | Oui |
| `IMAP_FOLDER` | Dossier à lire (défaut: INBOX) | Non |

## Dépendances

```
openai>=1.0.0
```

Note: `imaplib` et `email` sont des librairies Python standard.

## Format de sortie

Chaque email parsé retourne un JSON conforme au contrat API Email2Extranet :

```json
{
  "email_id": "...",
  "date_reception": "2025-01-07T10:00:00Z",
  "syndic_detecte": "Foncia",
  "confiance_globale": 0.9,
  "bien": {
    "adresse": "...",
    "code_postal": "...",
    "ville": "..."
  },
  "demande": {
    "ref_syndic": "...",
    "objet": "...",
    "metier": "Etancheite"
  },
  "gestionnaire": {
    "nom": "...",
    "email": "..."
  }
}
```
