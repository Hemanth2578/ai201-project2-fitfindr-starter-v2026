# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

You describe a thrifted piece you're looking for in plain words, like "vintage graphic tee under $30", and can add a size or a price limit. FitFindr finds a matching secondhand item and tells you what it is, how much it costs and where to buy it. It then shows you how to wear it with clothes you already own, plus a short caption ready to post about your find. If nothing matches, it tells you what to change, like using broader words, trying another size or raising your budget.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:**
  `search_listings` takes a description and the optional filters `size` and `max_price`, and returns the listings that match all of them.
- **Inputs:**
  `description` (string), `size` (string | None, optional), `max_price` (float | None, optional)
- **Returns:**
  A list of matching listing dicts, each with the fields `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, `platform`
- **When it has nothing:**
  An empty list (`[]`), meaning no listings match the description, size and max_price.

### `suggest_outfit`

- **What it does:**
  `suggest_outfit` takes a new item and the user's wardrobe and suggests one or two outfits.
- **Inputs:**
  `new_item` (dict), `wardrobe` (dict)
- **Returns:**
  A string (the LLM's response) describing one or two outfits that pair the new item with pieces from the user's wardrobe.
- **When it has nothing:**
  If the wardrobe is empty (`wardrobe["items"]` is an empty list), returns general styling ideas for the new item as a non-empty string, instead of raising an error or returning an empty string.

### `create_fit_card`

- **What it does:**
  `create_fit_card` takes the suggested outfit and the new item and writes a short caption someone would actually post about the find.
- **Inputs:**
  `outfit` (string), `new_item` (dict)
- **Returns:**
  A string containing a short caption someone would actually post about the find.
- **When it has nothing:**
  If `outfit` is empty or only whitespace, returns a descriptive message string instead of a caption, rather than raising an error.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:**
If `search_listings` returns an empty list, put a message in `session["error"]` telling the user what they could change (not just "No results"), and return the session without calling any other tool. Otherwise, choose an item and store it in `session["selected_item"]`, call `suggest_outfit` with the selected item and the wardrobe and store the result in `session["outfit_suggestion"]`, then call `create_fit_card` and store the result in `session["fit_card"]`. Finally, return the session.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->
The query is parsed with regex because it gives the same output every time it runs. The downside is that it misses phrasings it has never seen: for example, "nothing over thirty dollars" parses to no price at all.

**What moves through the session:** <!-- which fields, in what order -->
The session fills these fields in order: the parsed query goes into `parsed`, and the matching listings go into `search_results`. If there are no results, a message saying what the user could change is stored in `error`, and `selected_item`, `outfit_suggestion` and `fit_card` stay `None`. Otherwise, the first result is stored in `selected_item`. That item and the `wardrobe` are used to get an outfit suggestion, which is stored in `outfit_suggestion`. The outfit suggestion and `selected_item` are then used to create a short caption, which is stored in `fit_card`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask "vintage graphic tee under $30"

[1] parse_query
in: vintage graphic tee under $30
out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
in: dict with keys: description, size, max_price
out: 8 items: Graphic Tee — 2003 Tour Bootleg Style, Y2K Baby Tee — Butterfly Print, Vintage Graphic Hoodie — Faded Black … +5 more
→ 8 match(es)
[3] select_item
out: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
[4] suggest_outfit
in: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
out: Here is a practical, streetwear-inspired outfit you can build using your new graphic tee and pieces from your …
→ 10 wardrobe item(s)
[5] create_fit_card
in: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
out: Scored this vintage 2003 tour bootleg graphic tee for just $24 on Depop and I'm obsessed! I paired it with bag…

Found: Graphic Tee — 2003 Tour Bootleg Style — $24.0 on depop

Outfit: Here is a practical, streetwear-inspired outfit you can build using your new graphic tee and pieces from your wardrobe:

### Outfit: 90s Grunge Streetwear

This look leans into the vintage, worn-in vibe of the tour tee by pairing it with dark denim and rugged boots for an effortless, everyday outfit.

- **Top:** Graphic Tee — 2003 Tour Bootleg Style (Thrifted)
- **Bottoms:** Baggy straight-leg jeans (`w_001`)
- **Shoes:** Black combat boots (`w_008`)
- **Accessory:** Black crossbody bag (`w_010`)

**Styling Tip:** Since the tee has a slightly boxy fit and the jeans are high-waisted and baggy, you can do a loose front-tuck to define your waist while keeping that relaxed, streetwear silhouette. Throw on the black crossbody bag to complete the look.

Fit card: Scored this vintage 2003 tour bootleg graphic tee for just $24 on Depop and I'm obsessed! I paired it with baggy denim and combat boots for the ultimate 90s grunge streetwear fit. That effortless, worn-in vibe is unmatched. ✨🖤

2 model calls this session, 1060 prompt + 260 output tokens
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two wearable, everyday outfits centered around your new Vintage Levi's 501 Jeans:

### Outfit 1: Effortless Off-Duty (Casual & Streetwear)

_This look plays on classic casual basics, letting the vintage wash of the 501s take center stage while keeping the vibe relaxed and comfortable._

- **Thrifted Item:** Vintage Levi's 501 Jeans — Medium Wash
- **Wardrobe Pieces:**
  - **White ribbed tank top** (`w_003`) tucked into the waistband to create a clean, fitted base.
  - **Oversized grey crewneck sweatshirt** (`w_004`) layered over the tank for an easy, cozy silhouette.
  - **Chunky white sneakers** (`w_007`) to anchor the streetwear aesthetic.
  - **Black crossbody bag** (`w_010`) for hands-free daily errands.

---

### Outfit 2: Edgy Contrast (Retro-Cool)

_Leaning into the 90s vintage roots of the 501s, this outfit pairs classic denim with black layers and hardware for a subtle grunge edge._

- **Thrifted Item:** Vintage Levi's 501 Jeans — Medium Wash
- **Wardrobe Pieces:**
  - **Black cropped zip hoodie** (`w_005`) worn zipped up to create a sharp contrast against the medium-wash denim.
  - **Brown leather belt** (`w_009`) threaded through the loops to add a classic touch and break up the black and blue.
  - **Black combat boots** (`w_008`) to give the straight-leg hem a tougher, grounded finish.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Scored these Vintage Levi's 501 Jeans for just $38 on Depop, and they honestly have the absolute best faded wash. I styled them with a simple white tee and crisp sneakers for that effortless, off-duty model street style. Let me know if you're living for this effortless casual fit as much as I am!
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- _What I asked for:_ I gave Claude my five criteria and asked, "Could someone check this without asking me what I meant?" I also asked it to fix the wording and to check whether each "Why this target" explained the number.
- _What came back:_ It said criterion 4 ("the same caption every time, because temperature is 0 and the cache is on") could never fail, and that `tools.py` treats identical captions as a bug. It also said criterion 3 ("the agent reads the session state") wasn't something a checker could see, and that my reasons described what happens rather than why I picked that number.
- _What I changed:_ I rewrote criterion 4 to check content instead of exact words (the caption mentions the item, its price and its platform, in at least 4 of 5 tries). The rewrite also added a 2-to-4-sentence length rule, which I removed because my own Tool Inventory never defines a length. I changed criterion 3 to compare what the trace shows each tool received against `session["selected_item"]` and `session["outfit_suggestion"]`.

**Moment 2**

- _What I asked for:_ I asked Claude to help me write the code for my tools, especially the prompts inside `suggest_outfit` and `create_fit_card`.
- _What came back:_ Claude pointed out edge cases my code didn't handle and a gap in my prompt. `suggest_outfit` would crash if the wardrobe was missing or had no `items` key, and `create_fit_card` would still call the model with an empty or blank outfit. My wardrobe prompt also didn't stop the model from suggesting pieces the user doesn't own.
- _What I changed:_ In `suggest_outfit`, I added a check that treats a missing wardrobe or a missing `items` key as an empty wardrobe instead of crashing. In `create_fit_card`, I added a guard (`if not outfit.strip()`) that returns a descriptive message instead of calling the model. In the wardrobe prompt, I added "Do not claim the user owns anything that is not listed."

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

Run: `python run_eval.py --label before`, caching off, temperature 0.9. Full output is in `results/run_2026-10-08_1729_before.md`.

| Criterion                                                                                                   | Target                   | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict                      |
| ----------------------------------------------------------------------------------------------------------- | ------------------------ | ----- | ----- | ----- | ----- | ----- | ---------------------------- |
| 1. A matching query completes all three tools and returns a fit card                                        | 4 of 5                   | PASS  | PASS  | PASS  | PASS  | PASS  | MET (5/5)                    |
| 2. An impossible query stops before `suggest_outfit` and names what to change                               | 5 of 5                   | PASS  | PASS  | PASS  | PASS  | PASS  | MET (5/5)                    |
| 3. The trace shows the session's item and outfit reached the next tools                                     | 5 of 5                   | PASS  | PASS  | PASS  | PASS  | PASS  | MET (5/5)                    |
| 4. The fit card for the same item mentions the item, its price and its platform                             | 4 of 5                   | PASS  | PASS  | PASS  | PASS  | PASS  | MET (5/5)                    |
| 5. `suggest_outfit` names owned pieces with the example wardrobe and gives general advice with an empty one | 4 of 5 for each wardrobe | PASS  | PASS  | PASS  | PASS  | PASS  | MET (5/5 example, 5/5 empty) |

How I scored each try:

- **Criterion 1** (`"vintage graphic tee under $30"`): PASS if the session has a `fit_card`. All five returned one for _Y2K Baby Tee — Butterfly Print_.
- **Criterion 2** (`"designer ballgown size XXS under $5"`): PASS if the trace ends at `branch` with no `suggest_outfit` step, and the error message lists things to change. All five did.
- **Criterion 3** (`"Chrome hearts black jacket with white embroidered text under $1000"`): PASS if the `suggest_outfit` input in the trace is the same item as `selected_item`, and the `create_fit_card` input starts with the same text as `outfit_suggestion`. The trace shortens long values to about 110 characters, so I compared the item's title and the start of the outfit text, not the full `id` and the full outfit.
- **Criterion 4** (`"Denim Jeans with grey color under $100"`): PASS if the caption mentions "Levi's 501", "$38" and "Depop". All five did.
- **Criterion 5** (`"cargo pants under $40"`): PASS for a try only if both wardrobes passed. With the example wardrobe, every suggestion named at least one owned piece. Four used IDs like `w_003`, and Try 2 named them by name ("White ribbed tank top", "Black cropped zip hoodie"). With the empty wardrobe, no suggestion named a wardrobe ID or said the user owned anything; all five gave general advice ("using everyday wardrobe basics").

**Real output from one try per criterion**, from `results/run_2026-10-08_1729_before.md`. The loop is `agent.py::run_agent` and the tools are in `tools.py`.

Criterion 1, Try 1: fit card from `tools.py::create_fit_card`

```
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Fit card:
Obsessed is an understatement for this Y2K butterfly baby tee I just scored on Depop for only $18! I'm leaning all the way into that cottagecore-meets-grunge aesthetic by styling it with edgy combat boots and wide-leg trousers for the ultimate contrast. Which vibe are we feeling more today—streetwear casual or vintage grunge? 🦋✨
```

Criterion 2, Try 1: the branch in `agent.py::run_agent`, message from `agent.py::_nothing_found_message`

```
- stopped early: yes — Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
- selected_item: (none)
- search_results: 0

