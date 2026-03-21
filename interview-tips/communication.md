# Communication Tips for Interviews

## Before You Start Coding

1. **Listen carefully.** Let the interviewer finish before asking questions.
2. **Repeat the problem.** "So, I need to find X given Y, and return Z. Is that right?"
3. **Ask clarifying questions.** Don't assume — ask about edge cases, input size, constraints.
4. **Announce your approach.** "I'm going to start with a brute-force, then optimize."

---

## While Coding

- **Narrate your thinking:** "I'm using a hash map here to get O(1) lookups."
- **Signal when changing direction:** "Actually, let me reconsider — a sliding window would be better here."
- **Don't go silent for more than 30 seconds.** Even saying "I'm thinking through the edge cases" is better than silence.
- **Ask if you're unsure about a detail:** "Can I assume the input is non-empty?"

---

## When You're Stuck

Don't panic. Instead:

1. **Verbalize what you know:** "I know I need to compare each element with something..."
2. **Try a simple example:** Work through a 3-element input by hand.
3. **Think out loud about approaches:** "I could use a nested loop, but that's O(n²). Is there something faster?"
4. **Ask for a hint gracefully:** "I'm exploring whether a two-pointer approach works here. Am I on the right track?"

---

## After You've Written the Code

1. **Don't immediately say "I'm done."** Review your code.
2. **Walk through with an example:** Trace through the input step by step.
3. **Check edge cases:** Empty input, single element, duplicates, negatives.
4. **State complexity:** "This runs in O(n log n) time and O(n) space."
5. **Suggest improvements:** "If memory is a concern, I could optimize this to O(1) space by..."

---

## Talking About Trade-offs

Interviewers love to hear you reason about trade-offs:

- "Using a hash map gives O(1) lookup but O(n) extra space. I think that's acceptable here."
- "Sorting first simplifies the logic, but adds O(n log n) time. The brute-force is O(n²), so sorting is better."
- "This approach is simpler to implement but doesn't handle duplicates. Do I need to handle duplicates?"

---

## Remote Interview Tips

- **Clean background, good lighting.**
- **Test your audio/video before the interview.**
- **Close unnecessary tabs and notifications.**
- **Have water nearby.**
- **Use the collaborative editor's language features** (syntax highlighting, autocomplete).
- **Type out helper functions** even if you don't implement them fully — it shows intent.

---

## Phrases That Help

| Situation             | What to Say                                                  |
|----------------------|--------------------------------------------------------------|
| Starting a problem   | "Let me make sure I understand the problem..."               |
| Before coding        | "I'm going to approach this by..."                           |
| Hitting a wall       | "I'm considering a few approaches, let me think aloud..."    |
| Making a change      | "I realize there's an issue with my current approach, let me adjust..." |
| After coding         | "Let me trace through this with the example to verify..."    |
| On complexity        | "The time complexity here is O(n) because..."                |
| At the end           | "Is there anything you'd like me to optimize or clarify?"    |
