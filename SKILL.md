---
name: surname-research
description: Full surname/family-name research pipeline — deep web research (etymology, origin, notable bearers), JewishGen archival record search via Chrome, cited HTML dossier artifact with family trees, and a raw records file. Use when the user asks to research a surname, family name origin, or build family history. Works best for Ashkenazi/Eastern-European surnames; web-research phase works for any surname.
---

# Surname Research Pipeline

A proven end-to-end workflow. Output: (1) a designed trilingual HTML dossier (published as an Artifact and/or PDF), (2) `<surname>-records.md` raw archival extracts, (3) family-tree sections reconstructed from records, (4) optionally the user's own documented direct line.

## Phase 0 — Preflight (ALWAYS run first, and show the user the result)

Before any research, probe your environment and REPORT it. Probe mechanism: (1) subagent tool (Agent/Task) — is it in your tool list? (2) web search — is a WebSearch-type tool present? (3) browser automation — try loading `tabs_context_mcp` via tool search; an error or no match = absent; (4) a "deep-research" style workflow — count it present only if your harness explicitly lists one; NEVER assume it. Then print this table to the user:

| Capability | Present? | Enables |
|---|---|---|
| Subagents (Agent tool) | yes/no | parallel research fan-out (Phase 1) |
| Web search | yes/no | all research |
| Browser extension (Claude-in-Chrome) | yes/no | Phases 3, 3b (archive record search) |
| deep-research workflow | yes/no | optional Phase 1 accelerator |

**Rule: a phase whose requirement is missing is SKIPPED ALOUD** — one sentence telling the user what was skipped, why, and what would unlock it (e.g. "no browser extension → I can't drive JewishGen; install Claude-in-Chrome to unlock archival records"). Silent degradation is a bug. If web search itself is absent, stop and say so — there is nothing honest to research from.

## Phase 1 — Deep web research

1. **Default path — build the fan-out yourself with subagents** (use a named deep-research workflow only if Phase 0 found one; then read its FULL result from the task output file, since notifications truncate). Spawn 4–6 parallel general-purpose subagents, one per angle:
   - etymology / onomastics (dictionaries, suffix guides)
   - geographic origin + modern distribution
   - notable bearers (encyclopedias, native-language Wikipedia)
   - genealogical record indexes (JewishGen, FamilySearch, national archives)
   - native-script sources (Cyrillic/Hebrew/etc. spellings of the name)
   Prompt template per agent: "Research the surname <X> from the angle of <angle>. Return 4+ findings as claim + source-URL pairs. Prefer primary/authoritative sources. Note confidence and conflicts."
2. Synthesize the returned findings yourself; then run a verification pass — one subagent per surprising or load-bearing claim, prompted to REFUTE it; drop claims that fail. **Floor: ≥10 distinct source URLs across the report, or tell the user explicitly that coverage was thin.** Last resort with no subagent tool: run ≥6 sequential web searches yourself across the same angles before writing anything.
3. Useful source patterns: Geneanet surname pages (DAFN2 = Dictionary of American Family Names, Oxford 2022 — pages 403 on direct fetch, use search-indexed mirrors), JewishGen KehilaLinks suffix guides, Russian Wikipedia for notable bearers, j-roots.info forum (Russian Jewish genealogy), Yiddish etymology blogs.
4. Etymology heuristic for -ovich/-owicz names: check sibling surnames (root + -er, -itz) in DAFN2; metronymics from Yiddish female names are common. Label as inference when no direct dictionary entry exists.

## Phase 2 — HTML dossier

*(If Phase 0 marked a needed capability unavailable: announce the skip and continue with what works — e.g. no Artifact hosting → deliver the HTML file directly.)*

1. Design: archival-dossier treatment — subject-derived palette (e.g. parchment + old gold), serif stack (Palatino), small-caps sans labels, confidence chips (high/medium), double-rule masthead. Both light/dark themes token-driven.
2. Sections: verdict-at-a-glance → etymology (derivation diagram: name-part + suffix = meaning) → geographic origin → archival records → family trees → notable bearers timeline → variants table → how-to-research-further (ranked) → honest caveats → key sources. Footer: stats.
3. Keep the canonical file in the project directory (not a temp dir) so future sessions update it.
4. Optional: trilingual (EN/HE/RU) — wrap each language in `<div class="lang" id="lang-XX">` (full page copy), buttons + small `setLang()` JS toggling `hidden`; Hebrew block gets `dir="rtl"` + scoped RTL CSS (blockquote border side, table alignment, list padding); keep tree containers `dir="ltr"` so connectors don't flip.
5. Optional hero image via a text-to-image API (e.g. DashScope/Qwen `wan2.2-t2i-flash`: POST image-synthesis with `X-DashScope-Async: enable`, poll `/api/v1/tasks/{id}`) — downscale to ~1100px JPEG q78 and embed as data URI (artifact CSP blocks remote images).
6. PDF export: build a single-language standalone HTML (strip other lang divs + langbar), then `chrome --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf=out.pdf file:///...`.

