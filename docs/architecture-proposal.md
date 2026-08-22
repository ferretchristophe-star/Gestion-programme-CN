# Proposition d'architecture (phase de cadrage)

Document de travail. Statut : proposition soumise a validation, non validee a ce jour.
Redige en francais car il s'agit d'un support de discussion avec l'utilisateur.
Derniere mise a jour : 2026-08-22.

---

## 1. Le principe general

L'application est un coffre-fort a programmes. Personne n'ecrit directement sur le serveur reseau.
Tout passe par l'application, qui est la seule a avoir le droit d'ecrire, de renommer et de deplacer
les fichiers. C'est ce qui garantit qu'un fichier ne peut pas etre modifie en douce sans laisser de trace.

Consequence importante : les operateurs et les programmeurs ont un acces en lecture seule sur le serveur.
Ils ne peuvent deposer ou recuperer un programme qu'en passant par l'application.

---

## 2. Les informations que l'application va gerer

Trois niveaux.

### Le referentiel (ce qui existe dans l'atelier)

| Element | Contenu |
|---|---|
| Pieces | Reference, designation, client |
| Indices de plan | L'indice A, B, C de chaque piece |
| Machines | Nom, type de directeur de commande (Fanuc, Siemens, Heidenhain, Num), extension de fichier utilisee |
| Utilisateurs | Nom, role (programmeur, operateur, regleur, qualite, administrateur) |

### Le programme (la fiche d'identite)

Une fiche unique par combinaison **Piece + Indice de plan + Phase + Machine**.
L'application empeche physiquement d'en creer deux identiques.
Cette fiche ne contient pas de fichier, elle contient l'historique.

### Les versions (l'historique)

Sous chaque fiche programme, la liste de toutes les versions successives. Chaque version enregistre :

- son indice (`ZZ` ou `01`, `02`, `03`...) ;
- son statut (en attente, en vigueur, remplacee, rejetee) ;
- son canal d'origine (bureau ou pied de machine) ;
- qui l'a deposee, quand ;
- qui l'a validee, quand ;
- le motif de la modification (champ obligatoire, c'est ce que l'auditeur EN 9100 demande en premier) ;
- une empreinte numerique du fichier.

L'empreinte numerique est une sorte d'ADN calcule a partir du contenu du fichier. Si quelqu'un modifie
ne serait-ce qu'une virgule dans le programme, l'empreinte change. L'application peut donc verifier a
tout moment que les fichiers archives sont bien intacts, et alerter si l'un d'eux a ete touche en dehors
de l'application.

---

## 3. L'arborescence sur le serveur reseau

Separation physique du valide et du non valide. C'est la protection la plus efficace : un operateur qui
navigue dans le dossier de production ne peut pas tomber sur un fichier `_ZZ`.

```
\\SERVEUR\CNC\
|
+-- PRODUCTION\                    (lecture seule pour l'atelier)
|   +-- TOUR_HAAS_01\              (une machine)
|       +-- PART123\               (une piece)
|           +-- IND_B\             (un indice de plan)
|               +-- PH_10\         (une phase)
|                   +-- PROG_PART123_P10_M1_03.nc     <-- la version en vigueur, seule
|                   +-- ARCHIVE\
|                       +-- PROG_PART123_P10_M1_01.nc
|                       +-- PROG_PART123_P10_M1_02.nc
|
+-- EN_ATTENTE\                    (invisible pour les operateurs)
|   +-- TOUR_HAAS_01\ ...          (les fichiers _ZZ)
|
+-- SYSTEME\                       (journaux, sauvegardes)
```

Point cle : dans le dossier de production, il n'y a toujours qu'un seul fichier visible, celui en vigueur.
Les anciens indices basculent automatiquement dans le sous-dossier ARCHIVE. Ils restent conserves, mais on
ne peut pas les charger par erreur. C'est exactement ce que l'ISO 9001 exige au chapitre maitrise des documents.

L'arborescence commence par la machine parce que la regle est « 1 programme = 1 machine ». L'operateur trouve
son dossier immediatement, et on peut regler les droits d'acces machine par machine.

---

## 4. Le passage de ZZ a valide, concretement

1. Une version arrive en `_ZZ` dans le dossier EN_ATTENTE.
2. Une personne habilitee clique sur « Valider » dans l'application.
3. L'application calcule l'indice suivant (`01` si c'est la premiere, sinon `02`, `03`...).
4. Elle renomme le fichier et le deplace dans le dossier PRODUCTION.
5. Elle bascule l'ancienne version en vigueur dans ARCHIVE.
6. Elle verifie que l'empreinte numerique est identique avant et apres. Le fichier valide est bien, au bit
   pres, celui qui a ete eprouve sur la machine. Aucune retouche n'est possible pendant la validation.
7. Tout est ecrit dans le journal.

Pour le canal terrain, les etapes 3 a 7 s'enchainent directement au depot, sans passer par la case attente.

---

## 5. Le journal de tracabilite

Un journal non modifiable et non effacable, meme par un administrateur. Chaque ligne : date et heure,
utilisateur, poste, action, programme concerne, empreinte du fichier.

Rien n'est jamais supprime dans l'application. Un programme qui ne sert plus est passe en « obsolete »,
il reste consultable mais n'est plus telechargeable.

Recommandation complementaire : enregistrer les telechargements, c'est-a-dire savoir quel operateur a
recupere quelle version, quel jour, a quelle heure. Le jour ou une non-conformite apparait sur une piece,
c'est cette information qui permet de prouver quel programme a reellement tourne.

---

## 6. Points a trancher avant d'aller plus loin

Choix metier, pas techniques, et ils changent la structure. Classes par importance.

**a) L'indice de plan dans le nom du fichier.**
L'exemple `PROG_PART123_P10_M1_ZZ` ne contient pas l'indice de plan, alors qu'il fait partie de la cle unique.
Faut-il l'ajouter (`PROG_PART123_B_P10_M1_01`) ou le laisser porte uniquement par le dossier ?
Question liee : les directeurs de commande ont-ils une limite de longueur sur le nom de programme ?
Certaines commandes anciennes sont limitees a 8 caracteres.

**b) Le changement d'indice de plan.**
La piece passe de l'indice B a l'indice C. D'apres la regle, cela cree un nouveau programme.
Son indice repart-il a `_01`, ou continue-t-il la numerotation du programme precedent ?

**c) Qui valide ?**
Quel role a le droit de faire passer un `_ZZ` en valide ? Faut-il une double signature (par exemple le
regleur qui a eprouve, puis le responsable methodes ou qualite), ou une seule suffit ?

**d) Comment le fichier terrain entre-t-il dans l'application ?**
L'operateur le depose lui-meme via l'application depuis un poste de l'atelier, ou l'application surveille
un dossier de depot ou la machine envoie le fichier ?

**e) Les documents associes.**
Fiche de reglage, liste d'outils, photos de montage, contre-piece. Faut-il les archiver avec le programme
des la premiere version de l'application, ou est-ce pour plus tard ?

**f) Une version rejetee.**
Un `_ZZ` qui ne donne pas satisfaction : on le garde en historique avec le motif de rejet, ou on le supprime ?

**g) Le contexte technique.**
Reseau Windows avec un domaine (comptes utilisateurs de l'entreprise) ? Combien de machines CN, combien
d'utilisateurs, et un serveur est-il disponible pour heberger l'application ?
