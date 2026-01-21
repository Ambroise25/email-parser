# US-EP-001 : Connexion IMAP

## User Story

**En tant qu'** utilisateur du système Email2Extranet,
**Je veux** que l'application se connecte automatiquement à ma boîte mail,
**Afin de** récupérer les nouveaux emails de syndics sans intervention manuelle.

## Critères d'acceptation

1. L'application se connecte à un serveur IMAP (Gmail, OVH, etc.)
2. Elle lit les emails non lus du dossier INBOX
3. Chaque email est passé au parser LLM
4. Les emails traités sont marqués comme lus

## Configuration requise

| Variable | Description | Exemple |
|----------|-------------|---------|
| `IMAP_HOST` | Serveur IMAP | imap.gmail.com |
| `IMAP_USER` | Adresse email | interventions@enerpur.fr |
| `IMAP_PASSWORD` | Mot de passe ou App Password | ******** |

## Notes techniques

- Utiliser la librairie Python `imaplib` (standard)
- Support SSL/TLS obligatoire
- Gérer les erreurs de connexion (timeout, mauvais identifiants)

## Priorité

Haute - Fonctionnalité de base pour l'automatisation

## Statut

A développer