## Phase 3 — JewishGen record search (Chrome)

Requires the user's Chrome + a JewishGen account. **If Phase 0 marked the browser extension absent: announce the skip, give the user the manual search URL (`https://www.jewishgen.org/databases/all/`) and what to type, and continue to Phase 4 with web-sourced data only.** Flow and pitfalls:

1. `tabs_context_mcp` → new tab → `https://www.jewishgen.org/databases/all/` (Unified Search; `/databases/unified/` is a 404).
2. Fill surname, keep "Phonetically Like" (catches spelling variants + Cyrillic transliterations), Search. Results = `jgform.php`, per-database rows with "List N records" buttons.
3. **Login gate**: record lists need login. Never enter passwords yourself — ask the user. New accounts: email verification + profile completion (real name/address — ask the user for values), then a human review (~24h) gates ONLY Family Finder (JGFF) + Family Tree of the Jewish People; regular databases work immediately after profile submit.
4. **Popup behavior**: "List N records" buttons open results in NEW tabs on real mouse clicks; JS `.click()` gets popup-blocked. Reliable path: same-tab submit — pick the TIGHTEST matching `<tr>` (sort candidates by textContent length; a loose match grabs a giant ancestor row and fires the wrong form), then `f.target='_self'; HTMLFormElement.prototype.submit.call(f)` (a form input named "submit" shadows the method).
5. Extract with `get_page_text`. Pagination: "Page N: Records X to Y" buttons (same submit trick). The search form resets after navigation — refill via JS and verify the field value landed before submitting.
6. Priority databases for Pale-of-Settlement names: Belarus Census & Revision lists, Births/Deaths/Marriages, Duma Voter Lists (1906–12), Vsia Rossiia business directories (occupations!), Lithuania sets (LitvakSIG — incl. **emigration/passport records via Libau**), **Ukraine Births/Deaths groups (Kiev, Kharkov, Odessa — where Pale families moved after 1860s)**, Holocaust sets (Extraordinary Commission, USC Shoah), JOWBR burials, Arolsen Red Line tracing cards.
7. **Phonetic noise**: cull unrelated variants (check patronymic style — Slavic Christian patronymics on voter lists = different, non-Jewish family).
8. Save extracts to `<surname>-records.md`: per-database sections, archive references (fond/file numbers, FHL microfilms), a "big picture" synthesis of family nests, follow-ups list.

## Phase 3b — Soviet WW2 military records (pamyat-naroda.ru)

For ancestors born ~1895–1925 in the USSR, military records are often the ONLY online source bridging the
post-1911 civil-register gap — one soldier's card can name his father (patronymic!) and confirm family stories.
**If Phase 0 marked the browser extension absent: announce the skip and hand the user the prefilled search URL
(pattern below) to open themselves — the site is free and public.**

1. **URL-parameter search** (no form needed): `https://pamyat-naroda.ru/heroes/?last_name=Х&first_name=Y&middle_name=Z&group=all&types=<full-type-list>&page=1&grouppersons=1` — Cyrillic only. Needs browser host permission for the site. After several requests the site throws a **symbol CAPTCHA — the user must solve it, never the agent** (hard rule); solving it once anywhere in that Chrome clears the session.
2. **Person pages** (`person-hero{id}`): the JS globals `documentIds` and `docInfo` expose every document ID and — critically — `hero_last_name` with **all spelling variants the archive grouped** (e.g. Зильберман/Зилберман/Сильберман). Search those variants everywhere else too.
3. **Date tolerance**: recorded birth dates are often weeks off (clerk errors). Match on year + city + patronymic, not exact date. Family memory sometimes beats the presumed date — check BOTH versions the family offers.
4. **The patronymic is the payload**: "Борисович" = father named Boris → extends the male line a generation instantly. Combine with the Ashkenazi naming custom (children named after DECEASED relatives): a man naming his son after his own father means the father had died — corroborates chains and dates.
5. **Document types and what they give**:
   - `nagrady_nagrad_doc` (wartime award docs, esp. 1944–45) — **SCANNED наградной лист with a personal feat description**; richest narrative source, viewable online.
   - `chelovek_yubileinaya_kartoteka` — 1985 jubilee Order of the Patriotic War = proof the veteran was alive/registered in 1985.
   - УПК (officer personnel cards) — full biography, parents, wife, education, photo; physical at ЦА МО Podolsk (archive request).
   - `card_vmf` (navy cards) — physical at ЦВМА Gatchina, exact cabinet/box cited; usually photo + next of kin.
   - `kld_ran` (wound cards) — physical at military-medical archive; the cited drawer range reveals the spelling the card is filed under.
   - Unit rosters (`именные списки частей`) can record an **evacuation-town registration** — documentary confirmation of family evacuation stories (e.g. Perm oblast).
