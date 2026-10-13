# Discord Wordle Solver

Discord bot that prints a plausible-looking sequence of NYT Wordle guesses ending on today's
answer, on the attempt you pick. `/wordle solve 3` guarantees the answer on guess 3.

It knows the answer from the start and steers every guess with it. Guesses before the last one are
always words that are still possible answers given the feedback so far, so the colours the bot
prints are the colours NYT will show. Design and reasoning: [SPEC.md](SPEC.md).

## Setup

Python 3.10 or newer.

```
python -m venv .venv
.venv/Scripts/activate        # source .venv/bin/activate on Linux/macOS
pip install -r requirements.txt
```

## Discord

1. <https://discord.com/developers/applications> → New Application.
2. Bot → Reset Token → copy it. No privileged intents, no Message Content intent — the bot uses
   slash commands only and needs no intents at all.
3. Invite it: open
   `https://discord.com/oauth2/authorize?client_id=<APPLICATION_ID>&scope=bot%20applications.commands`
   and pick the server.

## Run

```
set DISCORD_TOKEN=<token>     # export DISCORD_TOKEN=<token> on Linux/macOS
python -m wordle.bot
```

Commands register globally on startup. If they do not appear in Discord within a minute or two,
re-invite the bot — the `applications.commands` scope is what is usually missing.

## Commands

| Command | |
|---|---|
| `/wordle solve 3` or `4` | The sequence, with the answer landing on that attempt. |
| `/wordle answer` | Today's answer, spoiler-wrapped, visible only to you. |
| `/wordle setanswer <word>` | Override today's answer. Needs Administrator in the channel. |

Example:

```
Wordle · 2026-02-12 · solve in 4
consonant N · vowel E

1. RINSE  🟩⬛🟩⬛🟩
2. ABACK  🟨⬛⬛⬛⬛
3. ABASE  🟨⬛⬛⬛🟩
4. ||RANGE||  🟩🟩🟩🟩🟩
```

## Tests

```
python -m unittest discover -s tests                     # ~5 s
WORDLE_REPLAY_ALL=1 python -m unittest tests.test_solver  # ~2 min, all 2,315 answers
```

The replay proves the guarantee: for every answer, the sequence has the right length, ends on the
answer, never reveals it early, and never contains a guess the feedback had already eliminated —
except in the one documented case where only the answer was left standing.

## Limits

- The answer comes from an undocumented NYT endpoint. It can change or disappear;
  `/wordle setanswer` is the escape hatch, and `cache.json` keeps whatever was fetched.
- Fetching the answer is against NYT's terms. One request a day, personal use.
- `data/answers.txt` is a snapshot of the original 2,315-word answer list. About 12% of recent NYT
  answers are not in it (5 of 42 sampled dates since 2024). The fetched answer is always added to
  the pool, so the guarantee still holds, but those words never appear as guesses before the last
  one.
- Guess quality is capped on purpose. Forcing every guess to stay a live candidate is what makes the
  sequence look human, and it costs information: some middle guesses are weak.
- `/wordle setanswer` is unavailable in DMs — the Administrator check has nothing to check there.
