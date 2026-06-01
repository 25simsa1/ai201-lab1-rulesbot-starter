# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user query and a list of retrieved rule chunks, generate a response that directly answers the question using only the retrieved text as context. The response must be grounded — it should not draw on the model's general knowledge of board games, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's original question |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"game"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:
- Answer the question using only the retrieved rule text
- Identify which game the answer comes from
- Acknowledge clearly when the answer is not found in the loaded rules

Returns a fallback string (not an error) when `retrieved_chunks` is empty.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Context formatting

*How will you format the retrieved chunks before passing them to the LLM? Describe the structure — not the code. Consider: will you label chunks by game? Include distance scores? Separate chunks with delimiters?*

```
Each chunk is labeled with its game name and separated by a divider:

[Game: Catan]
When a 7 is rolled, no one collects any resources...

---

[Game: Catan]
The robber must be moved to a different terrain hex...

---

Distance scores are not included — they're internal metadata,
not useful context for the LLM.
```

---

### System prompt — grounding instruction

*Write the exact system prompt instruction you will use to prevent the model from answering beyond the retrieved text. This is the most important design decision in this function.*

```
You are RulesBot, a board game rules assistant. Answer the user's 
question using ONLY the rule passages provided below. Do not use 
your general knowledge of board games. If the answer is not 
contained in the passages, say so clearly.
```

---

### System prompt — citation instruction

*Write the exact instruction you will use to tell the model to identify which game its answer comes from.*

```
Always identify which game your answer comes from by name 
(e.g. "According to the Catan rules...").
```

---

### Fallback behavior

*What should the response say when the answer isn't found in the loaded rule books? Write the exact fallback message.*

```
I couldn't find an answer to that in the loaded rule books. 
Try rephrasing your question or ask about a specific game: 
Catan, Clue, Codenames, Monopoly, Pandemic, Risk, 
Ticket to Ride, or Uno.
```

---

### Handling low-relevance chunks

*`retrieved_chunks` may include chunks with high distance scores (weak relevance). Will you filter these out before building context, pass them all in, or handle them another way? What are the tradeoffs?*

```
Pass all retrieved chunks in regardless of distance score.
Tradeoff: may include weak context, but the system prompt 
instructs the model to only answer from the provided text — 
if no chunk is relevant, the model should say so. Filtering 
risks discarding the only available context for niche questions.
```

---

### Message structure

*Describe how you will structure the messages list for the API call — what goes in the system message vs. the user message?*

```
System message: grounding instruction + citation instruction + 
                the formatted rule passages as context

User message: the user's original question

Keeping context in the system message separates it clearly 
from the question, and reinforces that it's the authoritative 
source the model must use.
```

---

## Implementation Notes

*Fill this in after implementing and testing.*

**Test query and response:**

```
Query: How do you set up the board in Catan?
Response: According to the Catan rules, arrange terrain hexes randomly...
Correctly grounded? Yes
Cited the right game? Yes


```

**One thing you changed from your original spec after seeing the actual output:**

```
Nothing major, grounding instruction worked as written on the first try.
```
