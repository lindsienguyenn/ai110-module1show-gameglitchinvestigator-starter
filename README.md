# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

- Glitchy Guesser is a number-guessing game built with Streamlit. The player picks a difficulty level (Easy, Normal, or Hard), which sets the secret number's range and the number of allowed attempts. Each guess earns or loses points depending on how quickly and accurately you guess. The goal is to find the secret number before running out of attempts.
- Bugs Found

1. Hints were backwards — "Go HIGHER!" showed when the guess was too high, and "Go LOWER!" showed when it was too low, making the hints actively misleading.
2. New Game didn't reset status — After winning or losing, clicking New Game left status as "won" or "lost", so the game immediately hit st.stop() on every rerun and never actually restarted.
3. New Game ignored difficulty — The new secret was always random.randint(1, 100) regardless of which difficulty was selected.
4. Hard difficulty was easier than Normal — Hard used range 1–50 (smaller = easier to guess) while Normal used 1–100, despite Hard having fewer attempts.
5. Score formula over-penalized early wins — 100 - 10 * (attempt_number + 1) subtracted an extra 10 points, so winning on the first guess gave 80 points instead of 90.
6. attempts had inconsistent starting values — Initialized to 1 on fresh load but reset to 0 on New Game, meaning the first guess was treated as attempt #2 on initial load.
7. Switching difficulty didn't change the secret — The secret was only generated once at startup. Switching from Normal to Easy left a secret like 73 in a 1–20 range.
8. Hint info bar was hardcoded — It always displayed "Guess a number between 1 and 100" regardless of the actual difficulty range.

- Fixes Applied

1. Swapped the hint messages in check_guess — "Go LOWER!" now appears when guess > secret, and "Go HIGHER!" when guess < secret.
2.Added st.session_state.status = "playing" to the New Game block so the game properly resets after a win or loss.
3. Changed random.randint(1, 100) in the New Game block to random.randint(low, high) so the new secret respects the selected difficulty.
4. Changed Hard's range from 1, 50 to 1, 1000 so it is meaningfully harder than Normal.
5. Removed + 1 from the score formula — now points = 100 - 10 * attempt_number.
6. Changed attempts initialization from 1 to 0 so fresh load and New Game behave the same way.
7. Added difficulty tracking in st.session_state — the secret, attempts, status, and history all reset when the difficulty changes:

if "secret" not in st.session_state or st.session_state.get("difficulty") != difficulty:
    st.session_state.secret = random.randint(low, high)
    st.session_state.attempts = 0
    st.session_state.status = "playing"
    st.session_state.history = []
    st.session_state.difficulty = difficulty
8. Updated the hint info bar to use {low} and {high} instead of hardcoded values.

## 📸 Demo

<img width="1333" height="708" alt="image" src="https://github.com/user-attachments/assets/ef98b7c9-7376-482f-be52-d0232d6b5ad3" />


## 🚀 Stretch Features

- [ ] [If you choose to complete Challenge 4, insert a screenshot of your Enhanced Game UI here]