[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
      →    description= 'designer ballgown' / Size = XXS / max_price = 5.0
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping before suggest_outfit
```

Criterion 3, Try 1: trace from `agent.py::run_agent` (`trace.step` calls), compared with the session

```
- selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)

[3] select_item
      out: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
[4] suggest_outfit
      in:  90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
[5] create_fit_card
      in:  ('Here is a wearable, everyday outfit styled around your new 90s Champion track jacket, utilizing pieces strai…

Outfit suggestion (session["outfit_suggestion"]) begins:
Here is a wearable, everyday outfit styled around your new 90s Champion track jacket, utilizing pieces strai...
```

Criterion 4, Try 1: fit card from `tools.py::create_fit_card`

```
- selected_item: Vintage Levi's 501 Jeans — Medium Wash ($38.0, depop)

Fit card:
Score! Just scored these Vintage Levi's 501 Jeans for only $38 on Depop, and they seriously have the best medium-wash fade. I styled them two ways—keep it cozy with an oversized crewneck for effortless streetwear vibes, or lean into an edgy double-denim look with a cropped hoodie and combat boots. Which fit are you rocking?
```

Criterion 5, Try 2: outfit suggestions from `tools.py::suggest_outfit`

```
Example wardrobe (first outfit):
Here are two wearable outfits centered around your new Y2K low-rise khaki cargo pants, using pieces straight from your wardrobe:

### Outfit 1: Y2K Streetwear Edge
* **Thrifted Item:** Low-Rise Cargo Pants — Khaki
* **Wardrobe Pieces:**
  * **White ribbed tank top** (Tucked in to highlight the low-rise waistline)
  * **Black cropped zip hoodie** (Layered open over the tank for an authentic Y2K proportion play)
  * **Chunky white sneakers** (To anchor the streetwear aesthetic)
  * **Black crossbody bag** (For an easy, hands-free everyday accessory)

Empty wardrobe:
Here are two wearable ways to style these low-rise khaki cargo pants using everyday wardrobe basics:

**1. Casual Streetwear (Balanced Proportions)**
Since low-rise cargos have a relaxed, Y2K-inspired silhouette, balance the volume by pairing them with a fitted basic black or white ribbed tank top. Add a simple leather belt and a pair of classic white sneakers to keep the look clean, comfortable, and grounded.

**2. Effortless Layering**
Embrace the 2000s aesthetic mentioned in the description by layering a boxy, oversized graphic t-shirt or a simple heather gray crewneck sweatshirt over a contrasting long-sleeve tee. Let the hem peek out and finish the outfit with chunky sneakers or retro trainers.
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| #   | Criterion | Target | Verdict | How I decided |
| --- | --------- | ------ | ------- | ------------- |
| 1   | A matching query completes all three tools and returns a fit card | 4 of 5 | MET (5/5) | All five tries of "vintage graphic tee under $30" ran `search_listings`, `suggest_outfit` and `create_fit_card` and ended with a fit card in the session. 5 passes is at least 4, so the target held. |
| 2   | An impossible query stops before `suggest_outfit` and names what to change | 5 of 5 | MET (5/5) | In all five tries of "designer ballgown size XXS under $5", the trace ended at `branch` with no `suggest_outfit` step, `fit_card` stayed `None`, and the error listed three things to change. The target needed all five, and all five passed. |
| 3   | The trace shows the item and outfit from the session reached the next tools | 5 of 5 | MET (5/5) | In all five tries, the `suggest_outfit` input in the trace was the same item as `selected_item`, and the `create_fit_card` input began with the same text as `outfit_suggestion`. I compared titles and the start of the outfit, because the trace never shows the item's `id` and cuts long values off at about 110 characters. |
| 4   | The fit card for the same item mentions the item, its price and its platform | 4 of 5 | MET (5/5) | All five captions for the Vintage Levi's 501 Jeans named the jeans, "$38" and "Depop". 5 passes is at least 4. |
| 5   | `suggest_outfit` names owned pieces with the example wardrobe and gives general advice with an empty one | 4 of 5 for each wardrobe | MET (5/5 and 5/5) | With the example wardrobe, all five suggestions for the cargo pants named at least one owned piece (four by ID such as `w_003`, one by name). With the empty wardrobe, all five gave general advice and none named a wardrobe ID or claimed the user owned anything. Both wardrobes passed 5 of 5. |

**Diagnoses**

I missed nothing: all five criteria passed in all five tries. Looking back, my targets were on the easy side. They check that each path through the agent works (a full run, the empty-search branch, the session state, the fit card and the empty wardrobe), but none of them checks whether the search returned the kind of item the user asked for. I saw that go wrong outside the test run. "WWII Navy Peacoat Size XL under $50" returned an Oversized Crewneck Sweatshirt, because the only word it matched was "navy". Criterion 1 still counts a run like that as a pass, since it only checks that a fit card came back. That's the criterion I would tighten: the selected item should also be the type of item the query asked for (a peacoat request should return a coat or jacket, not a sweatshirt).

I'm also revising criterion 3 in `criteria.md`, because it couldn't be measured as written. It says the item `suggest_outfit` received has the same `id` as `session["selected_item"]`, but my trace only shows the item's title and price, never its `id`. I scored it by comparing titles instead, so the revision makes the trace log the `id` and compares that.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
$ python app.py ask "Denim Jeans with grey color" --trace

[1] parse_query
      in:  Denim Jeans with grey color
      out: dict with keys: description, size, max_price
      →    description= 'Denim Jeans with grey color' / Size = None / max_price = None
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Vintage Levi's 501 Jeans — Medium Wash, Straight Leg Black Jeans — Faded, Denim Jacket — Light Wash, Cropped … +7 more
      →    10 match(es)
[3] select_item
      out: Vintage Levi's 501 Jeans — Medium Wash ($38.0, depop)
[4] suggest_outfit
      in:  Vintage Levi's 501 Jeans — Medium Wash ($38.0, depop)
      out: Here are two wearable, everyday outfits centered around your new Vintage Levi's 501 Jeans:  ### Outfit 1: Effo…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  ("Here are two wearable, everyday outfits centered around your new Vintage Levi's 501 Jeans:\n\n### Outfit 1: …
      out: Found my new holy grail Vintage Levi's 501 Jeans for just $38 on Depop, and I am obsessed with this effortless…
      →    10 wardrobe item(s)

  Found:    Vintage Levi's 501 Jeans — Medium Wash — $38.0 on depop

  Outfit:   Here are two wearable, everyday outfits centered around your new Vintage Levi's 501 Jeans:

### Outfit 1: Effortless Off-Duty (Casual & Streetwear)
*This look plays on classic casual basics, letting the vintage wash of the 501s take center stage while keeping the vibe relaxed and comfortable.*

* **Thrifted Item:** Vintage Levi's 501 Jeans — Medium Wash
* **Wardrobe Pieces:**
  * **White ribbed tank top** (`w_003`) tucked into the waistband to create a clean, fitted base.
  * **Oversized grey crewneck sweatshirt** (`w_004`) layered over the tank for an easy, cozy silhouette.
  * **Chunky white sneakers** (`w_007`) to anchor the streetwear aesthetic.
  * **Black crossbody bag** (`w_010`) for hands-free daily errands.

---

### Outfit 2: Edgy Contrast (Retro-Cool)
*Leaning into the 90s vintage roots of the 501s, this outfit pairs classic denim with black layers and hardware for a subtle grunge edge.*

* **Thrifted Item:** Vintage Levi's 501 Jeans — Medium Wash
* **Wardrobe Pieces:**
  * **Black cropped zip hoodie** (`w_005`) worn zipped up to create a sharp contrast against the medium-wash denim.
  * **Brown leather belt** (`w_009`) threaded through the loops to add a classic touch and break up the black and blue.
  * **Black combat boots** (`w_008`) to give the straight-leg hem a tougher, grounded finish.

  Fit card: Found my new holy grail Vintage Levi's 501 Jeans for just $38 on Depop, and I am obsessed with this effortless off-duty streetwear vibe! Paired them with a cozy grey crewneck and chunky sneakers for the ultimate casual, running-errands look. Honestly, nothing beats broken-in vintage denim that fits like a glove. ✨👖

0 model calls this session, 2 served from cache
```

**Empty search**

```
$ python app.py ask "Denim Jeans under $20" --trace

[1] parse_query
      in:  Denim Jeans under $20
      out: dict with keys: description, size, max_price
      →    description= 'Denim Jeans' / Size = None / max_price = 20.0
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping before suggest_outfit

  Nothing in the listings matched description 'Denim Jeans', under $20.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; raise the price ceiling above $20.

0 model calls this session
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->

I moved `search_listings` onto MCP. In `mcp_server.py`, I registered it with `@mcp.tool()`, kept the same inputs (`description: str`, `size: str | None`, `max_price: float | None`), and wrote a description for a reader who can't see the code. The function body just calls the existing implementation in `tools.py`. In `agent.py`, `run_agent` now calls the tool through `_search()`, which uses `mcp_client.call_tool("search_listings", ...)`. If the MCP server can't be reached, it prints "MCP server not available, falling back to direct call." and calls `search_listings` directly, so the agent keeps working. The trace labels this step `search_listings (via MCP)`.

The rewire worked. `python mcp_client.py` listed `search_listings` with its description and its three inputs, and none of my runs printed the fallback message, so every search went through the server. The MCP move itself didn't change any results, because the server calls the same search code. In the same commit, though, I also changed the search scoring to match keywords against the title, category, colors, style tags, brand and condition, not just the description. After that, "vintage graphic tee under $30" returned 10 matches instead of 8, and the first result changed from "Graphic Tee — 2003 Tour Bootleg Style" to "Y2K Baby Tee — Butterfly Print".

**Failure tests**

I triggered each failure on purpose and recorded what the agent said.

| Failure                                                          | How I triggered it                                                                                                    | What the agent said                                                                                                                                                                                                                                                                                                                          | Result                                                                                                                                                                                                                                |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Empty search                                                     | `python app.py ask "Denim Jeans under $20"`                                                                           | "Nothing in the listings matched description 'Denim Jeans', under $20. Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; raise the price ceiling above $20."                                                                                                                                         | Stopped before `suggest_outfit` and named what to change. No model calls.                                                                                                                                                             |
| Empty wardrobe                                                   | `python app.py ask "hoodie under $50" --empty-wardrobe`                                                               | "Here are two easy, wearable ways to style this vintage faded black hoodie using common wardrobe basics: …"                                                                                                                                                                                                                                  | Returned general styling advice. The trace shows `0 wardrobe item(s)`. No crash and no empty string.                                                                                                                                  |
| Model unavailable (bad key)                                      | Changed one character of `GEMINI_API_KEY` in `.env`, then ran `python app.py ask "hoodie under $50" --empty-wardrobe` | "The model couldn't be reached, so the outfit and caption steps didn't run. The search worked — 1 listing(s) were found. Check GEMINI_API_KEY in your .env, then run the same query again. What the service said: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com."    | Stopped with a message instead of a stack trace. The search results were kept.                                                                                                                                                        |
| Model unavailable (service overloaded, not triggered on purpose) | `python app.py ask "vintage graphic tee under $30"` during a Gemini outage                                            | "The model couldn't be reached, so the outfit and caption steps didn't run. The search worked — 10 listing(s) were found. Check GEMINI_API_KEY in your .env, then run the same query again. What the service said: Couldn't reach the model: 503 UNAVAILABLE … 'This model is currently experiencing high demand … Please try again later.'" | No crash, but the message is misleading. The key was fine, so "Check GEMINI_API_KEY" points the user at the wrong fix. It also says the outfit step didn't run, but `suggest_outfit` had succeeded and only `create_fit_card` failed. |

```
$ python app.py ask "hoodie under $50" --empty-wardrobe     # with a broken key

[1] parse_query
      in:  hoodie under $50
      out: dict with keys: description, size, max_price
      →    description= 'hoodie' / Size = None / max_price = 50.0
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 1 items: Vintage Graphic Hoodie — Faded Black
      →    1 match(es)
[3] select_item
      out: Vintage Graphic Hoodie — Faded Black ($26.0, depop)
[4] model unavailable
      →    stopping, search results kept

  The model couldn't be reached, so the outfit and caption steps didn't run. The search worked — 1 listing(s) were found. Check GEMINI_API_KEY in your .env, then run the same query again.
What the service said: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.

1 model calls this session
```

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

One change, in the search tool (`tools.py::search_listings` and `tools.py::_score_listing`). A color word can no longer make a match on its own. The search now splits the query's keywords into color words (every word used in the listings' `colors` field, such as "navy" or "black") and item words (everything else, such as "peacoat" or "boots"). If the query has any item words, a listing must match at least one of them, or it scores 0 and is dropped. A query made only of colors, like "black", works as before. Scoring and ranking are otherwise unchanged.

