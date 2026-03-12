# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it? The game ran but behaved strangely. The hints were backwards — if you guessed too high, it told you to go even higher. And if you won or lost, clicking New Game did nothing; the game stayed locked.
  
- List at least two concrete bugs you noticed at the start  
  (for example: "the secret number kept changing" or "the hints were backwards").
  The "Go HIGHER / Go LOWER" hints were completely inverted, making the game unwinnable through logic.
  After winning or losing, the New Game button was broken because st.session_state.status was never reset, so st.stop() triggered immediately on every rerun.

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?
-> Claude (Claude Code) was used to read the file, identify bugs, explain them, and apply fixes.
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
-> Claude suggested that the hints were backwards at lines 37–40. It pointed out that if guess > secret, it returned "Go HIGHER!" when it should say "Go LOWER!". This was easy to verify manually. If my guess is 80 and the secret is 42, I should be told to go lower, not higher. Swapping the messages fixed it immediately.
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).
->  Claude suggested fixing the even-attempt string comparison bug (Bug 7) by removing the str() conversion entirely. You decided to leave it as-is, which was the right call — it may have been an intentional quirk of the "glitchy" game theme. The AI treated it as a pure correctness bug without considering the design intent.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
-> By tracing through the logic manually. For example, with the status reset bug: after Claude explained that st.stop() would trigger immediately after New Game because status was still "won", it was clear the fix (adding st.session_state.status = "playing") was correct. The condition on line 142 would now pass, and the game would continue normally.
- Describe at least one test you ran (manual or using pytest) and what it showed you about your code.
-> The difficulty-range bug was caught by switching from Normal to Easy in the sidebar and checking the Developer Debug Info expander. It showed a secret number like 73, clearly outside Easy's 1–20 range. After the fix (tracking st.session_state.difficulty and regenerating the secret when it changes), switching to Easy immediately showed a secret within 1–20.

- Did AI help you design or understand any tests? How?
-> Claude walked through the Streamlit rerun lifecycle to explain why the secret didn't reset when the difficulty changed. That explanation made it clear what to check: open the debug expander, switch difficulty, and see if the secret updates.
---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
-> Every time you interact with a Streamlit app — clicking a button, typing in a box — Streamlit reruns the entire Python script from scratch. Any variable assigned with plain = gets reset. In the original code, secret = random.randint(...) ran on every rerun, so a new random number was picked every single time something happened on the page.
- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
-> Imagine every button click refreshes the whole page, like pressing F5 in a browser. All your variables disappear and start over. st.session_state is like a sticky notepad that survives the refresh; anything you write there stays put between reruns. The pattern is "key" not in st.session_state: st.session_state.key = value means "only set this the very first time; leave it alone after that."
- What change did you make that finally gave the game a stable secret number?
-> Storing the secret in st.session_state with the guard:

if "secret" not in st.session_state or st.session_state.get("difficulty") != difficulty:
    st.session_state.secret = random.randint(low, high)
    st.session_state.difficulty = difficulty
This generates a new secret only once (or when difficulty changes), and then leaves it untouched across all reruns.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
  -> Reading code carefully before trusting it. In this project, we read app.py fully before touching anything, which is how we caught 8 bugs in one pass. In future labs, the habit of "read first, understand the flow, then edit" prevents making fixes that break something else downstream.


- What is one thing you would do differently next time you work with AI on a coding task?
-> Ask the AI to explain why before saying yes to a fix. A few times you rejected edits and asked "explain" or "what's the difference?" 
- In one or two sentences, describe how this project changed the way you think about AI generated code.
-> AI-generated code can look completely reasonable and still be subtly wrong in multiple places at once : the hints were backwards, the score math was off, and the New Game button was silently broken, all at the same time. The code wasn't garbage, it just required a human to actually play the game and trace the logic to catch what the AI got wrong.
