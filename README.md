# 🎬 Test Technique Backend — MiniSeries

## Contexte

**MiniSeries** est une plateforme de short dramas (épisodes de 1 à 3 minutes).
Les utilisateurs disposent d'un portefeuille de **coins** pour débloquer les épisodes
premium. Vous êtes chargé·e de développer le service backend qui gère **l'accès aux
épisodes** selon les règles métier décrites ci-dessous.

> **Durée estimée** : 1h30 – 2h00
> 
> **Stack** : TypeScript, Node.js, framework HTTP au choix (Express, Fastify, Koa, Hono…)
> 
> **Stockage** : in-memory (pas de base de données requise)
> 
> **Livrable** : un repo Git avec un historique de commits lisible

---

## Données

Les fichiers `data/catalog.json` et `data/users.json` sont fournis.
Ils constituent votre source de données initiale à charger en mémoire au démarrage.

---

## Règles métier

### Accès aux épisodes

- **Épisodes `free`** : accessibles à tous, sans condition.
- **Épisodes `premium`** : nécessitent un déblocage via coins.
- **Ordre séquentiel** : un épisode premium ne peut être débloqué que si **tous les
  épisodes précédents** de la même série ont été :
  - soit **visionnés à ≥ 80%** (présents dans le `watchHistory` avec `completedPercent >= 80`)
  - soit **déjà débloqués** (présents dans `unlockedEpisodes`)

### Tarification

| Mode            | Coût                                                                  |
| --------------- | --------------------------------------------------------------------- |
| **Unitaire**    | **10 coins** par épisode                                              |
| **Lot (batch)** | Tous les épisodes premium restants d'une série : **7 coins/épisode**  |

> Le coût du batch = nombre d'épisodes premium restants × 7.

### Promotion "Happy Hour"

- Entre **18h00 et 20h00 UTC** (inclus–exclus), le coût **unitaire** passe à **8 coins**.
- Le coût **lot** reste à **7 coins/épisode**.
- L'heure est déterminée par le moment de la requête.

---

## Endpoints à implémenter

### A. `GET /users/:userId/series/:seriesId/feed`

Retourne la liste ordonnée des épisodes d'une série pour un utilisateur donné.

**Réponse attendue** (exemple de structure) :

```json
{
  "seriesId": "series-1",
  "seriesTitle": "Le Piège",
  "episodes": [
    {
      "episodeId": "ep-101",
      "number": 1,
      "title": "L'appel",
      "durationSec": 120,
      "status": "free",
      "completedPercent": 100,
      "unlockCost": null
    },
    {
      "episodeId": "ep-103",
      "number": 3,
      "title": "Le doute",
      "durationSec": 140,
      "status": "unlocked",
      "completedPercent": 45,
      "unlockCost": null
    },
    {
      "episodeId": "ep-104",
      "number": 4,
      "title": "La chute",
      "durationSec": 110,
      "status": "locked",
      "completedPercent": 0,
      "unlockCost": 10
    }
  ],
  "batchUnlockCost": 14
}
```

> `batchUnlockCost` = nombre d'épisodes premium non débloqués × 7.
> `unlockCost` tient compte du Happy Hour si applicable.

---

### B. `POST /users/:userId/unlock`

Déblocage **unitaire** d'un épisode.

**Body** :

```json
{ "episodeId": "ep-104" }
```

**Réponse succès** :

```json
{
  "unlockedEpisode": "ep-104",
  "cost": 10,
  "newBalance": 40
}
```

**Erreurs possibles** :

| Code                    | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `EPISODE_NOT_FOUND`     | L'épisode n'existe pas                                       |
| `ALREADY_UNLOCKED`      | L'épisode est déjà débloqué ou gratuit                       |
| `INSUFFICIENT_BALANCE`  | Solde de coins insuffisant                                   |
| `PREREQUISITE_NOT_MET`  | Les épisodes précédents ne sont pas complétés/débloqués      |
| `USER_NOT_FOUND`        | L'utilisateur n'existe pas                                   |

---

### C. `POST /users/:userId/unlock-batch`

Déblocage **en lot** de tous les épisodes premium restants d'une série.

**Body** :

```json
{ "seriesId": "series-1" }
```

**Réponse succès** :

```json
{
  "unlockedEpisodes": ["ep-104", "ep-105"],
  "costPerEpisode": 7,
  "totalCost": 14,
  "newBalance": 36
}
```

**Erreurs possibles** :

| Code                      | Description                                              |
| ------------------------- | -------------------------------------------------------- |
| `SERIES_NOT_FOUND`        | La série n'existe pas                                    |
| `NO_EPISODES_TO_UNLOCK`   | Tous les épisodes premium sont déjà débloqués            |
| `INSUFFICIENT_BALANCE`    | Solde de coins insuffisant                               |
| `USER_NOT_FOUND`          | L'utilisateur n'existe pas                               |

---

### D. `GET /users/:userId/next`

Pour chaque série dans laquelle l'utilisateur a de l'activité (au moins un épisode
vu ou débloqué), retourne le **prochain épisode à regarder**.

Le prochain épisode est le premier épisode (dans l'ordre) qui n'a **pas été
complété à ≥ 80%**.

**Réponse attendue** :

```json
{
  "nextEpisodes": [
    {
      "seriesId": "series-1",
      "seriesTitle": "Le Piège",
      "episode": {
        "episodeId": "ep-103",
        "number": 3,
        "title": "Le doute",
        "status": "unlocked",
        "completedPercent": 45
      }
    }
  ]
}
```

> Si une série est entièrement complétée (tous les épisodes ≥ 80%), elle n'apparaît
> pas dans la liste (ou avec un flag `completed: true`, à votre discrétion).

---

## Tests unitaires

Écrivez au minimum **3 tests** couvrant la logique métier (framework au choix) :

Vous êtes libre d'en ajouter d'autres si le temps le permet.

---

## Ce qui compte pour nous

- La **clarté et la maintenabilité** de votre code importent plus que
  le nombre de features terminées.
- Votre manière de **structurer le projet** et de **séparer les
  responsabilités** sera regardée avec attention.
- Nous préférons un code **bien pensé et bien testé** à un code
  qui "fonctionne" sans rigueur.
- L'entretien qui suit portera sur vos **choix de conception** : soyez prêt·e à les expliquer et les défendre.

---

## Ce qui n'est PAS attendu

- ❌ Authentification / autorisation
- ❌ Base de données réelle
- ❌ Docker / CI-CD
- ❌ Documentation OpenAPI
- ❌ Interface frontend

---

## Remarques

- Explicitez les choix que vous faites memes les choix de librairies et technologies employés
- Privilégiez la **qualité** à la quantité : un endpoint bien conçu vaut mieux que
  quatre bâclés.
- Un entretien de 30 à 45 minutes suivra pour discuter de vos choix d'architecture
  et d'évolutions possibles.

Bon courage ! 🚀