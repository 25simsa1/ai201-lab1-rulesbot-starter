# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"game"` | `str` | The game name this chunk came from |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Query approach

*Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?*

```
[your answer here]
```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

```
One item in the return list looks like:

{
  "text": "When a 7 is rolled, no one collects any resources...",
  "game": "Catan",
  "distance": 0.142
}

Where each field comes from:
- "text"     → results["documents"][0][i]
- "game"     → results["metadatas"][0][i]["game"]
- "distance" → results["distances"][0][i]

where i is the index of each result in the list.
```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```
_collection.query() returns nested lists because it supports multiple 
queries at once. Since we only pass one query at a time, our results 
are always at index [0]:

  results["documents"][0]  → list of chunk texts
  results["metadatas"][0]  → list of metadata dicts
  results["distances"][0]  → list of distance scores

Accessing results["documents"] without [0] returns a list of lists,
not the actual chunks — this is the most common indexing bug.
```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```
Return all n_results without filtering. If a query matches no chunks 
well, the LLM will receive weak context but can still say it doesn't 
know. Filtering risks discarding valid chunks for niche questions that 
naturally produce higher distances. The tradeoff: more noise in context 
vs. risk of returning nothing useful.
```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```
[your answer here]
```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```
Query: What happens when you roll a 7 in Catan?
Top result game: Catan
Distance score: 0.471
Does it make sense? Yes, the chunk mentions hex resources and rolling,
which is related to the 7/robber mechanic, though not a perfect match.
```

**One thing about the query results that surprised you:**

```

Distance scores were higher than expected (0.471–0.625). Even a 
directly relevant Catan chunk scored ~0.47, suggesting 300-character 
chunks may be too small to carry strong semantic signal for this query.

```
