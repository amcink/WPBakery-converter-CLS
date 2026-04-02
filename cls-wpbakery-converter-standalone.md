---
name: cls-wpbakery-converter
description: Converts a completed CLS blog post markdown file into WPBakery Page Builder shortcode ready to paste into WordPress. Handles Type A (original CLS blog post) and Type B (Insights-adapted piece) content structures. Invoke when a draft is approved and ready for publishing.
---

# cls-wpbakery-converter

Converts a completed CLS blog post markdown file into WPBakery Page Builder shortcode, ready to paste into the WordPress code editor.

---

## When to invoke this skill

- A markdown draft has been approved and is ready for WordPress
- The user says "convert this to WPBakery", "make this WordPress-ready", or similar

Do not invoke on drafts that are still being edited. The input must be a final or near-final markdown file.

---

## Required inputs

1. **Markdown content** — the approved post to convert (paste the content or provide the file path)
2. **Content type** — Type A (CLS blog post) or Type B (Insights-adapted piece). If unclear, ask before proceeding.
3. **For Type B only:** Issue URL and publication month/year for the byline block. If missing, **stop and ask** — do not proceed without it.

**Recommended:** Run an SEO review on the draft before conversion. Heading changes, slug edits, or FAQ rewrites after conversion require regenerating the output.

---

## Step 0 — Classify content type

Before converting anything, classify the post as **Type A** or **Type B**.

**Type A: CLS Blog Post**
- Longer, researched article
- H2 sections are numbered (e.g. "1. Facility and occupancy costs")
- Has a `## Key Takeaways` section
- Has a meaningful TOC (3+ H2/H3 headings in body copy)
- Has a FAQ accordion at the end
- No byline, no CTA block

**Type B: Insights Piece**
- Shorter opinion or thought-leadership article
- Has a byline (e.g. "Submitted by Name, Title")
- May have few or no H2 sections
- Has a CTA block before the FAQ
- Key Takeaways may appear as `### Key takeaways` (H3) — treat identically to H2
- TOC only included if 3+ H2/H3 headings appear in the body copy

**If unsure:** count the body headings (excluding Key Takeaways and FAQ) to decide on TOC inclusion.

If it's a repurposed Insights piece and the issue URL or publication month/year is missing, **stop and ask** before continuing.

---

## Pre-conversion steps

**Step 1 — Read the full source content.**
Do not begin conversion until you have read the complete document.

**Step 2 — Scan for SVGs.**
Before assembling output, scan the document for any `<svg>` blocks. For each one, invoke the `svg-exporter` skill to write a `.svg` file. The exporter returns a placeholder string to use in the shortcode output. Do not embed raw SVG code inline.

Placeholder format returned by svg-exporter:
```
[SVG-FILE: filename.svg — upload to WordPress media library and replace this placeholder with: [vc_single_image source="external_link" css="" custom_src="MEDIA_URL"]]
```

- Drop `<figure>` wrappers — pass only the SVG code to the exporter
- If a `<figcaption>` is present, render it as a `<p>` tag immediately after the placeholder
- If no SVGs are present, skip this step silently

**Step 3 — Strip editorial metadata.**
The markdown may contain a metadata table at the very top (publish date, keyphrase, canonical URL, meta title, meta description). **Strip this entirely** — do not include it in the output.

---

## Canonical shortcode patterns

These patterns must be reproduced **character-for-character**, including CSS class strings and numeric timestamp values. Do not guess, abbreviate, or alter any attribute values.

### 1. Outer wrapper (every post)

```
[vc_row][vc_column]
...all content here...
[/vc_column][/vc_row]
```

### 2. Opening separator

Always the very first element inside the outer wrapper:

```
[vc_separator css=".vc_custom_1773174855904{margin-top: 35px !important;}"]
```

### 3. TOC + Key Takeaways block

**Include only if the body copy has 3 or more H2/H3 headings** (excluding Key Takeaways and FAQ). If fewer than 3, skip this entire block.

