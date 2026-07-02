---
name: framadate
description: >
  Crée automatiquement un sondage de dates sur beta.framadate.org en pilotant Chrome.
  Utiliser dès que l'utilisateur demande de créer un sondage Framadate, un planning
  de séances, de permanences ou de réunions avec des créneaux et des lieux, ou toute
  variante : "crée un sondage Framadate", "nouveau planning de septembre à novembre",
  "permanences à X et Y du tant au tant". Déclencher aussi pour des descriptions en
  langage naturel du type "tous les mardis de 9h à 11h30 du 2 sept au 28 oct",
  ou quand un fichier CSV/Excel de planning est fourni.
---

# Skill : Création de sondage de dates sur Framadate

Ce skill pilote Chrome pour créer un sondage de dates sur beta.framadate.org,
évitant la saisie manuelle répétitive (ex. : 7 semaines × 2 lieux = ~42 clics minimum).

Framadate n'a pas d'API REST — le skill navigue le vrai navigateur via Chrome MCP.
La gestion des cookies et tokens CSRF est transparente (le navigateur s'en charge).

## Étape 0 : Collecter les informations

Deux modes d'entrée possibles — détecter lequel s'applique :

### Mode A — Fichier CSV (ou Excel)

Si l'utilisateur fournit un fichier CSV ou Excel, le lire avec les outils Read ou Bash.

Format CSV attendu (une ligne = un créneau) :
```csv
date,lieu,debut,fin
2026-09-01,Saint-Cyr,9h,11h30
2026-09-01,Bois d'Arcy,9h,11h30
2026-09-08,Saint-Cyr,9h,11h30
```

Pour un Excel (.xlsx), utiliser Bash + Python :
```bash
python3 -c "
import openpyxl, csv, sys
wb = openpyxl.load_workbook('fichier.xlsx')
ws = wb.active
for row in ws.iter_rows(min_row=2, values_only=True):
    print(','.join(str(c) for c in row))
"
```

Construire ensuite la structure dates/slots à partir des lignes lues.

### Mode B — Langage naturel

Extraire du message (ou demander ce qui manque) :

| Information | Exemple |
|---|---|
| Titre du sondage | "Permanences Automne 2026" |
| Récurrence | "tous les mardis" |
| Période | "du 2 septembre au 28 octobre 2026" |
| Lieux | "Saint-Cyr, Bois d'Arcy" |
| Horaires | "9h à 11h30" (mêmes pour tous les lieux) |
| Email organisateur | demander si absent |

Calculer la liste complète des dates en appliquant la récurrence sur la période.

#### Règles de calcul des dates — OBLIGATOIRES

**1. Jamais de dates de mémoire.** Si la période est exprimée par référence au
calendrier scolaire ("fin des vacances d'été", "avant la Toussaint", "zone C") ou
aux jours fériés, TOUJOURS vérifier les dates via une recherche web sur une source
officielle avant de calculer : arrêté ministériel sur legifrance.gouv.fr,
service-public.gouv.fr ou education.gouv.fr. Les calendriers scolaires futurs sont
postérieurs à la connaissance interne du modèle — une date supposée est probablement fausse.

Rappels utiles : la zone C = académies de Versailles, Paris, Créteil, Montpellier,
Toulouse (demander la zone si elle n'est pas précisée). Les vacances de Toussaint, Noël et d'été
sont communes aux trois zones ; seules les vacances d'hiver et de printemps diffèrent.

**2. Bornes explicites.** Convention : "fin des vacances de X" = jour de la rentrée
inclus ; "début des vacances de Y exclus" = dernière date strictement antérieure au
premier jour des vacances. Cas particulier à faire confirmer par l'utilisateur : si le jour
de rentrée tombe sur un jour de séance (ex. rentrée un mardi), demander si la séance
a lieu dès ce jour-là ou seulement la semaine suivante.

**3. Jours fériés.** Si "hors jours fériés" est demandé, vérifier la liste des fériés
français sur la période (service-public.gouv.fr) et exclure les dates concernées.
Noter dans la restitution les dates exclues et pourquoi (traçabilité).

**4. Restituer le calcul avant de saisir.** Afficher la liste complète des dates
retenues (et exclues) et attendre la validation de l'utilisateur AVANT de lancer la
navigation Chrome. La saisie de 20 créneaux erronés coûte plus cher que la pause.

### Dans les deux cas, construire :
```
dates     = ['2026-09-01', '2026-09-08', ...]   # YYYY-MM-DD, triées
slots     = {date: ['Saint-Cyr 9h-11h30', "Bois d'Arcy 9h-11h30"], ...}
closed_at = dernière_date + 14 jours
```

Demander le titre du sondage et l'email organisateur s'ils ne sont pas fournis.

## Navigation Chrome — 4 étapes

Charger les outils Chrome MCP via ToolSearch si nécessaire :
`select:mcp__Claude_in_Chrome__navigate,mcp__Claude_in_Chrome__read_page,mcp__Claude_in_Chrome__form_input,mcp__Claude_in_Chrome__browser_batch,mcp__Claude_in_Chrome__get_page_text`

---

### Étape 1 — Créer le sondage

```
navigate → https://beta.framadate.org/polls/new?type=date&flow=on
read_page → repérer les refs des champs du formulaire "poll"
form_input → remplir :
  poll[title]       = titre
  poll[authorName]  = nom de l'utilisateur (demander s'il est inconnu)
  poll[authorEmail] = email
  poll[closedAt]    = closed_at  (format YYYY-MM-DD, c'est un input[type=date])
clic sur le bouton "Suivant"
```

Vérifier que l'URL change vers `/polls/{id}/{token}/dates`. Noter `{id}` et `{token}`.

---

### Étape 2 — Saisir les dates

La page affiche un calendrier interactif avec un **bouton de bascule clavier** (icône
clavier ou texte "Saisie au clavier"). Ce bouton révèle des `input[type=date]` directs
sous chaque entrée — c'est la méthode la plus fiable.

**Méthode clavier (préférée) :**
```
1. read_page → trouver le bouton de bascule clavier, cliquer dessus
2. Pour chaque date à saisir :
   - Si c'est la première : le champ poll_dates[dates][0][value] est déjà présent
   - Pour les suivantes : cliquer "Ajouter une date" puis remplir le nouveau champ
   - form_input sur poll_dates[dates][N][value] = 'YYYY-MM-DD'
3. Cliquer "Suivant"
```

**Méthode alternative (si la bascule clavier est absente) :**
Essayer de remplir directement les `input[type=date]` cachés sous le calendrier via
`form_input` — ils sont dans le DOM même si le calendrier les masque visuellement.
En dernier recours, cliquer les dates dans le calendrier (naviguer les mois si besoin).

---

### Étape 3 — Saisir les créneaux (horaires)

La page affiche des champs texte par date. Les noms de champs suivent ce pattern :
`poll_slots[dates][I][proposals][J][label]` (I = index date, J = index créneau).

**Si tous les créneaux sont identiques** : chercher le bouton "Appliquer les mêmes
horaires à toutes les dates" — il duplique automatiquement les créneaux de la première
date sur toutes les autres. Remplir seulement la première date, puis utiliser ce bouton.

**Sinon** : pour chaque date I, remplir chaque créneau J :
```
form_input → poll_slots[dates][0][proposals][0][label] = "Saint-Cyr 9h-11h30"
form_input → poll_slots[dates][0][proposals][1][label] = "Bois d'Arcy 9h-11h30"
... (répéter pour chaque date)
```

Cliquer "Suivant".

---

### Étape 4 — Publier

```
read_page → vérifier le résumé affiché
clic sur "Publier" (ou "Suivant")
get_page_text → lire la page /complete
```

Extraire le lien public depuis `<input id="poll-public-link" value="...">`.
L'URL admin est : `https://beta.framadate.org/polls/{id}/{token}/admin`

---

## Résultat à restituer

```
✓ Planning créé !

Lien à partager avec les bénévoles :
  https://beta.framadate.org/polls/...

Lien admin (organisateur uniquement, à ne pas diffuser) :
  https://beta.framadate.org/polls/{id}/{token}/admin
```

## En cas d'erreur

- **La page ne change pas après "Suivant"** : un champ obligatoire est vide ou invalide.
  Lire `get_page_text` pour voir les messages d'erreur Symfony affichés.
- **closedAt refusé** : vérifier que la date est dans le futur et au format YYYY-MM-DD.
- **Les champs de slots ont des indices inattendus** : faire un `read_page` sur la page
  /slots et lire les attributs `name` réels avant de remplir (voir `references/form-structure.md`).

## Référence technique

Pour la structure complète des formulaires Symfony (noms de champs, hiérarchie),
lire `references/form-structure.md`.

---

*Skill créée par Alain Lequin — licence MIT (voir fichier LICENSE). Le nom de l'auteur et cette notice doivent être conservés dans toute copie.*
