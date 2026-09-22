# Règles fiscales de « Mon Patrimoine »

Ce dépôt contient **un seul fichier utile** : `parametres-fiscaux.json`, la liste des règles
fiscales et bancaires françaises qu'utilise l'application *Mon Patrimoine* — barème de l'impôt
sur le revenu, décote, plafonnement du quotient familial, prélèvements sociaux, PASS, plafonds
des livrets et du PEA, fiscalité de l'assurance vie, droits de succession, taux réglementés.

**Il ne contient aucune donnée personnelle** : ni patrimoine, ni compte, ni nom, ni code — rien
que des chiffres publics, repris des textes officiels (lois de finances, BOFiP,
service-public.fr, Banque de France). Il est public uniquement pour que l'application puisse le
lire, sans jamais donner accès à quoi que ce soit d'autre.

## À quoi ça sert

Jusqu'à la version 6.0, ces nombres étaient écrits dans le code de l'application : les changer
demandait de recompiler et de réinstaller le logiciel. Deux conséquences fâcheuses :

- une application installée en 2026 et jamais mise à jour continuait de calculer l'impôt avec
  le barème 2026 en 2029, **sans le dire**. Un chiffre faux mais plausible ne se voit pas ;
- une simple révision du taux du Livret A (deux fois par an) imposait une nouvelle version.

Depuis la version 6.1, l'application lit ce fichier **à l'ouverture du profil**, le vérifie, et
affiche le millésime des règles qu'elle applique. Mettre à jour la fiscalité, c'est désormais
modifier ce fichier — rien à réinstaller.

## Comment l'application se protège

Ce fichier pilote des montants d'impôt : c'est une porte d'entrée, et elle est traitée comme telle.

- il ne contient **que des nombres**, jamais de code, et il est lu par `JSON.parse` — jamais exécuté ;
- il est **validé avant d'être appliqué** : bornes de chaque taux, tranches croissantes, dernière
  tranche ouverte, cohérence des seuils, interdiction de revenir à un millésime plus ancien.
  Au moindre problème, le fichier est **refusé** et l'application garde les règles qu'elle
  embarque, en le signalant. Refuser est toujours plus sûr qu'appliquer ;
- une panne de réseau ne change **aucun chiffre** : l'application donne exactement les mêmes
  montants hors ligne, elle indique simplement un millésime plus ancien ;
- si les règles publiées supposent une règle *nouvelle* (et pas seulement un nombre qui change),
  le champ `versionAppliMini` fait que les anciennes versions refusent le fichier et invitent à
  mettre l'application à jour — au lieu de calculer faux.

## Format

| Champ | Rôle |
|---|---|
| `schema` | version du format ; change seulement si la structure change |
| `millesime` | dernière année d'imposition couverte |
| `publieLe` | date de publication (`AAAA-MM-JJ`) — c'est elle qui décide si le fichier est plus récent |
| `versionAppliMini` | version minimale de l'application capable d'appliquer ces règles |
| `bareme` | tranches de l'impôt, **par année d'imposition** ; la dernière tranche a `max: null` |
| `ir` | décote et plafonnement du quotient familial, par année |
| `ps` | prélèvements sociaux ; la clé est la **première année** où le taux s'applique |
| `pass`, `plafonds`, `assuranceVie`, `succession`, `livrets` | les autres montants |

Les années passées ne sont **jamais** remplacées : quelqu'un qui recalcule ses revenus 2025 en
2030 doit retrouver les règles de 2026.

Le fichier est fabriqué et vérifié par
`scripts/parametres/publier-parametres.js`, qui utilise exactement le même code de validation
que l'application — deux copies auraient fini par diverger.

---

*Application personnelle, gratuite et locale. Ces chiffres sont fournis à titre indicatif et ne
constituent ni un conseil fiscal, ni un substitut à votre déclaration.*
