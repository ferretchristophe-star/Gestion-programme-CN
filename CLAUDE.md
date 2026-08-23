## Approach
- Read existing files before writing. Don't re-read unless changed.
- Thorough in reasoning, concise in output.
- Skip files over 100KB unless required.
- No sycophantic openers or closing fluff.
- No emojis or em-dashes.
- Do not guess APIs, versions, flags, commit SHAs, or package names. Verify by reading code or docs before asserting.

## Méthode de travail (règles absolues de l'utilisateur)
- Phase de cadrage en cours : discussion sur la base, l'architecture et les fonctionnalités uniquement.
- Interdiction d'écrire ou générer du code tant que l'autorisation explicite n'a pas été donnée.
- À chaque étape clé, produire un résumé clair et synthétique de ce qui a été validé.
- Ne démarrer l'écriture du code qu'après un « Validé » ou accord explicite de l'utilisateur sur le résumé.
- Langue des échanges (discussions, explications, résumés) : exclusivement en français.
- Langue du projet (code, documentation technique, commentaires, noms de fichiers/variables) : exclusivement en anglais.
- L'utilisateur n'est pas développeur : expliquer simplement, sans jargon technique ni détails d'implémentation non demandés. Raisonner en termes métier (atelier, qualité, traçabilité).

## Contexte projet
- Objectif : application de gestion des programmes FAO (Fabrication Assistée par Ordinateur) des machines à commande numérique (CN) d'un atelier d'usinage.
- Enjeu principal : traçabilité documentaire conforme ISO 9001 et EN 9100 (aéronautique) — maîtrise des documents et enregistrements, historique des révisions, approbation avant utilisation en production, association programme/pièce/machine/opérateur/OF.
- Détails à compléter au fur et à mesure des échanges avec l'utilisateur.

### Fonctionnalités clés validées
1. Archivage des programmes validés : l'application ne stocke que les versions validées des programmes machine.
2. Gouvernance et indiçage (versioning) :
   - Incrémentation automatique de l'indice/version à chaque modification enregistrée.
   - Historique complet des versions conservé pour un même programme.
3. Gestion multi-machines :
   - Association de chaque programme et de ses révisions aux machines correspondantes.
   - Filtrage et recherche par machine et par indice.

### Règles métier strictes validées

#### Workflow de validation hybride (deux canaux d'entrée)
- Canal FAO / Bureau d'études (différé) : import d'un programme non éprouvé sur machine. Statut initial « En attente de validation ». Inutilisable en série de production avant l'acte de validation.
- Canal Terrain / Pied de machine (direct) : mise au point ou modification faite sur le directeur de commande. Statut initial « Validé » (archivage immédiat d'une version opérationnelle).

#### Nomenclature des indices
- Version d'essai / en attente de validation : suffixe `_ZZ` (ex. `PROG_PART123_P10_M1_ZZ`).
- Version validée (production) : indice numérique incrémental à 2 chiffres `_01`, `_02`, `_03`...
- Au passage « En attente de validation » vers « Validé », le système convertit automatiquement l'indice `_ZZ` vers l'indice validé suivant (`_01` à la création initiale, `_02` si révision, etc.).

#### Clé unique métier
- Un programme CN est obligatoirement lié au quadruplet : Référence pièce + Indice de plan + Phase de gamme + Machine spécifique.
- Règle d'or : 1 programme = 1 machine. Deux machines, même identiques, ne partagent jamais le même enregistrement ni le même fichier. Une adaptation pour une autre machine génère un programme dédié.

#### Stockage physique
- L'application gère les métadonnées et les accès, elle ne stocke pas les fichiers dans sa base de données.
- Elle crée et administre automatiquement une arborescence de dossiers standardisée sur un serveur réseau centralisé.
- L'accès et l'écriture sur l'emplacement réseau racine sont sécurisés (compte de service dédié administré par l'application).

#### Traçabilité
- Journal d'opérations complet, horodatage, identification de l'utilisateur ayant déposé ou validé le fichier.

#### Identifiant technique unique
- À l'import d'un programme dans l'application, un identifiant unique (1, 2, 3, 4...) est attribué automatiquement.
- Cet identifiant est permanent : jamais modifiable, jamais réattribuable, jamais écrasé.
- Portée exacte (par fiche programme ou par version déposée) à confirmer avec l'utilisateur.