This is a two-column `vc_row_inner` — left column (1/3) = Table of Contents, right column (2/3) = Key Takeaways. Each column contains a single `vc_column_text` block with a custom div wrapper.

```
[vc_row_inner][vc_column_inner width="1/3"][vc_column_text css=".vc_custom_1774892922005{background-color: #F0F0F0 !important;border-color: #76BC2100 !important;}"]
<div class="cls-toc">
<div class="cls-toc-header"><span class="cls-toc-header-text">Table of Contents</span></div>
<ul>
 	<li><a href="#section-1-slug">Section 1 Name</a></li>
 	<li><a href="#section-2-slug">Section 2 Name</a></li>
 	<li><a href="#section-3-slug">Section 3 Name</a></li>
</ul>
</div>
[/vc_column_text][/vc_column_inner][vc_column_inner width="2/3"][vc_column_text]
<div class="cls-takeaways">
<div class="cls-takeaways-header"><span class="cls-takeaways-header-text">Key Takeaways</span></div>
<ul>
 	<li>Takeaway one.</li>
 	<li>Takeaway two.</li>
 	<li>Takeaway three.</li>
</ul>
</div>
[/vc_column_text][/vc_column_inner][/vc_row_inner]
```

There is no separator row after the TOC/Takeaways block — body content continues directly into the next element.

**TOC content rules:**
- "Table of Contents" label is a `<span class="cls-toc-header-text">` inside `<div class="cls-toc-header">` — not an `<h3>` tag
- TOC entries are `<li><a href="#anchor-slug">Section name</a></li>` items inside the `<ul>` in the `cls-toc` div
- Anchor slugs: lowercase, spaces → hyphens (e.g. "Why terminology matters" → `#why-terminology-matters`)
- Numbered H2 sections: strip the number, link just the name (e.g. "1. Facility costs" → link text is "Facility costs")
- List all body headings in document order — no grouping or blank lines between entries
- Do **not** include the FAQ heading in the TOC
- For glossary posts, letter headings use the letter as anchor: `<li><a href="#a">A</a></li>`

**Key Takeaways rules:**
- "Key Takeaways" label is a `<span class="cls-takeaways-header-text">` inside `<div class="cls-takeaways-header">` — not an `<h3>` tag
- Takeaway items are `<li>` elements inside the `<ul>` in the `cls-takeaways` div — never plain text lines
- Source may be `## Key Takeaways` or `### Key takeaways` — treat identically

### 4. Byline block (Type B only)

Appears immediately after the TOC block (or after the opening separator if no TOC):

```
[vc_column_text css=""]

<em>Submitted by AUTHOR_NAME, TITLE.<br>
Originally published in <a href="ISSUE_URL">Life Sciences Insights Magazine, MONTH YEAR</a><br>
</em>

[/vc_column_text]
```

### 5. Body text blocks

Each distinct section of body content gets its own `vc_column_text` block.

```
[vc_column_text css=""]

CONTENT

[/vc_column_text]
```

**Section boundary rules:**
- One block per major section: intro, each H2 section, each standalone named section
- Do **not** wrap individual paragraphs in separate blocks — all paragraphs in a section share one block
- Preserve paragraph breaks as blank lines within the block

**Heading treatment inside blocks:**
- H2 section titles that appear in the TOC: render as `<a name="slug"></a><h3>Section Name</h3>`
- Named subheadings (non-numbered) that appear in the TOC: same treatment — `<a name="slug"></a><h3>Subheading</h3>`
- Glossary letter headings: `<a name="a"></a><h3>A</h3>`
- H2/H3/H4 headings that do **not** appear in the TOC: render as plain text, not heading tags
- Anchor slugs must exactly match the `href` values used in the TOC links

**Inline formatting rules — all formatting must be HTML, never markdown:**
- `**bold**` → `<strong>bold</strong>` — never output raw `**`
- `*italic*` → `<em>italic</em>` — never output raw `*`
- `[link text](url)` → `<a href="url">link text</a>` — never output raw markdown link syntax
- Bold text at the start of a paragraph (used as a pseudo-heading) → `<strong>` HTML