6. **Дорога памяти** photo gallery (same search types): photos are uploaded BY relatives — an existing upload = a living-relative lead; absence = nobody has claimed the memory yet.
7. Holocaust cross-check: babynyar.org has a searchable API (`/api/v3/names/`); treat empty results as absence-from-index, not proof of survival — but a clean negative on a whole family is still meaningful.

## Phase 4 — Family trees & chain-tracing

1. Reconstruct sub-trees per geographic nest — do NOT merge unless records connect them.
2. Patronymics give parent names ("X, son of Y" → father Y; mark reconstructed roots "inferred from patronymic").
3. Same rare patronymic + same region + fitting dates across record sets = hypothesis sibling link — draw DASHED, label the reason.
4. **Chain-tracing across databases** (the technique that finds direct lines):
   - A birth record's (name, birth year, father) can match a death record in ANOTHER city decades later — Pale families migrated to Kiev/Kharkov/Odessa after the 1860s. Match on name + patronymic + computed birth year (±2).
   - Wives' names recur: a revision-list wife appearing as "mother" in a later birth record in another city confirms the same family.
   - Grandchildren named after ancestors (Ashkenazi naming: after deceased relatives) corroborate chains.
   - Civil registers on JewishGen typically END ~1911–1917; a person born in the 1910s–20s sits in a documented family but needs Soviet ZAGS records or family documents (patronymic!) to prove the final link. State the gap honestly.
5. Pure HTML/CSS tree (CSP blocks libs): nested `<ul class="tree">`, flex, connector pseudo-elements (`li::before/::after` half-width top borders, `::after` border-left drop, `ul::before` parent drop, `li:last-child::before` corner). `.hyp` = dashed. Wrap each tree in `overflow-x:auto`. Node = name + dates/place + source tag.
6. Revision-list ages = age at revision — write "b. ~YYYY".

## Phase 5 — Fold-in + follow-ups

1. Update dossier with records + trees; republish (same artifact URL).
2. Standing follow-ups: JGFF/FTJP after review; Yad Vashem central DB (JS-only site — needs browser host permission or manual, prefilled URL `https://yvng.yadvashem.org/index.html?language=en&s_lastName=<NAME>`); Pamyat-Naroda/OBD-Memorial for Soviet WW2 records (same permission caveat); Beider's Dictionary of Jewish Surnames from the Russian Empire (offline); ordering archive scans by fond/file; US immigration (Ellis Island/Ancestry) for emigration-record families; JOWBR full-record pages (headstone photos, sometimes father's Hebrew name).
3. Ask the user for family testimony (birthplaces, migration, sibling stories) — one fact (a patronymic, a town) can anchor them to a documented nest.
4. Update this skill with newly discovered pitfalls.

## Environment pitfalls (hard-won)

- Chrome-extension automation is per-site permissioned: new hosts (yadvashem.org, pamyat-naroda.ru, even claude.ai) may be denied — fall back to structural validation / manual links and say so.
- `fetch()` results in the page context can be blocked by the extension's data filter — same-tab form submit + `get_page_text` is the reliable extraction path.
- Screenshot API can fail intermittently; `get_page_text` + targeted JS extraction is more robust than pixel work for tables.
- Headless-Chrome PDF can fail silently — verify the output file's LastWriteTime, retry with `--no-sandbox`.
- JOWBR full-record URL pattern: `jowbr.php?rec=<DB>_<NNNNNNN>`; results tables are nested — match the tightest row before clicking links, or hand the user the one-click step.
