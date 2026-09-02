# arXiv Translate

An in-browser PDF viewer and translator. Put any PDF URL after the site
address and it renders the document and translates it into the language of
your choice — no upload, no backend, no API key.

**Live:** <https://longpanzhou.github.io/>

```
https://longpanzhou.github.io/arxiv.org/pdf/1706.03762
                              └──────────── any PDF URL ───────────┘
```

## Usage

Prefix a PDF URL with the site address:

| You want | Open |
| --- | --- |
| The "Attention Is All You Need" paper | `longpanzhou.github.io/arxiv.org/pdf/1706.03762` |
| Any arXiv paper | `longpanzhou.github.io/arxiv.org/pdf/<id>` |
| Any other PDF | `longpanzhou.github.io/https://example.com/paper.pdf` |

Then:

1. Pick a target language (or type any Google Translate language code).
2. Click **Translate**.
3. Toggle between **Original** and **Translate** — the latter shows the
   original page beside a translated copy with the text swapped in place.

The address bar at the top is editable; **Copy link** gives you a shareable
URL for the current document.

### Requirements

The PDF host must send permissive CORS headers
(`Access-Control-Allow-Origin`). arXiv does. Hosts that don't will show a
"could not load" message with a direct-open link.

## How it works

Everything runs client-side:

1. **Render** — [`pdfjs-dist`](https://github.com/mozilla/pdf.js) draws every
   page to a canvas. Standard fonts and CJK CMaps are pulled from a CDN so
   glyphs render correctly.
2. **Reassemble** — flat lists of positioned text fragments from pdf.js are
   rebuilt into lines and then paragraph-ish **blocks**, each with a
   bounding box.
3. **Classify** — a content heuristic marks each block as prose or
   *keep-as-is*. Equations, numeric table cells, symbol-heavy fragments and
   equation numbers are left untouched so the original pixels show through.
4. **Translate** — prose blocks are sent to the public Google Translate
   endpoints (`translate.googleapis.com` / `clients5.google.com`), with
   MyMemory as a fallback, chunked and throttled. Results are mapped back to
   their block positions.
5. **Show** — in the "Translate" view each translated block is drawn in place
   over the original page; figures, tables and formulas are never covered.

### Known limits

In-place translation can't perfectly match the original typesetting — text
length changes, fonts are substituted, and line-breaking is greedy. Text
*labels inside a figure*, table text cells, and some display equations are
hard to tell apart from prose and may still be replaced. For a faithful
re-typeset PDF you need a server-side tool such as
[PDFMathTranslate / pdf2zh](https://github.com/Byaidu/PDFMathTranslate).

## Source

The implementation lives in the main site repo,
[`LongpanZhou.github.io`](https://github.com/LongpanZhou/LongpanZhou.github.io),
under `src/pdf/`.
