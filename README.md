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

FitFindr lets a user describe a clothing item they want, with an optional size and price limit. It searches the sample listings and uses the first match to suggest outfits with the user's saved wardrobe. It then writes a short caption with the item, price, and platform. If nothing matches, it stops and suggests changing the search words, size, or budget.

---

## Tool Inventory

Milestone 1 data review: I read listings `lst_001` through `lst_006`. Listing fields are `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`. Sizes are strings such as `S/M`, `W30 L30`, and `XL (oversized)`, and `brand` can be null. A wardrobe contains an `items` list. Each item has `id`, `name`, `category`, `colors`, `style_tags`, and `notes`. Notes can be null. An empty wardrobe has `"items": []`.

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Loads listings using utils/data_loader.py::load_listings and finds items that match the description, size, and price limit.
- **Inputs:** description (str), size (str or None), and max_price (float or None). None means no filter for that field.
- **Returns:** A list of matching listing dicts, best match first. Each has id, title, description, category, style_tags, size, condition, price, colors, brand, and platform.
- **When it has nothing:** Returns an empty list, [].

Size matching uses whole size values: M matches S/M, L does not match XL, 8 matches US 8 but not US 8.5, and W30 matches W30 L30.

### `suggest_outfit`

- **What it does:** Uses generate.py::generate to suggest one or two ways to wear the new item with clothes the user owns.
- **Inputs:** new_item (listing dict) and wardrobe (dict with an items list).
- **Returns:** A string describing one or two outfits using the new item and clothes from the wardrobe.
- **When it has nothing:** If the wardrobe has an empty items list, returns general styling advice instead.

### `create_fit_card`

- **What it does:** Uses generate.py::generate to write a short caption about the item and outfit.
- **Inputs:** outfit (str) and new_item (listing dict).
- **Returns:** A string containing a two-to-four sentence caption that mentions the item, price, and platform.
- **When it has nothing:** If outfit is empty or only whitespace, returns "Cannot create a fit card without an outfit suggestion." without calling the model.

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

**Branch rule:** If search_listings returns [], put a message in session["error"] suggesting different search words, a different size, or a higher price limit, then stop. Do not call the other two tools. Otherwise, save the first result in session["selected_item"]. Read that item and the wardrobe from the session to call suggest_outfit, then save its answer in session["outfit_suggestion"]. Read the outfit and item from the session to call create_fit_card, and save the caption in session["fit_card"]. This is the plan; the loop will be built in Milestone 5.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex extracts the size and price limit, then removes them from the search description. No model call is used for parsing.

**What moves through the session:** parsed holds the search inputs, search_results holds the matching listings, and selected_item holds the first match. suggest_outfit reads selected_item and wardrobe from the session and saves its output in outfit_suggestion. create_fit_card reads outfit_suggestion and selected_item and saves its output in fit_card.

---

## Sample Run

### Milestone 1: starter check (before implementation)

```
$ .venv/bin/python app.py ask 'vintage graphic tee under $30'

  The planning loop isn't built yet — see the TODO in agent.py.

0 model calls this session
```

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ .venv/bin/python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two outfit suggestions using the new Y2K baby tee and items exclusively from your wardrobe:

### Outfit 1: Y2K Streetwear Vibe
Lean into the early 2000s aesthetic by pairing the fitted graphic tee with baggy denim and chunky sneakers.
* **Top:** Y2K Baby Tee — Butterfly Print
* **Bottoms:** Baggy straight-leg jeans, dark wash (w_001)
* **Shoes:** Chunky white sneakers (w_007)
* **Accessories:** Black crossbody bag (w_010)

### Outfit 2: Casual Earth-Tone Mix
Balance the pink and purple butterfly graphic with relaxed tan trousers and a vintage jacket for a retro-casual look.
* **Top:** Y2K Baby Tee — Butterfly Print
* **Bottoms:** Wide-leg khaki trousers (w_002)
* **Outerwear:** Vintage black denim jacket (w_006)
* **Shoes:** Chunky white sneakers (w_007)
* **Accessories:** Brown leather belt (w_009)

  Fit card: Channel total early 2000s nostalgia with this super cute Y2K Baby Tee — Butterfly Print, featuring a fitted cropped fit and dreamy pastel graphics. It has a playful retro-casual vibe that looks effortless paired with baggy denim or wide-leg trousers. Grab it now on depop for just $18.00!

2 model calls this session, 1198 prompt + 306 output tokens
```

```
$ .venv/bin/python app.py ask 'designer ballgown size XXS under $5'

  No listings matched. Try different search words, a different size, or a higher price limit.

