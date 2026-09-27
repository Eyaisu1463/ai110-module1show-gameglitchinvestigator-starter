# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable.

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the fixed app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: _"How do I keep a variable from resetting in Streamlit when I click a button?"_
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.**
   - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Game purpose:** A Streamlit-based number-guessing game. The player picks a difficulty, gets a limited number of attempts, and tries to guess a hidden number. Correct guesses add points; wrong guesses subtract points. The score tracks how well the player did.

**Bugs found:**

1. **Inverted hints.** Guessing _higher_ than the secret returned "Go HIGHER!" and guessing _lower_ returned "Go LOWER!" — exactly backwards.
2. **Score went up on wrong guesses.** The `update_score` function added +5 points for a "Too High" guess on even attempts (instead of always subtracting 5).
3. **Secret converted to a string on even attempts.** Inside the submit handler, `secret` was being stringified when `attempts % 2 == 0`, which silently broke the comparison logic (it was masked by a `try/except TypeError` fallback inside `check_guess`).

**Fixes applied:**

- Swapped the "Go HIGHER" / "Go LOWER" messages so they match their outcomes.
- Simplified `update_score` so any wrong guess always subtracts 5 points.
- Removed the string conversion so the secret is always an `int`.
- Moved all four helper functions (`get_range_for_difficulty`, `parse_guess`, `check_guess`, `update_score`) out of `app.py` and into `logic_utils.py` for separation of concerns.

## 📸 Demo Walkthrough

1. Player opens the game in a browser at `http://localhost:8501`. The sidebar shows a difficulty dropdown (Easy / Normal / Hard) and the attempt limit for that difficulty.
2. Player selects **Normal** (range 1–100, 8 attempts allowed) and enters a guess of **40** in the text box.
3. Player clicks **Submit Guess 🚀**. The game responds with a yellow warning banner: **"📈 Go HIGHER!"** — correctly telling the player their guess was too low.
4. Player enters **70** and submits. The game responds: **"📉 Go LOWER!"** — correctly telling the player their guess was too high.
5. The score updates after each guess: wrong guesses subtract 5 points. The score no longer increases on wrong guesses.
6. Player guesses the actual secret number. The game shows **"🎉 Correct!"**, fires balloons, sets the status to `"won"`, and displays the final score.
7. Player clicks **New Game 🔁** to reset — the secret, attempts, score, status, and history all reset cleanly.

**Screenshot** _(optional)_: _Not included — the walkthrough above is the required text-based demo._

## 🧪 Test Results

```
collected 3 items

tests\test_game_logic.py ...    [100%]

===== 3 passed in 0.06s =====
```

Tests cover:

- `test_winning_guess` — guess equal to secret returns `("Win", "🎉 Correct!")`
- `test_guess_too_high` — guess 60 vs. secret 50 returns `"Too High"` with a hint containing `"LOWER"`
- `test_guess_too_low` — guess 40 vs. secret 50 returns `"Too Low"` with a hint containing `"HIGHER"`

## 🚀 Stretch Features

- [ ] _No stretch challenges completed. The core project (all three required phases) was completed._
