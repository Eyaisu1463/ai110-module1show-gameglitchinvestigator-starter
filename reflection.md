# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

When I first ran the game, it looked like a normal Streamlit guessing game — difficulty selector, guess input, submit button. The UI worked, but the game's behavior was off. The biggest clue was in the hints: when I guessed a number higher than the secret, the game told me to "Go HIGHER" instead of "Go LOWER". When I guessed lower than the secret, it told me to "Go LOWER" instead of "Go HIGHER". I also noticed the score could go *up* on a wrong guess (odd behavior), and there was some suspicious code in `app.py` that converted the secret to a string on even attempts — a bug I only understood after reading the code.

**Bug Reproduction Log**

Document at least 3 bugs you found. Add rows as needed.

| Input | Expected Behavior | Actual Behavior | Console Output / Error |
|-------|-------------------|-----------------|------------------------|
| Guess a number higher than the secret (e.g., secret=50, guess=90) | Hint says "Go LOWER" | Hint says "Go HIGHER" | No error — silent logic bug |
| Guess a number lower than the secret (e.g., secret=50, guess=10) | Hint says "Go HIGHER" | Hint says "Go LOWER" | No error — silent logic bug |
| Guess wrong on an even attempt (e.g., attempt 2, 4, 6) | Score decreases by 5 | Score *increases* by 5 | No error — silent logic bug |
| Guess wrong on an even attempt after the secret was stringified | Compare guess to secret correctly | Comparison silently coerces types via `except TypeError` fallback | No error — hidden by try/except |
| Enter `hello` | Friendly "That is not a number" message | Friendly message shown (this one worked correctly) | No error |

---

## 2. How did you use AI as a teammate?

I used ChatGPT as my AI coding assistant for this project. I attached `app.py` and `logic_utils.py` to the chat and asked the AI to help me find and fix the bugs.

**One AI suggestion I accepted:** I asked the AI to help me refactor the four logic functions (`get_range_for_difficulty`, `parse_guess`, `check_guess`, `update_score`) out of `app.py` and into `logic_utils.py`, and to fix the inverted hints while doing so. The AI correctly:
- Moved the functions to `logic_utils.py`
- Swapped the "Go HIGHER" / "Go LOWER" messages so they matched their outcomes
- Added `from logic_utils import ...` at the top of `app.py`
- Removed the duplicated function definitions from `app.py`

I verified this by (a) reading the diff line-by-line, (b) running `pytest`, and (c) playing the game and confirming the hint now matches the guess direction.

**One AI suggestion I did NOT accept as written:** The AI initially suggested I keep the `try / except TypeError:` fallback inside `check_guess` (the one that converts `guess` to a string if comparison fails). I rejected that suggestion because it was treating a *symptom* rather than the *root cause*. The real bug was in `app.py`, where `secret` was being converted to a string on even attempts — that's what triggered the TypeError in the first place. Removing the string conversion at the source made the whole fallback unnecessary, and it made `check_guess` shorter and easier to read. I verified this by running `pytest` (3 tests passed) and by checking that the game no longer behaved differently on even-numbered attempts.

---

## 3. Debugging and testing your fixes

I decided a bug was truly fixed when **three conditions** were met: (1) the reasoning was clear — I understood *why* the fix worked, not just that it worked; (2) `pytest` passed, including tests that specifically targeted the fixed behavior; and (3) the live game behaved correctly when I played it.

One test I wrote was `test_guess_too_high`, which verifies that `check_guess(60, 50)` returns `("Too High", "📉 Go LOWER!")`. This test proved two things at once: the outcome was correct, and the hint message was no longer inverted. Before my fix, this test would have failed because the message was "Go HIGHER". After the fix, all three tests passed:

```
collected 3 items

tests\test_game_logic.py ...    [100%]

===== 3 passed in 0.06s =====
```

AI helped me design these tests by suggesting the pattern of **asserting on both the outcome and the message** — I had only planned to check the outcome. Checking the message caught the inverted hint and locks in the fix.

---

## 4. What did you learn about Streamlit and state?

Streamlit works differently from a normal web app: every time you interact with a widget (click a button, type in a text box), the **entire Python script re-runs from the top**. This is called a "rerun." If you use regular variables to store things like the secret number or the score, those variables get wiped on every rerun — the game would forget everything.

`st.session_state` is Streamlit's solution. It's a dictionary that **survives reruns**. In this project, `st.session_state.secret`, `st.session_state.attempts`, `st.session_state.score`, and `st.session_state.history` are all stored there, so the game remembers them between interactions. If I were explaining this to a friend: "Streamlit re-runs your whole script like refreshing a page, and `session_state` is the little notebook that survives the refresh."

---

## 5. Looking ahead: your developer habits

**Habit I want to reuse:** Writing a **small, focused test for each bug I fix**. Before this project, I thought of tests as something you write at the end. Now I see them as a way to *prove* a fix works and to prevent the same bug from sneaking back in later. I also want to reuse the habit of **reading AI-generated diffs line by line** instead of accepting them blindly.

**What I'd do differently next time:** I'd set up `conftest.py` and understand the pytest import path **before** I started fixing code. I hit a `ModuleNotFoundError: No module named 'logic_utils'` error that was completely avoidable — it wasn't a code bug, it was a project setup issue.

**How this project changed my thinking about AI-generated code:** I learned that AI-generated code can look confident and complete while hiding subtle logic bugs (like an inverted comparison, or a stringified value that "just works" by accident through a try/except). The AI's claim that the code was "production-ready" was a **hallucination** — my job as the engineer was to verify with tests and manual observation, not to trust the output.