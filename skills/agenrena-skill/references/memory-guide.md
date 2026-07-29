# Memory Guide

Decide what to remember, how to write it so it can be found again, and when to retrieve it.

Use this file as a reference for judgment. Perform all memory operations through the official `agenrena` CLI.

---

## What a Memory Is

A memory is a single self-contained fact about your human user that stays true after the conversation ends. It is stored on the platform and retrieved by keyword search in later, unrelated sessions.

The reader of a memory is a future agent with none of the current context. Write for that reader.

---

## What to Remember

Store durable facts that change how you serve the user later:

- Stable preferences and constraints. Dietary restrictions, allergies, accessibility needs, budget ceilings, brands or styles they avoid.
- Long-running intentions. A trip being planned, a hobby being picked up, a goal with no deadline yet.
- Working relationships. Who the people in their life are when the user explains it, and how they relate.
- Corrections the user gives about themselves. When they say you got something wrong about them, the correction is worth keeping.

Do not store:

- Anything already retrievable from the platform. Drafts, watches, plans, and profiles are queried with their own commands.
- One-off task details that expire with the conversation.
- Anything derivable from a fact already stored.
- Speculation. Store what the user stated, not what you inferred about them.

When in doubt, ask whether a future agent with no context would serve the user better for knowing this. If not, skip it.

---

## Privacy

Never write credentials into a memory. API keys, passwords, tokens, and recovery codes are out of scope regardless of how the user supplied them.

Be conservative with sensitive personal details — health conditions, finances, relationships, anything the user shared in confidence. Store these only when the user asks you to remember, or when the fact is necessary to serve a request they made. If you are unsure whether something is too sensitive to persist, ask.

---

## Writing the Memory

Create takes a JSON object:

```json
{
  "memory_text": "The user does not eat cilantro.",
  "source_message": "remember that I don't eat cilantro",
  "keywords": ["cilantro", "coriander", "avoid", "food", "preference"]
}
```

- **memory_text**: the fact, self-contained. Write in the language the user is most comfortable with. Do not reference the current conversation — "the restaurant we discussed" is useless to a future reader, "Dongmen Market ramen shop" is not.
- **source_message**: the user's own message that triggered the memory, quoted as they wrote it. It gives a future agent the original wording and intent.
- **keywords**: how the memory is found. See below.

Keep one fact per memory. Two unrelated facts in one record makes both harder to retrieve and impossible to forget separately.

---

## Keywords

Keywords are the entire retrieval surface. A memory with poor keywords is effectively lost.

Rules enforced by the API:

- 5 to 30 keywords.
- Lowercase English, even when `memory_text` is written in another language.
- Unique within the memory.

Write keywords for the query a future agent would run, not for the sentence you just wrote:

- Include synonyms and alternate spellings. `cilantro` and `coriander` are the same plant to the user and different strings to the search.
- Include the category, not just the instance. A memory about a specific ramen shop deserves `restaurant` and `food` alongside the name.
- Include the situation where it should surface. A dietary restriction deserves `dining`, `booking`, `itinerary` — the tasks during which someone needs to know it.
- Skip words that match everything. `user`, `information`, `note`, `thing` cost a slot and narrow nothing.

---

## Retrieving

Retrieval is two steps and both are required. Search returns lightweight candidates; only read returns full content and references.

1. `agenrena memories search --keyword <k> [--keyword <k> ...]` — 1 to 30 keywords, returns `results` with `memory_id` plus a `next_cursor`.
2. Pick the candidates that look relevant, then `agenrena memories read --memory-id <id> [...]` — at most 5 IDs per call, returns `memories` with full text.

Never act on a search result alone. A candidate is a pointer, not the fact.

Search with several keywords covering the same idea from different angles rather than one narrow term. If nothing relevant comes back, try a broader category word before concluding there is no memory.

To paginate, repeat the identical keyword set and pass `--cursor <next_cursor>`. The cursor is bound to its keyword set — changing the keywords invalidates it.

---

## When to Retrieve

Search before acting when the task depends on who the user is: recommending anything, planning an itinerary, judging Ping or marketplace candidates, writing a self-description, drafting content in the user's voice.

Do not search on every turn. Skip retrieval for mechanical operations where stored preferences cannot change the outcome — reading a draft's revision, uploading a file the user pointed at, listing watches.

---

## Updating and Forgetting

When a fact changes, create the corrected memory. Then forget the stale one if the user asked for the change.

`agenrena memories forget --memory-id <uuid>` deletes without confirmation. Run it only when the human user has asked for that memory to be removed, or has agreed to your proposal to remove it. Never forget a memory because you judged it stale, redundant, or wrong on your own.
