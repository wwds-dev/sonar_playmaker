# Playmaker — TODO

> **Legend** — priority `P0` critical · `P1` high · `P2` normal · `P3` low
> categories `security` `bug` `feature` `performance` `design` `docs` `testing` `infra` `research`
> owner `@me` (needs you — accounts, keys, money, judgement) · `@ai` (Claude can do this)

---

## Needs you

- [ ] `P2` `research` `@me` **A cycling results source, if you want cycling
      rated.** ESPN serves no cycling endpoint at all, and the free
      alternatives are unofficial scrapers of ProCyclingStats or FirstCycling
      rather than APIs. Until one exists, cycling is priced and screened but
      not rated — which the tab says plainly. Worth deciding whether it is
      worth a scraper, given that a field of 150 riders would also need a
      different model from anything here (§6 territory: a Plackett-Luce
      ordering rather than a head-to-head rating).


- [ ] `P1` `infra` `@me` **Multi-book odds feed.** The Kaunitz consensus-outlier
      strategy (`MODELS.md` §8) is the highest-value item in the whole plan and
      the only one that needs an account. The Odds API's free tier is quoted as
      25 requests/day by one source and 500/month by another — worth checking
      which is real before paying. Everything else in the plan runs on keyless
      data (§10).

---

## v2 — rebuilding the prediction

Planned from `MODELS.md`, which surveys the models with real track records and
sets the build order (§11). Stages 1-5 are done. NFL/NBA/EPL Elo all beat their base rate out of
sample (KEEP); Dixon-Coles scores RPS 0.2149 on held-out EPL matches.

### Stage 1 — fix what is provably wrong (no data, no model) — **done**

Shipped 2026-09-15. `devig.py`, `staking.py`, 82 new tests, and the Playmaker
tab rebuilt around the cross-book screen.

- [x] `P1` `bug` `@ai` **Three devig methods, Shin as default.** Multiplicative
      (ranked last in every comparison), Clarke's power method, and Shin's
      iterative solve, all generalised to N-way markets. The UI shows the three
      side by side, because the disagreement is the argument.
- [x] `P1` `bug` `@ai` **Both sides of a market are now required.** Found while
      testing: a margin is how far prices sum past certainty, so a single price
      cannot reveal one. The form asked for exactly one for years — nothing
      downstream of it could have been right. `MODELS.md` §7a.
- [x] `P1` `bug` `@ai` **"Edge" and "EV" were the same number.** The stat row
      showed both; `edge_versus_market()` is exactly EV rescaled by the decimal
      odds. `staking.edge_versus_fair()` is the independent quantity. §7b.
- [x] `P1` `feature` `@ai` **Cross-book consensus and the outlier screen.** The
      Kaunitz arithmetic, with leave-one-out so a price cannot vote for itself,
      plus a parser for pasted book prices.
- [x] `P1` `design` `@ai` **A narrative estimate can no longer size a bet.**
      `staking.Estimate` carries a source with every probability and
      `staking.kelly()` returns zero for a narrative one.
- [x] `P2` `feature` `@ai` **Probabilities carry an interval and Kelly uses it.**
      Sized at the interval's low end rather than a fixed quarter, so an honest
      interval spanning break-even produces no bet.

### Found while building Stage 1

- [x] `P1` `testing` `@ai` **The test suite could hang forever, and did.**
      `MainWindow.shutdown()` falls through to `QThread.terminate()`, which never
      returns what the thread held — the GIL, or a pthread mutex. Both deadlock
      stacks were sampled out of hung runs. Lives in SONAR's `tests/conftest.py`
      rather than here: the network and the real data directory are closed off,
      `PollThread.run` is a no-op for the session, `terminate()` raises instead
      of wedging, and a faulthandler watchdog aborts with every thread's stack
      after 120s. 15 clean runs; the suite went from 31s to 3-9s.

### Stage 2 — data — **done**

- [x] `P1` `feature` `@ai` **ESPN results adapter.** One adapter plus a league
      code covers NFL, NBA, MLB, NHL, EPL, UCL, NCAAB, UFC and ATP with full
      history and no key — verified live 2026-09-15. Local cache. `MODELS.md` §10.

### Stage 3 — game-level models — **done**

- [x] `P1` `feature` `@ai` **Elo with margin-of-victory and autocorrelation
      correction**, plus between-season regression. §3. The reviewed literature
      says this captures most of the available pre-match signal.
- [x] `P2` `feature` `@ai` **Dixon-Coles** for soccer and hockey — attack/defence
      Poisson, low-score dependence term, time decay ξ=0.0065/half-week. §4.
- [x] `P3` `feature` `@ai` **Pythagorean expectation** as an independent
      cross-check on Elo (exponents 1.83 MLB / 2.37 NFL / 13.91 NBA / 2.15 NHL). §5.

### Stage 4 — measurement — **done**