Two other edits happened between the before and after runs, and neither changes what the agent does. For the criterion 3 revision, I added the item's `id` to the trace notes for `suggest_outfit` and `create_fit_card` in `agent.py::run_agent`. I also added two diagnostic scenarios to `scenarios.py`.

**Which failure it was meant to fix:**

The wrong-item search from my Milestone 4 diagnosis. The step was the tool (`search_listings`), and the mechanism was keyword scoring that counted a color as a full match. "WWII Navy Peacoat Size XL under $50" returned an Oversized Crewneck Sweatshirt because "navy" was the only word that matched, and "black boots under $60" ranked an Oversized Flannel Shirt first because it matched "black". The agent then styled and captioned an item the user never asked for.

### Run Log — After

Run: `python run_eval.py --label after`, caching off, temperature 0.9. Full output is in `results/run_2026-10-08_1813_after.md`. Same scenarios and queries as the before run, plus two diagnostics.

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
| --------- | ------ | ----- | ----- | ----- | ----- | ----- | ------- |
| 1. A matching query completes all three tools and returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before `suggest_outfit` and names what to change | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. (revised) The trace logs the `id` of the item passed to `suggest_outfit` and `create_fit_card`, both equal `selected_item`'s `id`, and the outfit matches the session | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. The fit card for the same item mentions the item, its price and its platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. `suggest_outfit` names owned pieces with the example wardrobe and gives general advice with an empty one | 4 of 5 for each wardrobe | PASS | PASS | PASS | PASS | PASS | MET (5/5 example, 5/5 empty) |

