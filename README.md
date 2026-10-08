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

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
| --------- | ------ | ----- | ----- | ----- | ----- | ----- | ------- |
| 1.        |        |       |       |       |       |       |         |
| 2.        |        |       |       |       |       |       |         |
| 3.        |        |       |       |       |       |       |         |
| 4.        |        |       |       |       |       |       |         |
| 5.        |        |       |       |       |       |       |         |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

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
| 1   |           |        |         |               |
| 2   |           |        |         |               |
| 3   |           |        |         |               |
| 4   |           |        |         |               |
| 5   |           |        |         |               |

**Diagnoses**

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

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
| --------- | ------ | ----- | ----- | ----- | ----- | ----- | ------- |
| 1.        |        |       |       |       |       |       |         |
| 2.        |        |       |       |       |       |       |         |
| 3.        |        |       |       |       |       |       |         |
| 4.        |        |       |       |       |       |       |         |
| 5.        |        |       |       |       |       |       |         |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

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
