# US002 - Detection du syndic expediteur

## User Story

**En tant que** systeme automatise  
**Je veux** identifier le syndic qui envoie l'email  
**Afin de** appliquer le bon template de parsing

---

## Criteres d'acceptation

### Scenario 1: Syndic connu (Foncia)
**Donne** un email de no-reply@foncia.com  
**Quand** le systeme analyse l'expediteur  
**Alors** le syndic "foncia" est detecte avec confiance 100%

### Scenario 2: Syndic connu par domaine
**Donne** un email de gestionnaire@nexity.fr  
**Quand** le systeme analyse l'expediteur  
**Alors** le syndic "nexity" est detecte

### Scenario 3: Syndic inconnu
**Donne** un email de inconnu@gmail.com  
**Quand** le systeme analyse l'expediteur  
**Alors** le syndic est marque "inconnu" et le parsing LLM est utilise

---

## Specifications techniques

### Configuration des syndics
```json
{
  "syndics": [
    {
      "id": "foncia",
      "name": "Foncia",
      "domains": ["foncia.com", "foncia.fr"],
      "email_patterns": ["no-reply@foncia.com"],
      "parser": "template_foncia"
    },
    {
      "id": "nexity",
      "name": "Nexity",
      "domains": ["nexity.fr"],
      "parser": "template_nexity"
    }
  ]
}
```

### Algorithme de detection
1. Verifier si l'email expediteur match un pattern exact
2. Sinon, extraire le domaine et chercher dans la liste
3. Sinon, marquer comme "inconnu"

---

## Definition of Done

- [ ] Fichier de configuration syndics.json
- [ ] Detection par pattern email exact
- [ ] Detection par domaine
- [ ] Fallback vers "inconnu"
- [ ] Tests pour chaque syndic configure
