# 🎬 Backend Technical Test — MiniSeries

## Context

**MiniSeries** is a short drama platform (episodes of 1 to 3 minutes).
Users have a **coins** wallet to unlock premium episodes.
You are tasked with developing the backend service that manages **episode access**
according to the business rules described below.

> **Estimated duration**: 1h30 – 2h00
>
> **Stack**: TypeScript, Node.js, HTTP framework of your choice (Express, Fastify, Koa, Hono…)
>
> **Storage**: in-memory (no database required)
>
> **Deliverable**: a Git repository with a readable commit history

---

## Data

The files `data/catalog.json` and `data/users.json` are provided.
They constitute your initial data source to be loaded into memory at startup.

---

## Business Rules

### Episode Access

- **`free` episodes**: accessible to everyone, without any condition.
- **`premium` episodes**: require unlocking via coins.
- **Sequential order**: a premium episode can only be unlocked if **all previous
  episodes** in the same series have been:
  - either **watched at ≥ 80%** (present in `watchHistory` with `completedPercent >= 80`)
  - or **already unlocked** (present in `unlockedEpisodes`)

### Pricing

| Mode            | Cost                                                                      |
| --------------- | ------------------------------------------------------------------------- |
| **Single**      | **10 coins** per episode                                                  |
| **Batch**       | All remaining premium episodes in a series: **7 coins/episode**           |

> Batch cost = number of remaining premium episodes × 7.

### "Happy Hour" Promotion

- Between **18:00 and 20:00 UTC** (inclusive–exclusive), the **single** cost drops to **8 coins**.
- The **batch** cost remains at **7 coins/episode**.
- The time is determined by the moment of the request.

---

## Endpoints to Implement

### A. `GET /users/:userId/series/:seriesId/feed`

Returns the ordered list of episodes in a series for a given user.

**Expected response** (example structure):

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

> `batchUnlockCost` = number of unlocked premium episodes × 7.
> `unlockCost` takes Happy Hour into account if applicable.

---

### B. `POST /users/:userId/unlock`

**Single** episode unlock.

**Body**:

```json
{ "episodeId": "ep-104" }
```

**Success response**:

```json
{
  "unlockedEpisode": "ep-104",
  "cost": 10,
  "newBalance": 40
}
```

**Possible errors**:

| Code                    | Description                                                      |
| ----------------------- | ---------------------------------------------------------------- |
| `EPISODE_NOT_FOUND`     | The episode does not exist                                       |
| `ALREADY_UNLOCKED`      | The episode is already unlocked or free                          |
| `INSUFFICIENT_BALANCE`  | Insufficient coin balance                                        |
| `PREREQUISITE_NOT_MET`  | Previous episodes have not been completed/unlocked               |
| `USER_NOT_FOUND`        | The user does not exist                                          |

---

### C. `POST /users/:userId/unlock-batch`

**Batch** unlock of all remaining premium episodes in a series.

**Body**:

```json
{ "seriesId": "series-1" }
```

**Success response**:

```json
{
  "unlockedEpisodes": ["ep-104", "ep-105"],
  "costPerEpisode": 7,
  "totalCost": 14,
  "newBalance": 36
}
```

**Possible errors**:

| Code                      | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `SERIES_NOT_FOUND`        | The series does not exist                                    |
| `NO_EPISODES_TO_UNLOCK`   | All premium episodes are already unlocked                    |
| `INSUFFICIENT_BALANCE`    | Insufficient coin balance                                    |
| `USER_NOT_FOUND`          | The user does not exist                                      |

---

### D. `GET /users/:userId/next`

For each series in which the user has activity (at least one episode
watched or unlocked), returns the **next episode to watch**.

The next episode is the first episode (in order) that has **not been
completed at ≥ 80%**.

**Expected response**:

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

> If a series is fully completed (all episodes ≥ 80%), it does not appear
> in the list (or with a `completed: true` flag, at your discretion).

---

## Unit Tests

Write a minimum of **3 tests** covering the business logic (framework of your choice):

You are free to add more if time allows.

---

## What Matters to Us

- The **clarity and maintainability** of your code matter more than
  the number of completed features.
- Your way of **structuring the project** and **separating
  responsibilities** will be closely examined.
- We prefer **well-thought-out and well-tested** code over code
  that "works" without rigor.
- The follow-up interview will focus on your **design choices**: be ready to explain and defend them.

---

## What Is NOT Expected

- ❌ Authentication / authorization
- ❌ Real database
- ❌ Docker / CI-CD
- ❌ OpenAPI documentation
- ❌ Frontend interface

---

## Notes

- Explain the choices you make, including library and technology choices.
- Prioritize **quality** over quantity: one well-designed endpoint is worth more than
  four rushed ones.
- A 30 to 45-minute interview will follow to discuss your architecture choices
  and possible evolutions.

Good luck! 🚀
