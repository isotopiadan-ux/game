# בורסת הדירקים — The Dirks Exchange

An internal, tongue-in-cheek prediction market for the office: everyone gets a weekly
allowance of "dirks", stakes it on operational-risk markets (will the shipment clear
customs, will the meeting start on time, how many times someone says Tb-161), and the
winnings feed a season table.

Redesigned as a multi-user app with a shared dashboard.

## Running it

It is a static page. No build step, no dependencies.

```bash
git clone https://github.com/isotopiadan-ux/game.git
cd game
python3 -m http.server 8000   # then open http://localhost:8000
```

GitHub Pages works too: Settings → Pages → deploy from `main` / root.
`.nojekyll` is included so nothing gets filtered.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole application — markup, logic, and copy (Hebrew + English) |
| `support.js` | Small client runtime that renders the component |
| `.nojekyll` | Serve files as-is on GitHub Pages |

## Features

- **Accounts** — name + 4-digit personal code. First sign-in creates the account; the
  same name and code sign back in. The session is remembered per browser.
- **Bet tab** — weekly budget strip (left to stake / bets placed / max payout), market
  cards with inline staking, quick-add chips, live payout preview, and live market
  sentiment bars showing where everyone's money sits.
- **Dashboard tab** — aggregated across all players: total on the table, who has bet and
  who has not, biggest win and biggest loss of the round, season leaderboard, per-market
  sentiment.
- **Season tab** — cumulative net, rounds played, hit rate, and a per-round archive grid.
- **Admin tab** (code `lu177` — change `ADMIN_CODE` in `index.html`) — record results per
  market, lock/unlock trading, add custom markets for the round, publish a notice banner
  to all players, manage the roster (reset code, remove player).
- **Bilingual** — Hebrew (RTL) with an English toggle in the header.

## Game rules

- Every Sunday each player receives the weekly allowance (default 1,000 dirks).
- Trading locks Monday 12:00 Asia/Jerusalem. Unstaked dirks evaporate.
- One bet per market. Multi-select markets split the stake evenly across picks.
- Results land Thursday; payouts are stake × odds and feed the season table.
- Configurable at the top of the logic class: `weeklyGrant`, `defaultLang`,
  `showSentimentToAll`.

## Data layer

The app asks its host for a shared document store and **falls back to `localStorage`**
when none is available — which is what happens on GitHub Pages, so a plain Pages deploy
is single-browser only (a yellow "local copy" badge appears bottom-left).

To make it genuinely multi-user, replace the store in `boot()` with any backend that
satisfies this small interface:

```js
db.doc(path)              // "players/p_abc", "bets/<weekId>__<pid>__<marketId>", "weeks/<weekId>"
  .get() -> Promise<{exists, data()}>
  .set(obj) / .update(obj) / .delete() -> Promise
db.collection(name)       // "players" | "bets" | "weeks"
  .onSnapshot(cb)         // cb({docs:[{id, data()}]}) on every change
```

Firestore matches this shape almost exactly, so swapping it in is a few lines.

## Security note

The 4-digit code is friction, not security — codes are stored in plain text in the shared
store. It keeps colleagues from betting as each other; it is not an auth system. Do not
put anything sensitive in here.

## License

Internal use.
