# US001 - Extraction de base des emails

## User Story

**En tant que** systeme automatise  
**Je veux** extraire les informations cles d'un email de demande d'intervention  
**Afin de** les transmettre au CRM sans intervention manuelle

---

## Criteres d'acceptation

### Scenario 1: Email standard
**Donne** un email avec format standard (syndic connu)  
**Quand** le systeme traite l'email  
**Alors** les champs suivants sont extraits:
- [ ] Nom du syndic
- [ ] Adresse de l'immeuble
- [ ] Type d'intervention
- [ ] Description du probleme
- [ ] Coordonnees contact

### Scenario 2: Email mal formate
**Donne** un email avec format non standard  
**Quand** le systeme traite l'email  
**Alors** l'email est mis en file d'attente pour traitement manuel

---

## Notes techniques

- Utiliser regex pour extraction initiale
- Fallback sur NLP si regex echoue
- Logging detaille pour debug

---

## Definition of Done

- [ ] Code implemente et revise
- [ ] Tests unitaires passes (>80% coverage)
- [ ] Tests d'integration avec emails reels
- [ ] Documentation mise a jour
- [ ] Revue par Technical Partner
