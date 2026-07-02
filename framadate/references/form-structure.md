# Structure des formulaires Framadate (Pollaris/Symfony)

> Source : analyse du code source framagit.org/pollaris/pollaris — juin 2026

## Vue d'ensemble des 4 étapes

```
POST /polls/new?type=date&flow=on          → crée le sondage, redirect vers /dates
POST /polls/{id}/{token}/dates?flow=on     → enregistre les dates, redirect vers /slots
POST /polls/{id}/{token}/slots?flow=on     → enregistre les créneaux, redirect vers /summary
POST /polls/{id}/{token}/summary?flow=on   → finalise, redirect vers /complete
```

Chaque POST contient un token CSRF dans un champ caché. Le navigateur gère tout ça
automatiquement — ces informations sont utiles pour déboguer ou écrire des tests.

---

## Étape 1 : Formulaire `poll`

**Classe PHP** : `App\Form\PollForm`

| Champ HTML | Type Symfony | Valeur attendue |
|---|---|---|
| `poll[title]` | TextType | Titre du sondage |
| `poll[description]` | TextareaType | Description (optionnel) |
| `poll[closedAt]` | DateType (single_text) | YYYY-MM-DD |
| `poll[authorName]` | TextType | Nom de l'organisateur |
| `poll[authorEmail]` | EmailType | Email de l'organisateur |
| `poll[_token]` | hidden (CSRF) | Extrait de la page |

---

## Étape 2 : Formulaire `poll_dates`

**Classe PHP** : `App\Form\PollDatesForm` → `CollectionType(DateForm)`

| Champ HTML | Type | Valeur |
|---|---|---|
| `poll_dates[dates][0][value]` | DateType (single_text) | YYYY-MM-DD |
| `poll_dates[dates][1][value]` | DateType (single_text) | YYYY-MM-DD |
| `poll_dates[dates][N][value]` | … | … |
| `poll_dates[_token]` | hidden (CSRF) | Extrait de la page |

Les indices 0, 1, 2… sont séquentiels. Autant de champs que de dates à saisir.

L'interface calendrier masque ces inputs visuellement, mais ils sont toujours dans le
DOM (`input[type=date]`). La bascule "saisie au clavier" les rend visibles.

---

## Étape 3 : Formulaire `poll_slots`

**Classe PHP** : `App\Form\PollSlotsForm` → `CollectionType(SlotsForm)` → `CollectionType(SlotType)`

Hiérarchie : dates → proposals → label

| Champ HTML | Description |
|---|---|
| `poll_slots[dates][I][proposals][J][label]` | Label du créneau J de la date I |
| `poll_slots[_token]` | CSRF token |

Exemple pour 3 dates × 2 lieux :
```
poll_slots[dates][0][proposals][0][label] = "Saint-Cyr 9h-11h30"
poll_slots[dates][0][proposals][1][label] = "Bois d'Arcy 9h-11h30"
poll_slots[dates][1][proposals][0][label] = "Saint-Cyr 9h-11h30"
poll_slots[dates][1][proposals][1][label] = "Bois d'Arcy 9h-11h30"
poll_slots[dates][2][proposals][0][label] = "Saint-Cyr 9h-11h30"
poll_slots[dates][2][proposals][1][label] = "Bois d'Arcy 9h-11h30"
```

Label max : 200 caractères.

Le bouton "Appliquer les mêmes horaires à toutes les dates" (modal JS) permet de
copier les proposals de la date 0 sur toutes les autres — utiliser si disponible.

---

## Étape 4 : Formulaire `poll_summary`

**Classe PHP** : `App\Form\PollSummaryForm` — vide (aucun champ métier)

Seul le CSRF token est soumis : `poll_summary[_token]`.

---

## Page /complete

Après finalisation, la page `/polls/{id}/{token}/complete` contient :

```html
<input id="poll-public-link" type="url" value="https://beta.framadate.org/polls/{slug}" readonly>
<a href="/polls/{slug}">Voir le sondage</a>
```

Le `{slug}` est différent du `{id}` — c'est une propriété séparée de l'entité Poll.
L'URL admin reste : `https://beta.framadate.org/polls/{id}/{token}/admin`