How I scored the after run:

- **Criterion 1:** every try selected *Y2K Baby Tee — Butterfly Print* and returned a fit card.
- **Criterion 2:** every trace ended at `branch` with no `suggest_outfit` step, and the error listed things to change.
- **Criterion 3:** in every try, both trace steps showed `item id=lst_004`, which is the `id` of `selected_item` (*90s Track Jacket — Navy/White Stripe*). The `create_fit_card` input began with the same text as `outfit_suggestion`. This is the first run where the `id` could be checked directly. The outfit is still compared by its start, because the trace cuts long values off at about 110 characters.
- **Criterion 4:** every caption mentioned Levi's 501, "$38" and "Depop".
- **Criterion 5:** with the example wardrobe, every suggestion named owned pieces by ID (such as `w_003`, `w_007`). With the empty wardrobe, none named a wardrobe ID or said the user owned anything.

**Diagnostics: the failure the change was meant to fix**

These two queries aren't part of my five criteria. They're the wrong-item searches from my diagnosis. The before run didn't include them, so their "before" comes from running the old search on the same queries. Search doesn't call the model, so it returns the same result every time.

| Query | Before the change | After the change (5 of 5 tries) |
| ----- | ----------------- | ------------------------------- |
| `WWII Navy Peacoat Size XL under $50` | 1 result: *Oversized Crewneck Sweatshirt — Vintage Navy*, matched only on "navy". The agent styled a sweatshirt for someone who asked for a peacoat. | 0 results. The agent stopped at the branch: "Nothing in the listings matched description 'WWII Navy Peacoat', size XL, under $50. Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $50." |
| `black boots under $60` | 9 results, top: *Oversized Flannel Shirt — Plaid Red/Black*, matched only on "black". | 1 result: *Suede Chelsea Boots — Tan* ($44, poshmark), the only boots in the listings. All three tools ran in every try. The boots are tan, not black, because no black boots are listed. |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->

