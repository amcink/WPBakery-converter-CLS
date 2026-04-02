# cls-wpbakery-converter skill

## What this skill does

Converts a completed CLS blog post markdown file into WPBakery Page Builder shortcode, ready to paste into the WordPress code editor. Handles both Type A (original CLS blog post) and Type B (Insights-adapted piece) content structures.

---

## When to invoke this skill

Invoke this skill when:

- A markdown draft in `working/blog-drafts/` or `working/insights-adapted/` has been approved and is ready for WordPress
- The user says "convert this to WPBakery", "make this WordPress-ready", or similar
- A blog post draft has passed SEO review and is entering the publishing pipeline

Do not invoke this skill on drafts that are still being edited. The input must be a final or near-final markdown file — conversion is not a drafting step.

---

## Required inputs

1. **Markdown file path** — the approved post to convert (e.g. `working/blog-drafts/0425 - CLS - Blog - Life-Sciences-Grants.md`)
2. **Content type** — Type A (CLS blog post) or Type B (Insights-adapted piece). If unclear, ask before proceeding.
3. **For Type B only:** Issue URL and publication month/year for the byline block. If missing from the markdown, **stop and ask** — do not proceed without it.
4. **SEO check confirmation** — `blog-seo-check` must have been run on this draft before conversion begins. If the user has not confirmed this, stop and ask: *"Has blog-seo-check been run on this draft? WPBakery conversion should happen after SEO review, not before — heading changes or slug edits will require regenerating the output."* Proceed only with explicit confirmation.

---

## Pre-conversion steps — required

**Step 1 — Load the conversion ruleset.**
Read `_templates/wpbakery-rules.md` in full before beginning. This file is the authoritative source for all shortcode patterns, CSS class strings, assembly order, and edge case handling. Do not rely on memory — read it fresh every time.

**Step 2 — Read the source markdown.**
Read the full markdown file provided. Do not begin conversion until you have read the complete document.

**Step 3 — Classify content type.**
Confirm whether the post is Type A or Type B per the criteria in `_templates/wpbakery-rules.md` Step 0. If the type is ambiguous, ask the user before proceeding.

**Step 4 — Scan for SVGs.**
Before assembling output, scan the document for any `<svg>` blocks. If found, invoke the `svg-exporter` skill to write each to a `.svg` file and use its returned placeholder string in the output. If no SVGs are present, skip this step silently — do not note or flag their absence.

**Step 5 — Strip editorial metadata.**
Identify and discard the metadata table at the top of the file (publish date, keyphrase, canonical URL, meta title, meta description). Do not include it in the shortcode output.

---

## Conversion rules

Follow `_templates/wpbakery-rules.md` exactly for all of the following. Do not improvise, abbreviate, or alter any attribute value, CSS class string, or numeric timestamp:

- Outer wrapper
- Opening separator
- TOC + Key Takeaways block (include only if 3+ body H2/H3 headings)
- Byline block (Type B only)
- Body text blocks and section boundary rules
- Heading treatment inside blocks
- Inline formatting (all must be HTML — no raw markdown bold, italic, or links in output)
- List rendering rules
- Image blocks (external URL pattern)
- SVG / figure blocks
- Blockquotes
- Alphabet / jump navigation (glossary posts)
- CTA block (Type B only — canonical wording and URLs as defined in wpbakery-rules.md)
- Closing separator
- FAQ accordion

Assembly order is type-dependent — follow the Type A or Type B sequence defined in `_templates/wpbakery-rules.md`.

---

## Output format

Deliver the complete shortcode as a **single unbroken block of text** with no line breaks between shortcode tags that are meant to be inline. The output should be ready to paste directly into the WordPress code editor (Text tab or Code Editor view in WPBakery).

Do not wrap the output in a markdown code block. Deliver it as plain text so the user can copy and paste without stripping backticks.

Structure the response as:

```
[converted shortcode — single unbroken block]
```

If the post contains SVG placeholder strings, list them after the shortcode block under a heading:

```
## SVG files to upload

- [SVG-FILE: filename.svg — upload to WordPress media library and replace placeholder with: vc_single_image shortcode]
```

---

## File naming and save location

Save the shortcode output to:

```
working/wpbakery-output/MMYY - CLS - WPB - [Title-Slug].txt
```

Example: `working/wpbakery-output/0425 - CLS - WPB - Life-Sciences-Grants.txt`

Use `.txt` extension — this prevents editors from interpreting the shortcode syntax.

---

## Quality checks before delivering output

Before returning the converted shortcode, verify:

- [ ] No raw `**bold**`, `*italic*`, or `[link](url)` markdown remains in the output — all must be HTML
- [ ] No bullet characters (-, *, •) in body lists — plain text, one item per line
- [ ] Key Takeaways rendered as `<ul><li>...</li></ul>` — never plain text lines
- [ ] TOC anchor slugs exactly match the `<a name="...">` values in body headings
- [ ] CSS class strings reproduced character-for-character from `wpbakery-rules.md` — not paraphrased
- [ ] Tab IDs on FAQ sections are distinct and follow the `TIMESTAMP-XXXXXXXX-XXXX` format
- [ ] Editorial metadata table is absent from the output
- [ ] For Type B: byline block and CTA block are present; CTA uses canonical wording from wpbakery-rules.md
- [ ] For Type A: no byline block, no CTA block
- [ ] SVG code is not embedded inline — placeholder strings used instead

---

## Common mistakes — never do these

These are reproduced from `_templates/wpbakery-rules.md` as a quick-reference reminder:

| Mistake | Rule |
|---|---|
| Embedding raw SVG inline | Always use svg-exporter and placeholder |
| Outputting `**bold**` or `*italic*` | Convert to `<strong>` / `<em>` HTML |
| Outputting `[text](url)` | Convert to `<a href="url">text</a>` |
| Bullet characters in body lists | Plain text, one item per line |
| Plain text Key Takeaways | Must be `<ul><li>...</li></ul>` |
| One `vc_column_text` per paragraph | One block per section, not per paragraph |
| Modified CSS class strings | Reproduce exactly, including numeric timestamps |
| Including the metadata table in output | Strip before conversion |
| Extra `[vc_row]` wrappers | Entire post is one `[vc_row][vc_column]` |
| FAQ heading in the TOC | TOC covers body headings only |
| `<h2>` tags in body text | Use `<h3>` for TOC-linked headings |

---

## Handoff after conversion

After delivering the shortcode:

- Confirm the output file has been saved to `working/wpbakery-output/`
- Flag any SVG files that need to be uploaded to the WordPress media library
- Note if the Type B byline was missing an issue URL (flag for user to update in WordPress)
- Remind the user to paste into the **code editor** (not the visual editor) in WPBakery — opening the visual editor tab after pasting will strip SVG code

---

## What this skill does NOT do

- Edit or rewrite the source markdown
- Validate SEO elements (run `blog-seo-check` before conversion)
- Upload content to WordPress
- Generate images or media
- Handle non-CLS content or non-WPBakery WordPress builders
