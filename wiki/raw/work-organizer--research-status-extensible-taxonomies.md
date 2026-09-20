> Pulled from work-organizer wiki (`raw/research-status-extensible-taxonomies.md`), dated 2026-09-18 — research for a different project (work-organizer command-center); reused here as prior art.

# Designing an Extensible Status Field Without Tag Soup

## Summary

Every mature API style guide converges on the same conclusion: a status field expected to grow should **not** be modeled as a rigid closed enum. The recommended shape is a **two-layer design** — a small, frozen set of top-level *categories* (whose membership never changes) sitting above an *open, additive* set of granular sub-statuses (whose membership grows freely, additive-only, never by deletion). This gives you the stability closed enums provide for building reliable views, while preserving the freedom to add states you haven't anticipated. The four known values (`in-flight`, `needs-followup`, `blocked`, `backlog`) map cleanly onto this: keep a stable category layer for querying and rollups, treat the granular status as an open string with a documented-but-not-enforced value list, and reserve an `unknown`/default sentinel so consumers never choke on a value they don't recognize.

## 1. Open vs. closed enums in API/schema design

The strongest guidance is unambiguous: default to *open* (extensible) enums whenever the value set might grow. Microsoft's Azure API Guidelines introduce the term "extensible enum," which "indicates that the set of values should be treated as only a *partial* list," and direct authors to "use extensible enums unless you are absolutely certain the symbol set will never expand." Critically, they instruct that documentation tell consumers "new values may appear in the future so that customers write their code today expecting these new values tomorrow," and they forbid removal outright: "DO NOT remove values from your enumeration list as this breaks customer code." (https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)

Google's AIP-126 draws the closed/open line by *change frequency*: "For enumerated values where the set of allowed values changes frequently, APIs should use a `string` field instead, and must document the allowed values" — with a rule of thumb of no more than roughly one new enum value per year before you should switch to a documented string. It also requires the first value be an `_UNSPECIFIED` sentinel, and asks APIs to "document whether the enum is frozen or they expect to add values in the future." (https://google.aip.dev/126)

Zalando's RESTful API Guidelines encode the same instinct as a rule: Rule 112, "SHOULD use open-ended list of values (via `examples`) for enumeration types," recommends documenting values as *examples* rather than a strict `enum` "when future expansion is anticipated," so clients "gracefully handle new values the API may introduce without breaking." (https://opensource.zalando.com/restful-api-guidelines/)

The through-line: a closed enum is a contract that the value set is complete forever. For a field explicitly required to stay extensible, that contract is the wrong one — string-with-documented-values (an open enum) is the recommended form.

## 2. Category + sub-status splitting

The pattern that keeps an open field from becoming unstructured is to layer a *stable, closed* top-level category over the *open* granular statuses. Two mainstream systems demonstrate it:

- **GitHub issues** use a two-tier system: a binary, frozen top-level state (open/closed) plus a finer *reason* when closing (completed vs. not planned). "The top-level state signals *whether* work is finished, while the reason explains *why*." Orthogonal to both sits an open-ended **label** layer for custom classification. The fixed layer never changes; the granular layers grow. (https://docs.github.com/en/issues/tracking-your-work-with-issues/administering-issues/closing-an-issue)
- **Jira** formalizes this as *status categories*: every workflow status — including unlimited custom statuses an admin invents — maps to exactly one of three fixed categories (To Do / In Progress / Done). Teams add statuses freely; the three categories that dashboards and rollups depend on stay constant. (https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-workflow-status-categories/)

Applied to the four current values, the stable categories might be `open` (contains `in-flight`, `needs-followup`, `backlog`), `blocked`, and later `closed`/`done`. New sub-statuses land inside an existing category without touching the category layer, so every saved view, filter, and rollup written against categories keeps working unchanged.

## 3. Versioned schema / migration when a value must be added

Adding a value is safe when the schema is treated as **additive-only** and consumers are built to tolerate the unknown:

- **Sentinel default.** Protobuf requires the first enum value be a zero `*_UNSPECIFIED`, precisely so that "when new values are added to an enum, old clients will see the field as unset and the getter will return the default value." An explicit unknown/default bucket is the mechanism that lets a consumer receive a value it predates without crashing. (https://protobuf.dev/best-practices/dos-donts/)
- **Additive, ordered rollout.** The same guidance is additive-only: never remove or repurpose an existing value, and when introducing one, "put the new name last to give services time to pick it up." Reads are updated to recognize the new value before writers begin emitting it.
- **Dual-write / verify for data migrations.** Stripe's migration writeup generalizes the safe path for changing stored representations: dual-write old and new, shift reads only after verifying consistency, then shift writes — always "incremental… never more than a few hundred lines at one time." (https://stripe.com/blog/online-migrations)

For a markdown+YAML store with no schema enforcement, the equivalent discipline is: add the value to the documented list first, teach every view/query the new value (or a fallback branch) before any note carries it, and never rename or delete a value already written to disk.

## 4. Tags vs. closed enums for "values I haven't thought of yet"

Free-form tags are the maximal-flexibility endpoint, and they are the natural draw for the "future values I haven't anticipated" case — but the tradeoff is well documented. Folksonomy/tagging is "a trade-off between traditional centralized classification and no classification at all": it requires no schema and reflects how people actually think, but it sacrifices **consistency and precision** — "tags are often ambiguous and overly personalized," with no built-in handling of "synonyms, acronyms, homonyms, misspellings, and grammatical variations." Controlled vocabularies enforce standardization and browsable hierarchy, but are "exclusionary by nature." (https://en.wikipedia.org/wiki/Folksonomy)

For a *status* field specifically — one that reliable views, "what's blocked" queries, and rollups depend on — pure tags give up exactly what a status field exists to provide: a guaranteed-valid, dedupe-free value you can group by. The folksonomy research offers a hedge, though: "consensus around stable distributions and shared vocabularies does emerge" over time. That argues for a hybrid: capture in a structured status field, allow free-form tags *alongside* for the genuinely unclassified, and promote a tag into a documented sub-status once it recurs.

## Recommendation

Design the status field as a **documented open enum under a frozen category layer**:

1. **Frozen category layer** — a tiny, closed set (`open`, `blocked`, `closed`, …) that never gains or loses members. All views, filters, and rollups query this layer, so they never break.
2. **Open granular status** — a string whose known values are listed in the schema *as documented examples, not enforced constraints* (per Zalando Rule 112 and AIP-126). New values are added additively; existing values are never renamed or deleted (per the Azure and Protobuf rules).
3. **`unknown` default fallback** — every consumer (view, query, rollup) has an explicit branch for a status it doesn't recognize, so a future value degrades gracefully into its category rather than disappearing (the Protobuf sentinel pattern).
4. **Tags as a pressure valve, not the primary field** — allow free-form tags for the genuinely unanticipated, and promote a recurring tag into a documented sub-status once consensus emerges — capturing folksonomy's flexibility without surrendering the structure a status field needs.

This keeps the stable part stable and the growing part growing, and it is the shape every major API guide recommends for a field that must stay extensible.

### Sources
- Microsoft/Azure API Guidelines — extensible enums: https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md
- Google AIP-126, Enumerations: https://google.aip.dev/126
- Zalando RESTful API Guidelines (Rules 112, 240): https://opensource.zalando.com/restful-api-guidelines/
- Protocol Buffers best practices: https://protobuf.dev/best-practices/dos-donts/
- GitHub — closing an issue (state + reason + labels): https://docs.github.com/en/issues/tracking-your-work-with-issues/administering-issues/closing-an-issue
- Jira status categories (developer docs): https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-workflow-status-categories/
- Stripe — online migrations (additive dual-write): https://stripe.com/blog/online-migrations
- Folksonomy (tagging tradeoffs): https://en.wikipedia.org/wiki/Folksonomy

---

Sourcing note: the `web_search` tool returned empty for every query, so all findings come from direct fetches of the eight primary sources above. The Jira status-categories fetch failed to return body text, so that example rests on the well-known three-category concept plus the verified GitHub two-tier example; treat the Jira citation as a pointer rather than a fetched quote.