Yes, for the failure it targeted, and it didn't break anything else.

- **The diagnostics show the fix.** The peacoat query now stops at the branch with a message saying what to change, instead of styling a sweatshirt. The black boots query now returns the only boots in the listings instead of a flannel shirt. Both held in 5 of 5 tries.
- **My five criteria didn't change.** All five were MET 5/5 before and 5/5 after, with the same item selected for criteria 1, 3, 4 and 5. One query returned fewer results: the criterion 3 query went from 10 matches to 4, because listings that only matched "black" or "white" were dropped. The top result stayed the same.
- **What the criteria can't show.** Because all five passed before the change, they couldn't show any improvement. None of them checks whether the search returned the kind of item the user asked for, which is the gap I named in my diagnosis. The diagnostics are the evidence, not the criteria table.

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

No criterion is still missed, since all five were MET in both runs. These problems are still there:

1. **The fix only covers colors.** Other descriptive words can still make a match on their own. I checked after the change: "vintage peacoat" returns *Vintage Levi's 501 Jeans*, and "oversized peacoat under $50" returns *Oversized Flannel Shirt*, because "vintage" and "oversized" aren't colors. The same rule would need to cover descriptive words like these (style words, fits, conditions). I stopped at colors to keep this to one change I could measure.
2. **No criterion checks search relevance.** Criterion 1 passes as long as a fit card comes back, even for the wrong item. I'd tighten it so the selected item has to be the type of item the query names, and add the peacoat and boots queries to it.
3. **A requested color that isn't available is ignored without a word.** "black boots" returns tan boots, because they're the only boots listed. That's better than a shirt, but the user isn't told the color didn't match. I'd add a note like "No black boots — the closest is tan."
4. **The model-unavailable message gives the wrong fix for an outage.** On a 503 "high demand" error, it still says to check `GEMINI_API_KEY`, and it says the outfit step didn't run even when `suggest_outfit` had succeeded (see Failure tests). I'd check the cause and which step failed before writing the message.
5. **Prices written in words are ignored.** The regex parser misses phrasings like "nothing over fifty dollars", so the price ceiling quietly drops off and the words become part of the search.
6. **The trace shortens the outfit to about 110 characters**, so criterion 3 can only compare the start of the outfit text, not the whole thing.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
