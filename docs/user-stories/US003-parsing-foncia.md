# US003 - Parsing template Foncia

## User Story

**En tant que** systeme automatise  
**Je veux** extraire les donnees structurees des emails Foncia  
**Afin de** creer automatiquement les biens et demandes

---

## Criteres d'acceptation

### Scenario 1: Email standard Foncia
**Donne** un email Foncia avec sujet "Ordre de service N° OSMIL807626360 – RECHERCHE DE FUITE..."  
**Quand** le parser Foncia traite l'email  
**Alors** les donnees suivantes sont extraites:
- Numero ordre de service: OSMIL807626360
- Objet: RECHERCHE DE FUITE MME PALENA / TOUZARD
- Adresse: 29 / 31 / 33 RUE DES ECONDEAUX
- Code postal: 93800
- Ville: EPINAY SUR SEINE
- Contacts sur place
- Gestionnaire (depuis signature)

### Scenario 2: Champs manquants
**Donne** un email Foncia incomplet  
**Quand** le parser traite l'email  
**Alors** les champs manquants sont marques et la confiance est reduite

---

## Specifications techniques

### Patterns regex pour Foncia

```python
PATTERNS = {
    "ordre_service": r"Ordre de service N° ([A-Z0-9]+)",
    "adresse_sujet": r"– .+ - (.+) (\d{5}) (.+)$",
    "immeuble": r"Immeuble N° (\d+) : (.+) - (.+) (\d{5}) (.+),",
    "objet": r"Objet : (.+)",
    "contact": r"- ([^(]+) \(([^)]+)\) ([+\d\s]+)",
    "gestionnaire": r"représenté\(e\) par ([^,]+),",
}
```

### Structure extraite

```json
{
  "ref_syndic": "OSMIL807626360",
  "bien": {
    "adresse": "29 / 31 / 33 RUE DES ECONDEAUX",
    "code_postal": "93800",
    "ville": "EPINAY SUR SEINE",
    "information": "LES AURELLES 4 - REF 6411 - Immeuble N° 501147727",
    "confiance": 0.95
  },
  "demande": {
    "objet": "RECHERCHE DE FUITE MME PALENA / TOUZARD",
    "detail": "Corps complet...",
    "contacts": [
      {"nom": "CHARLINE ET JULIE", "qualite": "coproprietaire", "tel": "+33651820580"}
    ]
  },
  "gestionnaire": {
    "nom": "ZOZAYA VIRION Marine",
    "email": "marine.zozaya@foncia.com",
    "agence": "Foncia Vaucelles"
  }
}
```

---

## Exemple d'email Foncia

```
Subject: Ordre de service N° OSMIL807626360 – RECHERCHE DE FUITE MME PALENA / TOUZARD - 29 / 31 / 33 RUE DES ECONDEAUX 93800 EPINAY SUR SEINE

Bonjour,

En notre qualité de syndic, représenté(e) par Marine ZOZAYA VIRION, du bien désigné en objet, nous vous remercions d'exécuter les travaux décrits dans l'ordre de service détaillé ci-dessous :

Immeuble N° 501147727 : LES AURELLES 4 - REF 6411 - 29 / 31 / 33 RUE DES ECONDEAUX 93800 EPINAY SUR SEINE,

Objet : RECHERCHE DE FUITE MME PALENA / TOUZARD

Contacts sur place :
- CHARLINE ET JULIE (copropriétaire) +33651820580
- TOUZARD / CIVEL DANIEL (copropriétaire) 06 72 52 XX XX
```

---

## Definition of Done

- [ ] Parser template Foncia implemente
- [ ] Extraction numero ordre de service
- [ ] Extraction adresse complete (rue, CP, ville)
- [ ] Extraction objet de la demande
- [ ] Extraction contacts sur place
- [ ] Extraction gestionnaire depuis signature
- [ ] Score de confiance calcule
- [ ] Tests avec 5+ emails Foncia reels
