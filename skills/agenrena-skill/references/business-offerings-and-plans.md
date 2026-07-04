# Business Offerings and Plans

## Contents

- [Choose the output](#choose-the-output)
- [Get search options](#get-search-options)
- [Confirm conditions](#confirm-conditions)
- [Search and evaluate offerings](#search-and-evaluate-offerings)
- [Share offerings](#share-offerings)
- [Build and edit plans](#build-and-edit-plans)

## Choose the Output

Use this workflow when the user wants to find business services or build a plan from multiple services. Use the same search and clarification process in both cases, then choose the output from the user's intent:

- For a service search, comparison, or list of alternatives, present matching offerings and include each offering's `share_url` when available. Do not create a plan merely because several results were returned.
- For a coordinated flow that combines multiple services, such as a half-day itinerary or dinner followed by a massage and a ride, build a plan from the selected offerings.

For example, "find a restaurant" and "give me a few restaurant options" require offering links. "Design a half-day itinerary" requires a plan.

## Get Search Options

Before searching, get the current search options for the target country:

```bash
agenrena businesses offerings search-options --country-code <country_code>
```

Use the returned categories, service periods, tags, tag limits, and area codes when building the search. To get cities within a returned state, query its options:

```bash
agenrena businesses offerings search-options --country-code <country_code> --state-code <state_code>
```

Treat search options as clarification choices, not defaults. After retrieving them, ask the user about the available conditions that are relevant to the request and would materially narrow the results. Use human-readable labels rather than raw keys, and ask a concise set of high-impact questions instead of listing every option.

## Confirm Conditions

Before searching:

- Reuse conditions the user already stated; do not ask for them again.
- For itinerary planning, confirm the travel dates or season, party size, budget, and interests. A condition is also confirmed when the user explicitly says it is unrestricted.
- Map an explicit preference to a returned option when the meaning is clear. Ask when the mapping is ambiguous.
- Do not invent unstated preferences or choose values merely to avoid asking the user.
- If the user has no preference for a condition, omit that filter rather than choosing a value for them.

Classify confirmed tags before searching:

- Use a required tag only when the user explicitly makes it non-negotiable. Every required tag must match.
- Use preferred tags for interests, scenery, food, mood, and other general preferences. Preferred tags rank results without excluding non-matches.
- Do not promote a preference to a required tag. Do not use the same tag as both required and preferred.
- Respect the `tag_limits` returned by search options. If the user has more tags than allowed, ask which matter most.

For example, map "must have parking; would like mountain or ocean views" to `--required-tag parking --preferred-tag mountain_view --preferred-tag ocean_view`.

Do not run the search until the relevant conditions are confirmed or the user explicitly leaves them unrestricted. This avoids building a plan around guessed constraints and having to redo it later.

## Search and Evaluate Offerings

Search for offerings that match the user's request:

```bash
agenrena businesses offerings search --category <category> [--state-code <state_code>] [--city-id <city_id>] [--party-size <number>] [--price-max <amount>] [--service-period <period>] [--required-tag <tag>] [--preferred-tag <tag>] [--latitude <latitude> --longitude <longitude>] [--page <number>]
```

Use only confirmed conditions as optional filters. Repeat `--service-period`, `--required-tag`, or `--preferred-tag` to provide multiple values. Use `--latitude` and `--longitude` together.

For a multi-service plan, search each needed category separately and reuse confirmed conditions that apply across the whole plan.

If the results are too broad, rank them with confirmed preferred tags or coordinates before asking whether the user wants another hard filter. If there are no results, explain which required tags or other filters may be too restrictive and ask which one the user wants to relax. Do not relax or replace required conditions without confirmation. If no result matches a preferred tag, keep the remaining results and state that the preference was not matched.

To inspect more detail for a business, use the result's `business.identity_id` to list that business's offerings:

```bash
agenrena businesses offerings list --identity-id <identity_id>
```

## Share Offerings

Do not fabricate offering IDs or share URLs. If the user asked for service options rather than a coordinated flow, stop after presenting the best matches with their `share_url` values.

## Build and Edit Plans

Create a plan when the user asks for a coordinated multi-service flow. Use `platform_offering` items for selected Agenrena offerings. Use `note` items only for necessary steps that are not represented by an offering.

```bash
agenrena plans create --json '<plan_json>'
```

The plan JSON may include `title`, `intent_text`, `start_date`, `end_date`, `metadata`, and nested `items`. Every field is optional; `end_date` must not be earlier than `start_date`.

Each item supports `source_mode` (`platform_offering` or `note`, default `platform_offering`), `offering_id`, `day_index`, `position`, `start_time`, `title`, `note`, `agent_reason`, and `metadata`.

- A `platform_offering` item must include `offering_id`. Its title, price, and details are read live from the offering, so do not set `title`.
- A `note` item must include a `title` or a `note`.
- Set `agent_reason` to briefly explain why you chose the item; it is shown to the owner.

An offering item:

```json
{"source_mode":"platform_offering","offering_id":"<offering_id>","day_index":0,"start_time":"18:30:00","agent_reason":"Highest-rated ramen near the hotel"}
```

A note item:

```json
{"source_mode":"note","title":"Walk to the station","note":"~10 min on foot","day_index":0}
```

`plans create` and `plans get` return the plan nested under `data.plan`. Read the key paths from there:

```json
{
  "ok": true,
  "data": {
    "plan": {
      "id": "7bd7a079-62d3-4a36-9e12-90c0d219e0cc",
      "revision": 0,
      "title": "...",
      "items": [
        {"id": "96e6124e-533c-44c9-b1f2-33484c3f7006", "source_mode": "platform_offering", "offering_id": "...", "day_index": 0}
      ]
    }
  }
}
```

- `data.plan.id` is the `--plan-id` for `plans get` and every item mutation.
- `data.plan.revision` is the `--expected-revision` for the next item mutation.
- `data.plan.items[].id` is the `--item-id` for update/delete and the `id` for each entry in a reorder array.

Read an existing plan before changing its items:

```bash
agenrena plans get --plan-id <plan_id>
```

Add, update, delete, or reorder items:

```bash
agenrena plans items add --plan-id <plan_id> --expected-revision <revision> --json '<item_json>'
agenrena plans items update --plan-id <plan_id> --item-id <item_id> --expected-revision <revision> --json '<item_patch_json>'
agenrena plans items delete --plan-id <plan_id> --item-id <item_id> --expected-revision <revision>
agenrena plans items reorder --plan-id <plan_id> --expected-revision <revision> --json '<ordered_items_json>'
```

Plan rules:

- Read the plan's current revision from `data.plan.revision` and pass it as `--expected-revision` for every item mutation. After a successful mutation, read `data.plan.revision` from the returned plan again for the next mutation. Item ids for update, delete, and reorder are at `data.plan.items[].id`.
- Do not put `expected_revision` inside item JSON; pass it only with `--expected-revision`.
- Treat item update JSON as a non-empty patch.
- For reorder, pass the full ordered array of items, each containing `id` and `day_index`.
- If a mutation fails because the revision is stale, get the plan again and reconsider the change before retrying. Preserve newer changes and ask the user if the requested change cannot be applied safely.
