# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. _"The agent handles errors"_ is an opinion.
_"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"_ is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; _"80% seemed reasonable"_ does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**

<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

My search is a plain keyword match, so some phrasings will miss.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**

<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

I picked 5 of 5 because this path depends on a single check: whether `session["search_results"]` is an empty list. If it is, the agent stores a message in `session["error"]` and stops. That check doesn't involve the model or how the query is phrased, so unlike criterion 1, nothing should make it miss.

---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

Given a query that matches at least one listing, the trace shows that the item `suggest_outfit` received has the same `id` as `session["selected_item"]`, and the outfit `create_fit_card` received is identical to `session["outfit_suggestion"]` — in 5 of 5 tries.

**Why this target:**

I picked 5 of 5 because each tool reads its input from the session state, so what is stored and what is passed should never differ. Tracing each tool's inputs lets me confirm the correct value reached it; any mismatch means a bug in how my loop stores or reads the session.

---

## 4. Something about the fit card

<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->

Given the same item on 5 runs, the caption is consistent in content: it mentions the item, its price and its platform — in at least 4 of 5 tries.

**Why this target:**

I picked 4 of 5 because the caption comes from a model, so the exact words change on every run and the model may occasionally drop a detail. What has to stay consistent is the content, not the word-for-word text.

---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

Given a query with at least one matching listing, `suggest_outfit` always returns a real outfit suggestion: with the example wardrobe, it names at least one piece the user actually owns; with an empty wardrobe, it names no wardrobe pieces and gives general styling advice instead — in at least 4 of 5 tries for each wardrobe.

**Why this target:**

I picked 4 of 5 because my code guarantees the response is never empty, but what goes in it comes from the model, which may sometimes ignore the wardrobe or invent a piece the user doesn't own.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
