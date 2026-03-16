# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
  When I first ran the game, it appeared functional on the surface but had several hidden logic bugs. The UI loaded correctly and I could type guesses, but the game behaved unexpectedly — winning was sometimes impossible, and the score moved in ways that didn't make sense.

- List at least two concrete bugs you noticed at the start
  (for example: "the secret number kept changing" or "the hints were backwards").
  - On every even-numbered attempt, the game could never register a win even if the guess was exactly correct — it returned "Too High" instead.
  - The "Go HIGHER!" hint used a downward chart emoji (📉) which directly contradicted the text.
  - Hard mode had a range of 1–50, which was actually easier than Normal mode's 1–100.
  - Guessing too high on even attempts incorrectly awarded +5 points instead of deducting them.
  - The initial attempt counter started at 1 on the very first game load, meaning the player effectively lost one attempt before doing anything.

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)? Claude Code

- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
  Claude identified that on even-numbered attempts, the code converts the secret number to a string (secret = str(st.session_state.secret)) before comparing it to the integer guess. This causes a TypeErro in check_guess, which falls through to the except block — and in that block, equal values incorrectly returned "Too High" instead of "Win" I verified this by manually tracing the code: attempt 2 increments to an even number, the string conversion happens, the TypeError fires, and the broken fallback runs. Once I saw the path clearly, the fix was obvious: change the fallback's equal-value return to "Win".

- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).
  Claude suggested changing Hard mode's range to 1–500, which is a reasonable fix for the logic bug (Hard should be harder than Normal). However, the original project may have intended Hard to mean fewer attempts on a smaller range rather than a wider range. I would need to check the original project spec to confirm — so I accepted the fix but noted that the intended design wasn't fully clear from the code alone.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
  I ran the app locally using streamlit run app.py and manually tested each scenario: I tried guessing correctly on attempt 2 (an even attempt) to verify the win now registers, toggled difficulty to confirm Hard mode now uses a wider range, and watched the score change after wrong guesses to confirm it always subtracts points. I also re-read each changed line alongside the original to make sure the fix matched the intended behavior.

- Describe at least one test you ran (manual or using pytest)
  and what it showed you about your code.
  I deliberately guessed the correct number on attempt 2 to test the even-attempt win bug. Before the fix, the game returned "Go LOWER!" even when my guess matched the secret exactly. After the fix (changing the fallback to return `"Win"`), the balloons animation triggered and the game ended correctly. This confirmed that the type-coercion bug was the root cause, not something deeper.

- Did AI help you design or understand any tests? How?
  Yes — Claude described exactly which line of code to trace and what inputs would trigger each bug, which made it easy to know what to test. Rather than testing randomly, I had a specific scenario (even attempt + correct guess) that would expose the bug immediately. This made manual testing much more targeted and efficient.

---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
  Streamlit reruns the entire Python script from top to bottom every time the user interacts with a widget — clicking a button, typing in a field, anything. Without session state, random.randint() would be called fresh on every rerun, generating a new secret number each time. This means the secret could silently change between when the user typed a guess and when they clicked submit.

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
  Imagine if every time you clicked a button on a webpage, the entire page reloaded from scratch and forgot everything. That's what Streamlit does by default — it re-executes your whole Python file on every interaction. Session state is like a small notebook the app keeps between reruns: you write values into it (st.session_state.secret = 42) and they survive the reload. Without it, nothing persists.

- What change did you make that finally gave the game a stable secret number?
  The fix was already partially in place via the if "secret" not in st.session_state guard, which only generates a new secret on the very first run. The key insight is pairing that with the difficulty-change block (lines 114–119), which intentionally resets the secret only when the player switches difficulty — not on every rerun. Together these two patterns give the game a stable, predictable secret.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  Reading code path-by-path with specific inputs in mind — rather than just skimming for obvious errors — helped me find the even-attempt win bug, which looked fine at first glance. Tracing "what happens if I do X on attempt 2?" revealed a bug that a casual read would miss. I want to keep using that input-driven tracing habit on future projects, especially when something "almost works."

- What is one thing you would do differently next time you work with AI on a coding task?
  I would ask AI to explain _why_ a bug exists before asking it to fix it. In this project, Claude explained the root cause clearly, which made it easy to verify the fix was correct. If I had just asked for a fix without the explanation, I might have accepted a patch without understanding it — and I'd be helpless if a similar bug appeared in a different form.

- In one or two sentences, describe how this project changed the way you think about AI generated code.
  This project showed me that AI-generated code can look polished and run without crashing while still containing meaningful logic bugs that only surface under specific conditions. I'll treat AI-generated code the way I'd treat any unfamiliar code: read it carefully, trace the logic, and test the edge cases rather than assuming it works because it compiles.