0 model calls this session
```

I printed the completed session and checked that the selected item was the first search result and the same item passed to suggest_outfit. The empty-search path left fit_card as None and skipped both model tools.

**The three tools, tested one at a time**

```
$ .venv/bin/python -c "from tools import search_listings; print(search_listings('graphic tee', size='L', max_price=30))"
[{'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ .venv/bin/python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[5], get_example_wardrobe()))"
Here are two outfits featuring the new Graphic Tee and pieces from your wardrobe:

### Outfit 1: Grunge Streetwear
Lean into the vintage, worn-in aesthetic of the tee with dark denim and chunky boots.
* **Top:** Graphic Tee — 2003 Tour Bootleg Style
* **Outerwear:** Vintage black denim jacket
* **Bottoms:** Baggy straight-leg jeans, dark wash
* **Shoes:** Black combat boots
* **Accessory:** Black crossbody bag

### Outfit 2: Casual Y2K Contrast
Pair the boxy black tee with lighter earth tones for a casual, textured look, finished off with classic streetwear sneakers.
* **Top:** Graphic Tee — 2003 Tour Bootleg Style
* **Bottoms:** Wide-leg khaki trousers
* **Accessory:** Brown leather belt
* **Shoes:** Chunky white sneakers
```

```
$ AI201_CACHE=0 .venv/bin/python -c "from tools import create_fit_card; from utils.data_loader import load_listings; item = load_listings()[5]; [print(f'Run {i + 1}: {create_fit_card(\"dark wash jeans and chunky white sneakers\", item)}') for i in range(3)]"
Run 1: Channeling that ultimate effortless grunge energy with this Graphic Tee — 2003 Tour Bootleg Style. It has the best worn-in feel and looks so cool paired with dark wash jeans and chunky white sneakers. Grab it on depop for $24.00 before it's gone!
Run 2: Pair this Graphic Tee — 2003 Tour Bootleg Style with dark wash jeans and chunky white sneakers for the ultimate grunge streetwear look. Grab it now for $24.00 over on Depop!
Run 3: Channeling serious effortless grunge energy with this Graphic Tee — 2003 Tour Bootleg Style. Pair it with dark wash jeans and chunky white sneakers for the ultimate off-duty look. Snag this worn-in favorite for just $24.00 live on my Depop right now!
```

The three captions used the same input with caching off and had different wording. They included the item, price, and platform, but sounded more like sales posts than personal outfit posts.

**Empty cases**

```
$ .venv/bin/python -c "from tools import search_listings; print(search_listings('designer ballgown', size='XXS', max_price=5))"
[]

$ .venv/bin/python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('   ', load_listings()[5]))"
Cannot create a fit card without an outfit suggestion.
```

```
$ .venv/bin/python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[5], get_empty_wardrobe()))"
Here are a couple of styling ideas for your new vintage-style graphic tee:

**1. 90s Grunge Casual**
*   **Bottoms:** Distressed light-wash blue jeans with a relaxed or straight-leg fit.
*   **Footwear:** Scuffed black canvas sneakers or chunky lace-up combat boots.
*   **Layering:** An oversized plaid flannel shirt worn open over the tee, or tied casually around the waist. 

**2. Elevated Streetwear**
*   **Bottoms:** Pleated black wide-leg trousers or utilitarian cargo pants.
*   **Footwear:** Retro-style leather sneakers (like worn-in runners).
*   **Accessories:** A silver chain necklace, a canvas crossbody bag, and a beanie to lean into the effortless, edgy aesthetic.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked the AI to help write the three tool specs before building them.
- *What came back:* It mixed a long explanation of search behavior into the input and output descriptions.
- *What I changed:* I asked it to shorten those sections to the input types and return values, and to separate invariants from behavior specs.

**Moment 2**

- *What I asked for:* I asked the AI to help build the tools.
- *What came back:* It generated the code and ran both a matching query and an empty search.
- *What I changed:* It generated complete functions that appeared to behave as expected. I conducted a series of tests for expected behaviors and edge cases. I did not need to change anything since it worked as expected. But I did ask to refactor a block of code that appeared to be repetitive.

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
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

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

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

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

```

**Empty search**

```

```

**On the MCP move:** I registered search_listings in mcp_server.py and changed agent.py::run_agent to call it through mcp_client.call_tool. The server listed the expected inputs, direct and MCP calls returned the same three listing dicts, and the full query completed.



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

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
