# Deep Research

Research topic: $ARGUMENTS

## When to use (vs web search)

ONLY use this capability when the user explicitly requests deep/exhaustive research. Deep research takes 5–20 minutes and is significantly more expensive than web search. For normal "research X" requests, quick lookups, or fact-checking, use **web search** instead.

## Command

Choose a descriptive filename based on the topic (e.g., `mrna-vaccine-platforms-2026`, `gut-microbiome-depression`). Use lowercase with hyphens, no spaces.

```bash
python scripts/google_research.py research "$ARGUMENTS" -o "$FILENAME.md" --timeout 1800
```

Important:
- Use `--timeout 1800` (30 minutes) — academic queries regularly take 15+ minutes.
- The command is **synchronous**: it handles polling internally. Do not run a separate poll step unless the command times out (see below).
- The script prints `interaction_id` and progress to stderr. Note the `interaction_id` — you will need it if the command times out.
- For scientific or technical queries, prepend context to steer toward scholarly sources. For example, instead of `"effects of sleep deprivation"`, use `"peer-reviewed research and clinical studies on the effects of sleep deprivation"`.

## If the command times out

If the command exits with a timeout error, it will print the `interaction_id` and a recovery command. Wait a few minutes, then poll:

```bash
python scripts/google_research.py poll "$INTERACTION_ID" -o "$FILENAME.md"
```

Tell the user the research is still running server-side, and re-run `poll` again if it hasn't completed yet.

## What the output looks like

The script writes a comprehensive markdown report to `$FILENAME.md` and prints it to stdout. The report contains:
- Full narrative with inline `[[N]](url)` citations
- A `## Sources` section (often 50–100+ entries for deep queries)

There is no separate `.json` metadata file or executive summary — the markdown is the complete deliverable.

**Expect mixed sources.** Google deep research often includes peer-reviewed journals *and* commercial supplement sites, health blogs, news outlets, or product pages — especially in practical/clinical sections. Do not assume every source is academic.

## Response format

**After the command completes:**

1. **Brief summary** — relay the report's opening summary section (usually `## Summary` or the first paragraph). Do **not** paste the full report into chat.

2. **Source Quality** — required section. Read only the `## Sources` section (and the summary if needed for context). Do **not** read the entire report into context.

   Use this template:

   ```markdown
   ### Source Quality

   - **Total sources cited:** ~N
   - **Peer-reviewed / preprint / clinical-registry** (PubMed, PMC, Nature, MDPI, Frontiers, arXiv, ClinicalTrials.gov, university domains): ~X (~Y%)
   - **Institutional / government** (NIH, WHO, .gov, .edu pages that are not journal articles): ~X
   - **News / industry / commercial / blog** (supplement retailers, product pages, health blogs, Wikipedia, YouTube): ~X (~Y%)

   **Assessment:** [1–2 sentences — e.g. "Core mechanistic and clinical claims are grounded in peer-reviewed literature; non-academic sources appear mainly in commercial/product sections." OR "Academic coverage is thin — consider a follow-up targeted search."]

   **Evidence note:** [If &lt;50% academic/institutional, flag explicitly. Note which key claims rely on academic vs non-academic sources.]
   ```

   Classification rules:
   - URLs are often grounding redirects — judge from **source titles and domain names** in the `## Sources` list (e.g. `nih.gov`, `mdpi.com`, `justthrivehealth.com`).
   - Do not read all 100+ sources line-by-line if the list is huge — sample systematically (first 20, middle 20, last 20) or grep for domain patterns, then extrapolate with a clear caveat.
   - Do not invent counts — if you cannot estimate, say "approximately" and explain your sampling method.

3. **Report file path** — tell the user: `$FILENAME.md`

4. **Offer to read more** — ask if the user wants to review the full report. **Do NOT read the file into context unless the user asks** — reports are often 5,000–10,000+ words.

5. **`interaction_id`** — note from stderr (`[research] interaction_id=...`) for potential follow-up or poll recovery.
