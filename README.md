# Créer des sondages de dates Framadate avec Claude, sans saisie manuelle

*Dernière mise à jour : juillet 2026.* <!-- À ACTUALISER À CHAQUE PUSH -->

Ce projet contient une « skill » (une recette d'automatisation) pour l'assistant IA
[Claude](https://claude.ai). Elle permet de créer un sondage de dates complet sur
Framadate en décrivant simplement ce que vous voulez, en français — au lieu de
cliquer des dizaines de fois dans des formulaires.

Vous êtes arrivé ici sans connaître Framadate, Claude ou les skills ? Ce document
est écrit pour vous : tout est expliqué, dans l'ordre.

---

## 1. C'est quoi, Framadate ?

[Framadate](https://framadate.org) est un service en ligne gratuit qui sert à
trouver une date qui convient à tout le monde — l'équivalent libre et sans
publicité de Doodle. Il est édité par [Framasoft](https://framasoft.org), une
association française qui promeut le logiciel libre.

Le principe : l'organisateur crée un « sondage » listant des dates et créneaux
possibles ; chaque participant reçoit un lien, coche ses disponibilités ; tout le
monde voit le tableau des réponses. Pas de compte à créer, ni pour l'organisateur
ni pour les participants.

C'est très utilisé dans le monde associatif : plannings de permanences, réunions,
répartition de créneaux entre bénévoles.

## 2. Ce qu'apporte cette skill

Créer un sondage Framadate à la main est simple pour 3 dates. Mais pour un
planning trimestriel — par exemple 14 dates avec 2 lieux différents chacune —
c'est une quarantaine de clics et de saisies, à refaire chaque trimestre, avec le
risque de se tromper de date.

Cette skill confie le travail à Claude. Vous écrivez une phrase comme :

> « Crée un sondage avec un créneau tous les lundis de 14h à 16h30 à Guyancourt,
> du 5 janvier au 27 mars »

…et Claude calcule les dates, ouvre Framadate dans votre navigateur Chrome,
remplit les formulaires, publie le sondage et vous donne les deux liens (le lien
à partager, et le lien d'administration à garder pour vous). Le travail se fait
dans de vrais onglets de votre Chrome — généralement en arrière-plan : vous
pouvez mettre la fenêtre Chrome au premier plan pour regarder faire, et les
onglets restent ouverts à la fin, ce qui vous permet de vérifier ce qui a été
fait.

La skill est née d'un besoin réel : le planning trimestriel de permanences d'une
activité associative. Elle fonctionne pour n'importe quel sondage de dates.

## 3. Ce qu'il vous faut avant de commencer

- Un abonnement **Claude Pro** (ou supérieur) chez Anthropic.
- L'application **Claude pour ordinateur** (macOS ou Windows), avec le mode
  **Cowork** — c'est le mode où Claude peut utiliser des outils. À la date
  d'écriture, Cowork est en avant-première (« research preview ») : les menus
  peuvent évoluer.
- Le navigateur **Google Chrome**, avec l'extension **Claude in Chrome**
  installée et connectée. C'est par elle que Claude pilote Framadate — le site
  n'ayant pas d'interface pour programmes, Claude utilise un vrai navigateur,
  comme vous le feriez. **Chrome doit être lancé au moment où vous faites votre
  demande** : si l'application est fermée, Claude ne peut pas agir dessus.

## 4. Récupérer la skill

Sur cette page GitHub, repérez le fichier **`framadate.skill`** dans la liste
des fichiers, cliquez dessus, puis sur le bouton de téléchargement (« Download »).
Le fichier arrive dans votre dossier Téléchargements.

Un fichier `.skill` n'est pas un programme : c'est une simple archive (un « zip »
renommé) contenant des fichiers texte. Vous pouvez le vérifier vous-même — voir
la section 6.

## 5. Installer la skill dans Claude

Dans l'application Claude de bureau : **Settings → Capabilities → Install skill**,
puis sélectionnez le fichier `framadate.skill` téléchargé.

La skill apparaît alors dans la liste de vos capacités. C'est tout — il n'y a
rien à redémarrer, rien à configurer.

> Si votre version de l'application présente les menus différemment (le produit
> évolue), cherchez la gestion des « skills » ou « capabilities » dans les
> paramètres. Vous pouvez aussi glisser le fichier `.skill` directement dans une
> conversation Cowork : Claude vous proposera un bouton « Save skill ».

## 6. Ce que l'installation fait concrètement sur votre ordinateur

C'est le point que les modes d'emploi oublient toujours, alors le voici.

**Ce qui se passe** : l'application Claude décompresse l'archive et copie son
contenu — deux fichiers texte au format Markdown (`SKILL.md` et un fichier de
référence technique) — dans son propre dossier de configuration, sur votre
ordinateur. Rien d'autre.

**Ce qui ne se passe PAS** : aucun programme n'est installé, aucun processus ne
tourne en arrière-plan, rien ne démarre avec votre ordinateur, aucun droit
administrateur n'est demandé, rien n'est modifié ailleurs sur votre système.

**Comment ça agit** : une skill est une *recette écrite en français* que Claude
lit au moment où votre demande correspond à sa description — et seulement à ce
moment-là. Entre-temps, elle est inerte, comme une fiche de cuisine dans un
tiroir. Vous pouvez d'ailleurs ouvrir `SKILL.md` avec n'importe quel éditeur de
texte (TextEdit, Bloc-notes) et lire exactement ce qu'elle demande à Claude de
faire : il n'y a rien de caché.

**Ce qui se passe à l'usage** : les actions ont lieu dans de vrais onglets de
votre navigateur Chrome — le plus souvent en arrière-plan, sans spectacle à
l'écran. Si vous voulez suivre en direct, mettez la fenêtre Chrome au premier
plan. Dans tous les cas, les onglets utilisés restent ouverts à la fin : vous
pouvez inspecter après coup exactement ce qui a été fait, et Claude vous
demande votre accord aux étapes qui comptent (notamment la validation des dates
avant toute saisie). Vos identifiants et cookies restent dans votre navigateur.

**Pour désinstaller** : supprimez la skill au même endroit des paramètres. La
recette est retirée, il ne reste rien.

## 7. Utilisation — trois exemples

Dans tous les cas : ouvrez une conversation Cowork, assurez-vous que Chrome est
lancé avec l'extension connectée, et écrivez votre demande en français normal.

**Comment Claude sait-il qu'il faut utiliser cette skill ?** Chaque skill
installée porte une description ; quand votre demande y correspond, Claude
charge la recette. Concrètement : citer « Framadate » dans votre phrase suffit
à la déclencher de façon fiable. Une demande du type « crée un planning de
permanences avec des créneaux » fonctionne aussi, mais nommer Framadate est le
plus sûr. C'est le mécanisme actuel — si un jour la skill ne se déclenche pas,
demandez explicitement : « utilise la skill framadate ».

### Exemple 1 — Un rendez-vous unique

> « Crée un sondage Framadate pour trouver la date de notre réunion de rentrée.
> Propose le mardi 15, le mercredi 16 et le jeudi 17 septembre, de 18h à 20h,
> salle des associations. Titre : "Réunion de rentrée". Mon email :
> prenom.nom@exemple.fr »

Claude ouvre Framadate, crée le sondage avec les trois dates et le créneau,
le publie et vous rend deux liens : le **lien public** (à envoyer aux
participants) et le **lien d'administration** (à conserver — il permet de
modifier ou clore le sondage).

### Exemple 2 — À partir d'un fichier tableur (CSV)

Si votre planning existe déjà dans un tableur, exportez-le en CSV avec quatre
colonnes — date, lieu, heure de début, heure de fin :

```csv
date,lieu,debut,fin
2026-01-05,Guyancourt,14h,16h30
2026-01-05,Trappes,14h,16h30
2026-01-12,Guyancourt,14h,16h30
```

Puis joignez le fichier à la conversation et écrivez :

> « Crée le sondage Framadate à partir du fichier planning.csv ci-joint.
> Titre : "Permanences hiver 2026". Mon email : prenom.nom@exemple.fr »

Claude lit le fichier, construit un créneau par ligne, et procède comme dans
l'exemple 1.

### Exemple 3 — Une demande « complexe » en langage naturel

C'est là que la skill rend le plus service :

> « Pour la zone C, de la fin des vacances de Noël au début des vacances
> d'hiver exclus, hors jours fériés, crée un créneau tous les lundis de 14h à
> 16h30 à Guyancourt, tous les lundis de 14h à 16h30 à Trappes, et tous les
> vendredis de 9h30 à 12h à Élancourt. »

Voici ce qui se passe, dans l'ordre :

1. **Claude vérifie les dates sur les sources officielles** (calendrier
   scolaire sur legifrance.gouv.fr ou service-public.gouv.fr, jours fériés).
   La skill le lui impose : il n'a pas le droit de citer un calendrier de
   mémoire, car les calendriers futurs changent.
2. **Claude vous montre la liste complète des dates retenues** — et celles
   qu'il a exclues (férié, vacances) avec la raison. Il vous pose les questions
   utiles : si la rentrée tombe un lundi, faut-il une permanence dès ce
   jour-là ? Quel titre pour le sondage ? Quel email ?
3. **Vous validez.** Rien n'est saisi dans Framadate avant votre accord.
4. Claude remplit alors le formulaire — ici environ 20 créneaux — et vous
   rend les deux liens.

Le point 2 est le garde-fou important : vous vérifiez 15 lignes en 30 secondes,
au lieu de découvrir une date fausse quand les bénévoles ont déjà répondu.

## 8. Limites connues

- La skill vise `beta.framadate.org` (la nouvelle version de Framadate). Si
  Framasoft fait évoluer ses formulaires, la skill peut nécessiter une mise à
  jour — c'est un fichier texte, elle est facile à adapter.
- Claude Cowork est en avant-première : noms de menus et boutons peuvent bouger.
- Comme toute automatisation de navigateur, une page qui met du temps à charger
  peut demander à Claude un deuxième essai. Il vous le dira.

## Licence et contributions

Ce projet est sous **licence MIT** (voir le fichier `LICENSE`), du nom du
Massachusetts Institute of Technology qui l'a popularisée. En clair : vous
pouvez utiliser, copier, modifier et redistribuer cette skill librement, y
compris dans un cadre commercial. La seule obligation est de **conserver la
notice de copyright — « Copyright (c) 2026 Alain Lequin » — et le texte de la
licence** dans toute copie. Le nom de l'auteur voyage donc avec le code.

Les retours (une étape qui coince, une formulation qui ne déclenche pas la
skill) sont bienvenus via les « Issues » GitHub — même sans savoir coder :
décrire le problème suffit.
