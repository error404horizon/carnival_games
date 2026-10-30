# carnival_games

Halloween Carnival arcade for MAGIC 6.0. A single static `index.html` that runs
three games (Sequence Lights, Card Match, Ghost Runner) and is wired to the
booth queue used by the rest of the carnival site.

## Booth

| Setting | Value |
| --- | --- |
| Slug | `carnival-games` |
| Booth key | `GAME88` |
| Queue / score API | `https://gzfpfnrukmsvlfqmdmgn.supabase.co/functions/v1/booth-queue` |
| Leaderboard | `.../booth-queue?leaderboard=sequence` (also `cards`, `runner`) |

The page is preconfigured with those values. No build step, no environment
variables: open `index.html` (or deploy it as a static site) and it connects.

## What the device does

- Nothing is fetched in the background. The operator presses `START`.
- `START` makes one `GET /booth-queue` call with `x-booth-key: GAME88` and
  shows a confirmation screen with the player at the head of the queue: name,
  ticket key, and how many more are waiting.
- `CONFIRM` loads that player and opens the attraction picker.
- `CANCEL` calls `POST /booth-queue { ticket_key, cancel: true }` so a player
  who walks away does not block the line, then returns to the lobby.
- The operator picks a game and the player plays it.
- When the run ends the device posts
  `POST /booth-queue` with `{ ticket_key, result: { game, score, round, won } }`.
  The edge function pops the queue entry and writes the score to
  `magic.game_scores` under the `carnival-games` booth.
- The high score board reads the live booth leaderboard, falling back to the
  local device copy if the network is down.
- `LOCAL PLAY` still works with no queue, and scores are kept in the browser.

## Pointing at a different environment

Use query parameters (they are saved on the device for next time):

```
index.html?api=http://127.0.0.1:54321/functions/v1/booth-queue&key=GAME88
```

`Machine Settings` shows the active booth key, slug and API URL.
