# FitFindr — Starter Kit

This starter kit now includes a complete baseline implementation for Project 2.

## What's Included

```
ai201-project2-fitfindr-starter/
├── data/
│   ├── listings.json          # 40 mock secondhand listings
│   └── wardrobe_schema.json   # Wardrobe format + example wardrobe
├── utils/
│   └── data_loader.py         # Helper functions for loading the data
├── planning.md                # Your planning template — fill this out first
├── tools.py                   # Required tool implementations
├── agent.py                   # Planning loop + session state orchestration
├── app.py                     # Gradio UI wired to the agent
└── requirements.txt           # Python dependencies
```

## Setup

```bash
pip install -r requirements.txt
```

Set your Groq API key in a `.env` file (get a free key at [console.groq.com](https://console.groq.com)):
```
GROQ_API_KEY=your_key_here
```

## The Mock Listings Dataset

`data/listings.json` contains 40 mock secondhand listings across categories (tops, bottoms, outerwear, shoes, accessories) and styles (vintage, y2k, grunge, cottagecore, streetwear, and more).

Each listing has: `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.

Load it with:
```python
from utils.data_loader import load_listings
listings = load_listings()
```

## The Wardrobe Schema

`data/wardrobe_schema.json` defines the format your agent uses to represent a user's existing wardrobe. It includes:

- `schema`: field definitions for a wardrobe item
- `example_wardrobe`: a sample wardrobe with 10 items you can use for testing
- `empty_wardrobe`: a starting template for a new user

Load an example wardrobe with:
```python
from utils.data_loader import get_example_wardrobe
wardrobe = get_example_wardrobe()
```

## Where to Start

1. Install dependencies and set `GROQ_API_KEY` in `.env`.
2. Verify the data loads correctly by running `python utils/data_loader.py`.
3. Run the CLI sanity check in `agent.py` or launch `python app.py`.

## Tool Interfaces

### `search_listings(description, size=None, max_price=None) -> list[dict]`
- Loads listings from local mock data.
- Applies optional size/price filters.
- Scores and sorts by relevance to query keywords.
- Returns `[]` when no match or data load failure.

### `suggest_outfit(new_item, wardrobe) -> str`
- Uses selected listing + wardrobe to generate 1–2 outfit ideas.
- If wardrobe is empty, switches to general styling guidance.
- If LLM call fails, returns deterministic fallback advice.

### `create_fit_card(outfit, new_item) -> str`
- Produces a short social caption for the final look.
- Validates outfit input and returns explicit error text if missing.
- If LLM call fails, returns deterministic fallback caption.

## Planning Loop (Agent Behavior)

`run_agent(query, wardrobe)` uses session-based planning:
1. Parse `query` into `description`, `size`, `max_price`.
2. Call `search_listings`.
3. If empty, retry with loosened constraints (drop size, then drop price).
4. If still empty, return early with a helpful error.
5. Select top listing and call `suggest_outfit`.
6. Call `create_fit_card` from outfit + selected item.
7. Return full session dict with all intermediate and final outputs.

## State Management

A single session dict carries:
- original query
- parsed query fields
- search results
- selected item
- wardrobe input
- outfit suggestion
- fit card
- error (if any)

This ensures outputs from one tool become inputs for the next without user re-entry.

## Error Handling Strategy

- **No search results:** agent retries with relaxed filters, then prompts user to broaden query.
- **Empty wardrobe:** outfit tool returns general styling guidance.
- **Missing outfit input:** fit card tool returns explicit error string.
- **LLM/API failures:** outfit and fit card tools return fallback text instead of crashing.

## Run

```bash
python app.py
```

Then open the local Gradio URL printed in your terminal.