**List rules:**
- Lists are plain text, one item per line, blank line before and after — no bullet characters (-, *, •), no `<ul>` or `<li>` tags
- Exception: Key Takeaways always use `<ul><li>` — see section 3 above

### 6. Images (external URL)

Images break out of the surrounding `vc_column_text` block. Close the current text block, insert an inner row with the image, then reopen a text block:

```
[/vc_column_text][/vc_column_inner][/vc_row_inner][vc_row_inner][vc_column_inner][vc_separator css=""][vc_single_image source="external_link" css="" custom_src="IMAGE_URL"][/vc_column_inner][/vc_row_inner][vc_column_text css=""]
```

- Extract the URL from `![alt text](url)` — use it as `custom_src`
- Discard the alt text — do not include it
- If an image appears before any text in a section, open an image row before opening the text block

### 7. SVG / figure blocks

See Pre-conversion Step 2. Always use the svg-exporter placeholder — never embed raw SVG inline.

### 8. Blockquotes

Each blockquote gets its own `vc_column_text` block:

```
[vc_column_text css=""]

<blockquote>
<p>BLOCKQUOTE_TEXT</p>
</blockquote>

[/vc_column_text]
```

- Do not merge blockquotes with adjacent paragraphs
- Bold text inside a blockquote → `<strong>` inside the `<p>` tag
- Strip the markdown `>` prefix

### 9. Alphabet / jump navigation (glossary posts)

Render as a single line of linked letters in its own `vc_column_text` block:

```
[vc_column_text css=""]

<a href="#a">A</a> · <a href="#b">B</a> · <a href="#c">C</a> · ...

[/vc_column_text]
```

Anchor targets on letter headings in the body must exactly match these `href` values.

### 10. CTA block (Type B only)

Appears just before the closing separator and FAQ accordion. Always use this exact wording and these exact URLs regardless of how the CTA appears in the source markdown:

```
[vc_column_text css=""]

<em>Get the latest thought leadership from California's life sciences sector in the quarterly <a href="https://www.califesciences.org/life-sciences-insights/">Life Sciences Insights magazine</a>. Share your own news and insights with <a href="https://www.califesciences.org/member-voice/">CLS Member Voice</a>.</em>

[/vc_column_text]
```

### 11. Closing separator (before FAQ)

```
[vc_separator css=""]
```

### 12. FAQ accordion

```
[vc_tta_accordion title_tag="h3" section_title_tag="h6" color="white" c_icon="chevron" active_section="0" no_fill="true" collapsible_all="true" title="FAQ: SECTION_TITLE"][vc_tta_section title="QUESTION_1" tab_id="TAB_ID_1"][vc_column_text css=""]

ANSWER_1

[/vc_column_text][/vc_tta_section][vc_tta_section title="QUESTION_2" tab_id="TAB_ID_2"][vc_column_text css=""]

ANSWER_2

[/vc_column_text][/vc_tta_section][/vc_tta_accordion]
```

- `title=""` on `vc_tta_accordion` = the FAQ heading as written in the markdown
- Question text in `title=""` attributes: reproduce exactly, preserving capitalisation
- Answer text: plain paragraphs only — no markdown formatting, no bullet characters
- Tab IDs format: `TIMESTAMP-XXXXXXXX-XXXX` where TIMESTAMP is a 13-digit Unix ms timestamp and the remainder are hex segments — generate a distinct value for every section

---

## Assembly order

### Type A (CLS Blog Post)

```
[vc_row][vc_column]
[opening separator]
[TOC/Takeaways block]        ← only if 3+ body headings
[body section blocks]        ← one per H2/section
[closing separator]
[FAQ accordion]
[/vc_column][/vc_row]
```

### Type B (Insights Piece)

