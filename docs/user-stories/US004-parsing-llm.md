# US004 - Parsing LLM (Mistral) pour emails non-structures

## User Story

**En tant que** systeme automatise  
**Je veux** utiliser un LLM pour extraire les donnees des emails non-structures  
**Afin de** traiter les emails de syndics inconnus ou mal formates

---

## Criteres d'acceptation

### Scenario 1: Email non-structure
**Donne** un email d'un syndic inconnu en texte libre  
**Quand** le parser LLM traite l'email  
**Alors** les donnees sont extraites avec un score de confiance

### Scenario 2: Email partiellement structure
**Donne** un email avec certains champs identifiables  
**Quand** le parser LLM traite l'email  
**Alors** les champs extraits sont completes par le LLM

### Scenario 3: Email ininterpretable
**Donne** un email sans information exploitable  
**Quand** le parser LLM traite l'email  
**Alors** l'email est marque pour revue manuelle avec confiance 0

---

## Specifications techniques

### Provider LLM
- **Modele**: Mistral (via OpenRouter / Replit AI Integrations)
- **Temperature**: 0.1 (reponses deterministes)
- **Max tokens**: 2000

### Prompt systeme

```
Tu es un assistant specialise dans l'extraction de donnees depuis des emails de demande d'intervention pour des travaux d'etancheite.

Extrait les informations suivantes au format JSON:
- adresse: adresse complete du bien
- code_postal: code postal (5 chiffres)
- ville: nom de la ville
- objet: objet de la demande (type d'intervention)
- detail: description detaillee du probleme
- contacts: liste des contacts sur place (nom, telephone, qualite)
- gestionnaire: nom du gestionnaire/syndic
- ref_syndic: reference ou numero de dossier si present
- date_demande: date de la demande si presente

Si une information n'est pas trouvee, utilise null.
Ajoute un champ "confiance" entre 0 et 1 pour chaque extraction.

Reponds UNIQUEMENT avec le JSON, sans explication.
```

### Exemple de reponse attendue

```json
{
  "adresse": "29 RUE DES ECONDEAUX",
  "code_postal": "93800",
  "ville": "EPINAY SUR SEINE",
  "objet": "Recherche de fuite",
  "detail": "Fuite d'eau provenant de l'etage superieur, traces d'humidite au plafond",
  "contacts": [
    {"nom": "Mme PALENA", "telephone": null, "qualite": "proprietaire"}
  ],
  "gestionnaire": "Foncia",
  "ref_syndic": null,
  "date_demande": "2025-10-14",
  "confiance": {
    "adresse": 0.9,
    "code_postal": 0.95,
    "ville": 0.95,
    "objet": 0.85,
    "detail": 0.8,
    "contacts": 0.7,
    "gestionnaire": 0.6,
    "ref_syndic": 0,
    "date_demande": 0.9
  }
}
```

---

## Gestion des erreurs

| Erreur | Action |
|--------|--------|
| Timeout LLM | Retry 3x puis marquer echec |
| Reponse non-JSON | Tenter extraction regex basique |
| Confiance < 0.5 | Marquer pour revue manuelle |
| Quota depasse | Alerter admin, mettre en queue |

---

## Definition of Done

- [ ] Integration OpenRouter/Mistral via Replit
- [ ] Prompt optimise pour extraction
- [ ] Parsing de la reponse JSON
- [ ] Calcul confiance globale
- [ ] Fallback si LLM echoue
- [ ] Logging des tokens utilises
- [ ] Tests avec emails varies
