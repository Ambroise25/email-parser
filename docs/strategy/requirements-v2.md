# Exigences Metier v2 - Retour Assistante de Direction

**Date** : 20 janvier 2026  
**Source** : Entretien avec l'assistante de direction

---

## Contexte operationnel

### Volume d'emails
- **40 a 50 emails par jour** sur la boite mail
- Tous ne sont pas des demandes d'intervention (filtrage necessaire)

### Identification des demandes
Mots-cles frequents dans l'objet des emails :
- "demande de devis"
- "OS" (Ordre de Service)
- "Ordre de service"
- "demande d'intervention"

> Note : Les objets ne sont pas standardises, le LLM devra analyser le contenu.

---

## Gestion des pieces jointes

### Types de fichiers
- **Images** (photos de fuites, degats, etc.)
- **PDF** (ordres de service, devis, plans)

### Exigence
- Les pieces jointes contiennent souvent des informations importantes
- Elles doivent etre **conservees et consultables** apres traitement de l'email

### URLs dans les emails
- Certaines informations sont contenues dans des **liens URL** dans le corps du mail
- Ces URLs doivent etre extraites et stockees

---

## Archivage des emails

### Workflow
1. Email recu dans INBOX
2. Email traite par le systeme
3. Email deplace vers le dossier **"Email traite"**

### Avantage
- Separation claire entre emails traites et non traites
- Pas de retraitement accidentel

---

## Identification du corps de metier

### Changement important
Contrairement a la version initiale, **tous les emails ne concernent pas l'etancheite**.

### Metiers a identifier
| Metier | Exemples de demandes |
|--------|---------------------|
| Etancheite | Fuite toiture, infiltration terrasse |
| Plomberie | Fuite canalisation, WC bouche |
| Couverture | Tuiles cassees, gouttiere |
| Autre | A preciser selon contexte |

### Action LLM
Le parser doit **analyser et identifier** le corps de metier concerne.

---

## Evaluation de l'urgence

### Echelle d'urgence
| Niveau | Critere | Exemple |
|--------|---------|---------|
| **Urgent** | Grosse fuite, degats en cours | "Eau qui coule dans l'appartement" |
| **Normal** | Demande de chiffrage, pas de fuite | "Devis pour refection etancheite" |
| **Faible** | Preventif, pas de dommage | "Controle annuel" |

### Logique
- Presence de fuite = urgence elevee
- Simple demande de devis = urgence normale

---

## Suivi des demandes (Statuts)

### Statuts possibles
| Statut | Description |
|--------|-------------|
| **A contacter** | Demande recue, premier contact a faire |
| **A relancer** | Contact fait, attente reponse client |
| **Rdv non pris** | Discussion en cours, rdv pas encore fixe |

> Note : Ces statuts sont geres manuellement par l'utilisateur dans le tableau de bord.

---

## Interface utilisateur - Tableau de bord

### Vue principale : Tableur
Une ligne par demande (= un email traite)

### Colonnes du tableur
| Colonne | Description |
|---------|-------------|
| Syndic | Nom du syndic |
| Adresse syndic | Adresse du cabinet |
| Adresse intervention | Lieu de l'intervention |
| Metier | Etancheite, Plomberie, Couverture, etc. |
| Urgence | Urgent / Normal / Faible |
| Statut | A contacter / A relancer / Rdv non pris |

### Details cliquables
Chaque ligne est **cliquable** et affiche plus d'informations :

| Information | Description |
|-------------|-------------|
| Codes d'acces | Digicode, interphone, cle gardien |
| Personnes a contacter | Nom, telephone, email du gestionnaire |
| Lien Google Earth | Lien vers l'adresse sur la carte |
| Recap demande | Resume rapide de la demande |
| Pieces jointes | Acces aux images/PDF attaches |

---

## Resume des impacts sur l'architecture

### Nouveaux composants necessaires
1. **Stockage fichiers** : Pour les pieces jointes (images, PDF)
2. **Base de donnees** : Pour stocker les demandes et statuts
3. **Frontend** : Tableau de bord avec vue tableur et details
4. **Gestion dossiers IMAP** : Deplacer vers "Email traite"

### Evolution du parser LLM
- Detecter le metier (pas seulement etancheite)
- Evaluer l'urgence
- Extraire les URLs
- Identifier les codes d'acces

---

## Integration avec le CRM Symfony

### Logique d'integration
Chaque demande extraite d'un email doit etre **ajoutee au CRM Symfony** (Extranet).

### Gestion des biens
| Situation | Action |
|-----------|--------|
| Bien **deja existant** dans le CRM | Ajouter la demande au bien existant |
| Bien **nouveau** (adresse non connue) | Creer le bien dans le CRM, puis ajouter la demande |

### Identification du bien
Un bien est identifie par les champs suivants :

| Champ | Exemple |
|-------|---------|
| Numero(s) de rue | 12, 12-14, 12 bis |
| Nom de rue | Avenue des Champs-Elysees |
| Ville | Paris |
| Code postal | 75008 |
| Nom de la copropriete | Residence Les Jardins |

Le LLM doit extraire ces 5 champs pour chaque demande afin de permettre la recherche/creation du bien dans le CRM.

---

## Tableau de bord - Precisions

### Integration a l'Extranet
- Le tableau de bord sera **integre a l'Extranet Symfony** a terme
- Accessible a **toute la societe** (pas d'authentification specifique, herite de l'Extranet)

### Gestion de l'urgence
- **Pas de notification** pour les demandes urgentes
- L'urgence est **mise en valeur visuellement** dans le tableau (couleur, icone, tri)

---

## Questions ouvertes

- [x] ~~Faut-il un systeme de notification quand un email urgent arrive ?~~ → Non, juste mise en valeur visuelle
- [x] ~~Les statuts doivent-ils etre synchronises avec le CRM Symfony existant ?~~ → Oui, integration complete
- [x] ~~Qui a acces au tableau de bord ?~~ → Toute la societe via l'Extranet
- [x] ~~Retention des pieces jointes~~ → On sauvegarde sans limite pour le moment
- [x] ~~Comment identifier un bien existant ?~~ → Par numero, rue, ville, code postal, nom copropriete

---

## Prochaines etapes

1. ~~Valider ces exigences avec le Business Partner~~ → En cours
2. Definir le contrat API avec le CRM Symfony
3. Mettre a jour l'architecture technique
4. Reviser les user stories existantes