- [x] `P1` `testing` `@ai` **Score the models before trusting them.** Brier, log
      loss, RPS and a calibration curve over held-out history, with a
      KEEP/WEAK/DROP verdict in the style of `backtest._verdict_for`. A model
      that does not beat the market baseline gets labelled, not shipped quietly.

### Stage 5 — more sports — **done, and earlier than planned**

- [x] `P1` `feature` `@ai` **Nine sports registered**: NFL, NBA, MLB, NHL,
      Premier League, Champions League, NCAA basketball, UFC, ATP tennis — 81
      prop types. The plan had this at Stage 5 on the assumption it needed the
      models first. It did not: everything Stage 1 shipped is pure odds
      arithmetic, so the screen works on a soccer 1X2 the same day it works on
      an NFL spread. Each entry carries its ESPN league path (all nine verified
      live, behind `-m network`) ready for Stage 2.
- [ ] `P2` `feature` `@ai` **Point each sport at a Stage 3 model** once those
      exist — Dixon-Coles suits soccer and hockey, Elo the rest.

### Field events — the gap the new registry opened

- [ ] `P2` `research` `@ai` **A model for golf and cycling.** Both are a
      finishing order across a large field, which no head-to-head rating fits.
      The literature route is a Plackett-Luce / rank-ordered logit over
      competitor strengths, or for golf specifically a strokes-gained model.
      Until then the tab prices and screens them and says it cannot rate them,
      which is the honest state rather than a gap to paper over.
- [ ] `P3` `research` `@ai` **MMA is WEAK and may stay that way.** +0.003 skill
      over 3,369 fights. Style-matchup features (striker vs grappler, reach,
      stance) are the obvious next thing to test, and the pre-registration
      discipline from `CONFIDENCE.md` §10b applies — predict what should
      improve before measuring, not after.

### Stage 6 — player props

- [ ] `P2` `feature` `@ai` **Distributional prop model.** A prop is a tail
      probability, not a projected mean: model opportunity (negative binomial)
      × efficiency, integrate past the line. NFL first — nflverse is the only
      free play-by-play source verified. `MODELS.md` §6.

### Stage 7 — the real scoreboard

- [ ] `P2` `feature` `@ai` **Closing Line Value tracking.** Record the price at
      pick time, compare against the close, report the distribution. Measurable
      within days on every pick, unlike P&L. `MODELS.md` §2.

### Found by SONAR's code-level readiness audit (2026-11-18, SONAR `docs/audit/code-readiness-2026-11-18.md`)

- [ ] `P1` `bug` `@ai` **A zero-width interval gets full Kelly.** `from_consensus`
      returns low = p = high when the peer SE is 0 or NaN (`staking.py:101-102`)
      and a NaN z counts as an outlier (`devig.py:259`): books A, B at -110/-110
      and C at +105/-125 stake 2.38% "at the low end of the interval (50.0%)".
      Two peers also use z = 1.96 where 1 df needs about 12.7. No stake under
      three peers or zero spread; t-quantile for small peer counts. Audit H3.
- [ ] `P2` `bug` `@ai` **The LLM prompt contradicts "commentary only".**
      `SYSTEM_PROMPT` (`__init__.py:355`) says the stated percentage is "the
      number the app converts into expected value" (false) and asks for a Lean;
      the prompt gets only the first book's price (`ui/app.py:4354`). Reword,
      drop the percentage, pass the fair price and screen. Audit M6.
- [ ] `P2` `bug` `@ai` **Impossible American odds are accepted.** `_ODDS`
      (`devig.py:300`) and `american_to_decimal` (`__init__.py:274`) take
      |odds| < 100: "-50" becomes decimal 3.0, a typo "-11" makes a phantom
      edge. Raise on |odds| < 100. Audit M7.
- [ ] `P2` `bug` `@ai` **Duplicate book names break leave-one-out.** `exclude`
      drops every row with that name (`devig.py:225`); "Same, Same, Other"
      prices each row against one book, z = NaN, flagged. Reject duplicate
      names in `parse_quotes`. Audit M8.
- [ ] `P3` `testing` `@ai` **Tests for the above and for the stake path.** No
      test covers H3, M7 or M8, and none asserts `_playmaker_done` leaves the
      stake cells alone. Also: the "best edge" cell shows `opportunities[0]`
      even when it is not an outlier (`ui/app.py:4279`); `results._fetch_token`
      returns `[]` after retries, so a `measure` can run on partial history
      silently (`results.py:289-290`); `calibrated_estimate` is called only from
      tests, so README's model-to-stake route is inert until a sport uses it.
      Audit L5.

---

## v1 — shipped

Odds arithmetic (american/decimal conversion, implied probability, vig removal,
EV, Kelly, edge vs market) with 28 tests; the NFL sport registry; the LLM prompt
and response parser; the Playmaker tab in SONAR.
