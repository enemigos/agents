# Treg budget

## Problem

`subagents/LIBRARIAN.md` limits web calls, source calls, and opened sources for each tier. It does not limit `treg` calls. A `treg` call bills the team account on each hit, repeats included.

The GTM section says to send `X-Treg-Route-Max-Cost` to cap one call. Nothing caps the total for a run. A `deep` lead-generation run with 40 companies and several contacts each can make hundreds of find and verify calls. The `research` skill can start several `deep` Librarians in parallel, so the cost can multiply.

The Budget output also does not report `treg` calls or spend. So nobody sees the cost after the run.

## Fix

### 1. Add paid calls and spend to the Budget table

Replace the table in `# Budget`:

```md
A paid call is one `treg` call, by CLI or HTTP. A failed paid call still counts.

| Tier | Rounds | Web calls per round | Source calls | Opened sources | Paid calls | Spend cap |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `lookup` | 1 | 3 | 2 | 2 | 3 | USD 0.25 |
| `standard` | 5 | 6 | 6 | 10 | 20 | USD 2 |
| `deep` | 8 | 8 | 12 | 16 | 60 | USD 10 |
```

These numbers are starting points. Set them from real `catalog_get` prices after a few runs.

### 2. Check the price before each new endpoint

Add to `# GTM with treg`:

```md
- Before the first call to an endpoint in a run, read its price with `catalog_get`.
- Reserve each paid call before you send it: add 1 to the call count and its maximum price to the spend total. Send a batch of parallel paid calls only when the whole batch fits in the limits that remain. After a call returns, replace its reserved price with the billed price.
- Send `X-Treg-Route-Max-Cost` on every routed call. Set it to the price you reserved for that call.
- Stop paid calls when the next call does not fit in the paid-call limit or the spend cap. Report the companies or people that remain without a contact.
- For lead generation, find and verify contacts only for companies that already pass the fit check. Do not enrich a company before it passes.
```

The reservation rule stops two parallel calls from each passing the check against the same remaining budget. The last rule stops spend on companies that the shortlist then drops.

### 3. Report the spend

Change the Budget output line:

```md
- `Budget`: rounds, web calls, source calls, opened sources, paid calls, and spend in USD.
```

The `lookup` output omits the Budget section today. Add to `# Output`:

```md
When the run made a paid call, include `Budget` in every tier, `lookup` included.
```

### 4. Let the caller set the cap

The rule "If the request gives lower limits, use them" already covers the new columns. Add one line to `skills/research/SKILL.md` Background research step 3: give each GTM lane a paid-call limit and a spend cap, and make the sum of all lanes fit the cap for the run. When a follow-up lane starts after collection, give it only the cap that the earlier lanes did not spend.

## Files to change

- `subagents/LIBRARIAN.md`: Budget table, GTM rules, Budget output line, `lookup` output rule.
- `skills/research/SKILL.md`: lane brief limits for paid calls.
