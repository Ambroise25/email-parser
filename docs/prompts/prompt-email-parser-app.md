# Prompt pour créer l'app Email Parser

Copiez ce texte dans une nouvelle app Replit pour démarrer :

---

## Prompt

Je veux créer une application Email Parser pour le projet Email2Extranet.

### Philosophie de travail (IMPORTANT)

1. **Baby steps** : On avance par petites étapes. Une fonctionnalité à la fois.
2. **Documentation d'abord** : Avant de coder, on documente ce qu'on va faire.
3. **Demander permission** : Avant toute action externe (push GitHub, API, etc.), demande-moi.
4. **Itératif** : On teste, on valide, puis on passe à l'étape suivante.

### Première étape uniquement

Pour commencer, je veux UNIQUEMENT :
- Me connecter à une boîte mail via IMAP
- Lire UN email non lu
- Afficher son contenu brut (pas de parsing pour l'instant)

### Variables d'environnement

- `IMAP_HOST` : Serveur IMAP (ex: imap.gmail.com)
- `IMAP_USER` : Adresse email
- `IMAP_PASSWORD` : Mot de passe

### Contexte du projet

Cette app fait partie du projet Email2Extranet :
- Extraction automatique des demandes d'intervention depuis les emails de syndics
- Parser LLM (Mistral) pour extraire les données
- Métier unique : Etanchéité

Ne code pas tout d'un coup. Propose-moi d'abord ce que tu vas faire, attends ma validation.

---