```
[vc_row][vc_column]
[opening separator]
[TOC/Takeaways block]        ← only if 3+ body headings
[byline block]
[body section blocks]        ← one per section
[CTA block]
[closing separator]
[FAQ accordion]
[/vc_column][/vc_row]
```

---

## Output format

Deliver the complete shortcode as a **single unbroken block of text**. Do not add line breaks between shortcode tags that are meant to be inline. The output should be ready to paste directly into the WordPress code editor (Text tab or Code Editor view in WPBakery).

Do not wrap the output in a markdown code block — deliver as plain text so the user can copy and paste without stripping backticks.

If the post contains SVG placeholder strings, list them after the shortcode under:

```
## SVG files to upload

- [SVG-FILE: filename.svg — upload to WordPress media library and replace placeholder with vc_single_image shortcode]
```

---

## Quality checks before delivering output

- [ ] No raw `**bold**`, `*italic*`, or `[link](url)` markdown remains — all HTML
- [ ] No bullet characters (-, *, •) in body lists — plain text, one item per line
- [ ] Key Takeaways rendered as `<ul><li>` inside `cls-takeaways` div — never plain text
- [ ] TOC anchor slugs exactly match the `<a name="...">` values in body headings
- [ ] CSS class strings reproduced character-for-character
- [ ] Tab IDs on FAQ sections are distinct and follow `TIMESTAMP-XXXXXXXX-XXXX` format
- [ ] Editorial metadata table absent from output
- [ ] Type B: byline block and CTA block present; CTA uses canonical wording above
- [ ] Type A: no byline block, no CTA block
- [ ] No raw SVG embedded inline

---

## Common mistakes — never do these

| Mistake | Rule |
|---|---|
| Embedding raw SVG inline | Always use svg-exporter and placeholder |
| Outputting `**bold**` or `*italic*` | Convert to `<strong>` / `<em>` HTML |
| Outputting `[text](url)` | Convert to `<a href="url">text</a>` |
| Bullet characters in body lists | Plain text, one item per line |
| Plain text Key Takeaways | Must be `<ul><li>` inside `cls-takeaways` div |
| One `vc_column_text` per paragraph | One block per section, not per paragraph |
| Modified CSS class strings | Reproduce exactly, including numeric timestamps |
| Including the metadata table in output | Strip before conversion |
| Extra `[vc_row]` wrappers | Entire post is one `[vc_row][vc_column]` |
| FAQ heading in the TOC | TOC covers body headings only |
| `<h2>` tags in body text | Use `<h3>` for TOC-linked headings |
| `<h3>` for TOC/Takeaways labels | Use `<span class="cls-toc-header-text">` / `<span class="cls-takeaways-header-text">` inside their header divs |
| Bare `<a>` links in the TOC | TOC entries must be `<li><a href="...">...</a></li>` inside the `cls-toc` `<ul>` |

---

## Known edge cases

- **SVGs absent from a post:** skip svg-exporter entirely — do not note or flag their absence
- **SVG Visual mode stripping:** SVGs pasted inline into WPBakery are destroyed if the Visual editor tab is ever toggled. Always use the placeholder/file approach via svg-exporter.
- **Key Takeaways must be bulleted:** use `<ul><li>` HTML inside the `cls-takeaways` div — plain text lines will not render as bullets in WordPress
- **Named subheadings need H3 + anchors:** any heading that appears in the TOC must get `<a name="slug"></a><h3>` treatment so TOC links resolve
- **CTA block wording is canonical:** always use the exact wording and URLs defined in section 10, regardless of how the source markdown words it
- **All formatting is HTML:** WPBakery's code editor renders HTML, not markdown — always convert before output
- **Insights piece attribution:** if the issue URL or publication date is missing from the markdown, stop and ask the user before continuing

---

## Handoff after conversion

- Flag any SVG files that need to be uploaded to the WordPress media library
- Note if a Type B byline was missing an issue URL (flag for the user to update in WordPress)
- Remind the user to paste into the **code editor** (not the visual editor) in WPBakery — opening the visual editor tab after pasting will strip SVG placeholder content
