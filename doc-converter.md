---
name: doc-converter
---

# Document converter (PDF / DOCX / Markdown)

Convert a document between PDF, Word and Markdown so that nothing important goes missing. "Important" means: all of the text in every language, the heading structure, every table (as a real table), every figure/image (with its caption), every equation (readable, and editable where the target allows), and small state such as ticked checkboxes. Fonts, colours, exact page layout and running headers/footers are not important and may change.

The reason this skill exists is that every converter fails silently somewhere - equations vanish, a table becomes loose numbers, a figure is skipped, Chinese text disappears from a PDF, a ticked box loses its tick - and the output still looks plausible. So the workflow is always: **convert with the right route, then verify, then repair**. Never hand back a conversion that has not been checked.

## Workflow

1. **Look at the source.** Identify the format, length, and whether it has tables, figures, equations, non-English text, form fields, or is scanned. Render it and look at a few pages (see "Looking at pages") - the look of the page is the ground truth the output is compared against, because the tools that read the source can themselves miss things.
2. **Convert** using the route for that direction below.
3. **Verify** with `losscheck.py` plus the visual checks under "Verify".
4. **Repair** anything lost, and re-check.
5. **Deliver** the file with its media folder if it has one. Report quietly: say nothing about the checks when everything survived; if something could not be preserved or had to change, say exactly what and where in one or two lines (e.g. "Equation 3 on p.4 is kept as an image, not editable text").

Keep the output beside the source with the same base name (`paper.pdf` -> `paper.md` + `paper_media/`). Never overwrite the source. Convert, do not edit: keep the author's wording, punctuation and order even when it could be improved, unless asked.

## Tools

Check what is present (`which pandoc soffice pdftoppm pdffonts tesseract`; `python3 -c "import pymupdf4llm"`) and install what is missing: `pip install --break-system-packages pymupdf pymupdf4llm pdf2docx`. Find Chromium with `CHROME=$(ls /opt/pw-browsers/chromium-*/chrome-linux/chrome 2>/dev/null | head -1 || which chromium chromium-browser google-chrome)`.

Two environment facts that cause silent loss, so test rather than assume:
- **LibreOffice without its Math module drops every Word equation** when exporting to PDF (the line simply has a gap). Check with `ls /usr/lib/libreoffice/program/libsmlo.so`.
- **Pandoc's LaTeX PDF engine is often incomplete** (missing packages, no CJK fonts). Do not rely on it; use the HTML -> Chromium route below.

## Routes

### Word -> Markdown

Pandoc does the conversion, but it silently ignores several things Word files contain. Run the pre-check first; it reports them and writes a copy in which form checkboxes become plain `☐`/`☒` text so their state survives.

Save as `docx_prep.py`:

```python
#!/usr/bin/env python3
"""docx_prep.py IN.docx OUT.docx - report what pandoc cannot read in a Word file,
and write a copy in which form checkboxes are turned into plain ☐ / ☒ text."""
import sys, re, zipfile
src, dst = sys.argv[1], sys.argv[2]
zin = zipfile.ZipFile(src); names = zin.namelist()
doc = zin.read("word/document.xml").decode("utf8")
count = [0, 0]
def box(m):
    run = m.group(0)
    if "<w:checkBox>" not in run: return run
    chk = re.search(r'<w:checked(?: w:val="([^"]*)")?/>', run)
    on = (chk.group(1) not in ("0", "false")) if chk else bool(re.search(r'<w:default w:val="(1|true)"/>', run))
    count[on] += 1
    return '<w:r><w:t xml:space="preserve">%s </w:t></w:r>' % ("☒" if on else "☐") + run
new = re.sub(r'<w:r\b(?:(?!</w:r>).)*?<w:fldChar w:fldCharType="begin">\s*<w:ffData>.*?</w:ffData>\s*</w:fldChar>\s*</w:r>',
             box, doc, flags=re.S)
with zipfile.ZipFile(dst, "w", zipfile.ZIP_DEFLATED) as zout:
    for item in zin.infolist():
        zout.writestr(item, new.encode("utf8") if item.filename == "word/document.xml" else zin.read(item.filename))
def has_text(prefix):
    return sum(1 for n in names if n.startswith(prefix) and re.search(r"<w:t[ >]", zin.read(n).decode("utf8", "ignore")))
report = {
    "form checkboxes (now written as text: unticked, ticked)": tuple(count),
    "text boxes / shapes with text": doc.count("<w:txbxContent"),
    "tracked insertions / deletions": (doc.count("<w:ins "), doc.count("<w:del ")),
    "comments": doc.count("<w:commentReference"),
    "headers / footers with text": has_text("word/header") + has_text("word/footer"),
    "charts / SmartArt / embedded objects": sum(n.startswith(("word/charts/", "word/diagrams/", "word/embeddings/")) for n in names),
    "dropdowns / other content controls": doc.count("<w:dropDownList") + doc.count("<w:comboBox") + doc.count("<w:ddList"),
    "equations": doc.count("<m:oMath>") + doc.count("<m:oMath "),
}
for k, v in report.items():
    if v and v != (0, 0): print(f"{k}: {v}")
print("docx_prep done ->", dst)
```

```bash
python3 docx_prep.py in.docx in_prepped.docx
pandoc in_prepped.docx -s -t gfm+tex_math_dollars --wrap=none --extract-media=in_media -o in.md
```

Act on what the pre-check reports: text in text boxes has to be pulled from `word/document.xml` and placed where it sits on the page; charts and SmartArt have no image inside the file, so render the page (Word -> PDF) and crop them; tracked changes are accepted by default (mention it); comments are dropped unless the user wants them (`--track-changes=all`); a header/footer that holds real content (a name, a student ID) should be added once at the top.

Then tidy the Markdown so its structure matches how the document looks, not how it happened to be formatted:

- `-s` keeps a Title-styled title as front matter. If the title is only big bold text, it arrives as a bold paragraph: make it the single `#` heading and move the real section headings down one level.
- Strip bold markers pandoc leaves inside headings (`# **Methods**` -> `## Methods`).
- Indented paragraphs arrive as `>` block quotes. Keep the quote only where the text is a quotation or call-out; turn indented checklist lines into task lists (`☐ item` -> `- [ ] item`, `☒ item` -> `- [x] item`) and other indented lines into plain paragraphs or list items.
- Equations become LaTeX in `$...$` / `$$...$$`; footnotes become `[^1]` notes.
- Simple tables become pipe tables. Tables with merged cells come out as HTML `<table>` blocks - keep them as HTML, because a pipe table cannot express merged cells and flattening them scrambles the data.
- Images come out as `<img ...>` tags; rewrite them to `![alt](path)`:
  ```bash
  python3 -c "import re,sys;p=sys.argv[1];s=open(p).read();s=re.sub(r'<img src=\"([^\"]+)\"[^>]*?alt=\"([^\"]*)\"[^>]*/>',r'![\2](\1)',s);s=re.sub(r'<img src=\"([^\"]+)\"[^>]*/>',r'![](\1)',s);open(p,'w').write(s)" in.md
  ```
- Pandoc sometimes repeats a table caption above and below the table; delete the duplicate.
- Colour and highlighting are lost. If colour carries meaning that is not also in the words (e.g. red = marker's comment with no label), add a short label rather than losing the distinction.

### Markdown -> Word

```bash
pandoc in.md -o in.docx --resource-path="$(dirname in.md)"
```

LaTeX maths becomes native, editable Word equations; pipe tables become Word tables; task lists become checkbox lines; images are embedded. If the user has a template, add `--reference-doc=template.docx`. If an image path is broken pandoc only warns - read the warnings, a missing figure is a major loss.

Pandoc turns an image's alt text into a visible caption. When the Markdown already has the caption or heading as its own line next to the image (as files produced by this skill from PDFs do), add `-f markdown-implicit_figures`, otherwise every figure gets its caption twice.

### Markdown -> PDF

Go through HTML and print with Chromium; it renders equations (MathML), tables, images and Chinese/Japanese/Korean text reliably.

```bash
cat > print.css <<'CSS'
@page { size: A4; margin: 2.5cm; }
html { font-size: 11pt; }
body { max-width: none; margin: 0; padding: 0; line-height: 1.5;
       font-family: "Liberation Serif", "Times New Roman", "DejaVu Serif", "Noto Serif CJK TC", serif; }
h1, h2, h3, h4 { break-after: avoid; }
table { border-collapse: collapse; margin: 1em auto; break-inside: avoid; }
th, td { padding: 0.3em 0.8em; border-bottom: 1px solid #999; }
th { border-bottom: 2px solid #333; }
img { max-width: 100%; }
figure { break-inside: avoid; text-align: center; margin: 1em 0; }
pre { white-space: pre-wrap; }
CSS
pandoc in.md -s --mathml --embed-resources --metadata pagetitle="in" --resource-path="$(dirname in.md)" -c print.css -o in.html
"$CHROME" --headless --no-sandbox --disable-gpu --no-pdf-header-footer \
  --print-to-pdf=in.pdf "file://$(realpath in.html)" 2>/dev/null
```

Keep a Latin font ahead of the CJK font in `font-family`: if the CJK font comes first, curly quotes and apostrophes are drawn full-width (`group' s`). Add `-f markdown-implicit_figures` under the same condition as for Word.

### Word -> PDF

First count the equations: `unzip -p in.docx word/document.xml | grep -o "<m:oMath" | wc -l`.

- **No equations, or LibreOffice has its Math module** -> `soffice --headless --convert-to pdf --outdir . in.docx`. This keeps the Word layout, including form checkboxes.
- **Equations present and no Math module** -> LibreOffice would silently drop them. Use `pandoc in_prepped.docx -s --mathml --embed-resources --extract-media=in_media -c print.css -o in.html` (after `docx_prep.py`), then print with Chromium as above. The layout becomes plain, but nothing is lost. Tell the user in one line that the layout was simplified to keep the equations.
- If the session is linked to the user's computer and a Word export tool is available there, that gives the most faithful PDF; prefer it for a final assignment submission.

Either way, rasterise the result and look at any page that should contain an equation.

### Plain text (.txt) -> anything

A text file is not Markdown: pandoc would merge its lines into one paragraph and treat stray `*`, `_`, `#` or `<` as formatting. Turn it into Markdown first - normalise line endings, make each non-empty line its own paragraph (for hard-wrapped text, join lines within a block and split on blank lines), backslash-escape Markdown's special characters, and bold obvious labels such as `SPEAKER 0` - then use the Markdown routes. Do not add headings, split long paragraphs or clean up a transcript's wording unless asked; that is editing, so offer it instead.

### PDF -> Markdown

**First decide whether the text layer can be trusted.** A PDF can look perfect and still hold text that is absent, incomplete or scrambled.

```bash
python3 -c "import pymupdf,sys;d=pymupdf.open(sys.argv[1]);print(len(d),'pages; chars/page:',[len(p.get_text().strip()) for p in d][:40]);print(d[min(1,len(d)-1)].get_text()[:600])" in.pdf
pdffonts in.pdf | head -15
```

- Almost no characters per page -> scanned; see "Scanned PDFs".
- Read the printed sample against the page image. If words on the page are missing from it, it is scrambled, or one script is gone (typically Latin text present and all the Chinese missing), the layer is unreliable. Fonts listed by `pdffonts` with `uni` = `no` and a `Custom` or `Identity` encoding are the usual cause (Type 3 fonts from plotting libraries, some LaTeX and print-to-PDF output). Treat the affected text as scanned: transcribe it from the page images. The parts of the layer that are correct still help - use them, and the word positions, to cross-check what you typed.

For a PDF with a sound text layer, extract (run from the output folder so image links are relative):

```bash
python3 -c "
import pymupdf4llm, sys, pathlib
src, stem = sys.argv[1], pathlib.Path(sys.argv[1]).stem
md = pymupdf4llm.to_markdown(src, write_images=True, image_path=stem + '_media', dpi=200)
pathlib.Path(stem + '.md').write_text(md)" in.pdf
```

That output is a first draft, not the result. Compare it against the page images and repair:

- **Equations** are the main casualty: they arrive as scrambled symbols, or not at all. Look at each page that has maths and retype every equation as LaTeX (`$...$` inline, `$$...$$` display), keeping equation numbers. Sometimes a whole paragraph containing maths is exported as a picture instead of text; retype the paragraph and remove the picture. This also applies to chemical formulas and anything with sub/superscripts (`CO<sub>2</sub>` or `$\mathrm{CO_2}$`).
- **Tables**: check each against the page - right number of rows and columns, numbers in the right cells, header row intact, units and footnote markers kept. Tables without ruled lines are often missed entirely and appear as loose lines of text; rebuild them as pipe tables. Use an HTML table for merged cells.
- **Figures**: embedded photos are exported, but charts and diagrams drawn as vector graphics are skipped or shredded into fragments, and their axis labels get dumped into the body text. Crop each one from the rendered page as a single image (see "Cropping figures") and delete the stray label text: words that belong to a figure stay inside the figure. Put the caption directly under the image as its own line.
- **Headings**: fix levels so they follow the document's real outline (one `#` title, `##` sections, ...) and strip stray bold markers from heading text. Do not invent headings the document does not have.
- **Reading order**: in two-column articles check that columns were not interleaved and that a figure or footnote did not split a sentence.
- **Noise**: remove running headers/footers and page numbers; rejoin words hyphenated across lines. Keep footnotes, and keep the reference list complete.
- **Slides and figure-heavy guides**: one heading per slide or panel (its title), its text as paragraphs or bullets, and every chart or diagram kept as an image. For these documents the extractor's draft is usually not worth repairing: build the Markdown directly from the page images and crops.

### Cropping figures

Save as `figcrop.py`. It crops a page region to a PNG and logs the region in `figures.json`, which lets the check ignore words that live inside figures.

```python
#!/usr/bin/env python3
"""figcrop.py IN.pdf PAGE X0 Y0 X1 Y1 OUT.png - crop a region (PDF points, top-left origin) and log it."""
import sys, json, os, pymupdf
pdf, page, out = sys.argv[1], int(sys.argv[2]), sys.argv[7]
rect = [float(v) for v in sys.argv[3:7]]
os.makedirs(os.path.dirname(out) or ".", exist_ok=True)
pymupdf.open(pdf)[page - 1].get_pixmap(clip=pymupdf.Rect(*rect), dpi=170).save(out)
log = "figures.json"
figs = json.load(open(log)) if os.path.exists(log) else []
figs.append({"page": page, "rect": rect}); json.dump(figs, open(log, "w"))
```

Find the region from the text around the figure rather than by guessing: `page.get_text("dict")` gives the position and font size of every text span (positions are right even when the characters are not), so a figure runs from the bottom of the paragraph or heading above it to the top of its caption or the next heading. When pages share one layout, work the boxes out once and loop. Look at a few crops afterwards: nothing cut off (legends and axis titles often sit outside the plot area), and no body text caught at the edges.

### PDF -> Word

Default: produce the clean Markdown as above (with repairs), then convert Markdown -> Word. This gives real heading styles, real tables and editable equations, which is what "make this PDF editable" usually means.

Only when the user wants the page to *look* like the original (forms, posters, designed layouts) use the layout-preserving converter instead:

```bash
python3 -c "
from pdf2docx import Converter; import sys
c = Converter(sys.argv[1]); c.convert(sys.argv[2]); c.close()" in.pdf in.docx
```

Its limits: headings are just bold text, footnotes lose their links, equations come out as loose characters, and it copies whatever the text layer says - so it must not be used on a PDF whose text layer failed the trust check.

### Scanned PDFs (and untrustworthy text layers)

The content has to be read from the image. Render the pages (`pdftoppm -r 150 -png in.pdf pages/p`) and read each one yourself with the Read tool, writing the Markdown as you go: this preserves equations, tables, sub/superscripts and any language, where OCR garbles all of them. For small print, crop the text region at higher resolution (220+ dpi) rather than reading a whole page; several strips can be stacked into one image. For long documents (more than ~30 pages) run OCR for the body text first (`pymupdf4llm.to_markdown` does this through Tesseract when installed), then go through the page images to correct it - always redo equations, tables and any non-English text by eye (check `tesseract --list-langs`; text in a language that is not installed comes out as nonsense).

Transcription is where errors creep in, so cross-check it with whatever independent evidence exists: any correct part of the text layer (every Latin word and number it holds should appear in your text), and for characters you were unsure of, a zoomed crop. Tell the user that the text was transcribed from page images, so names, numbers and rare characters deserve a second look.

## Looking at pages

```bash
mkdir -p pages && pdftoppm -r 100 -png in.pdf pages/p    # then Read pages/p-01.png ...
```

Use `-f N -l M` for a page range and `-r 150` when small print or subscripts matter. For a first overview of a long document, paste low-resolution pages into contact sheets (8 pages per image). For documents up to ~30 pages look at every page; for longer ones look at every page that has a table, figure or equation plus a sample of plain pages. To check a .docx or .md visually, convert it to PDF first.

## Verify

Save this as `losscheck.py` and run `python3 losscheck.py SOURCE OUTPUT [figures.json]` after every conversion. It prints `RESULT: OK` or `RESULT: CHECK` with the reasons.

```python
#!/usr/bin/env python3
"""losscheck.py SRC OUT [figures.json] - compare what a document held before and after conversion.
figures.json (optional) lists PDF regions kept as images, so the words inside them are not expected in OUT."""
import sys, re, json, subprocess, collections, zipfile, shutil

def tokens(text):
    return collections.Counter(re.findall(r"[a-z]{2,}|\d+(?:\.\d+)?|[㐀-鿿]", text.lower()))

def via_pandoc(path):
    fmt = "docx" if path.lower().endswith(".docx") else "markdown+tex_math_dollars"
    ast = json.loads(subprocess.run(["pandoc", "-f", fmt, "-t", "json", path],
                                    capture_output=True, text=True, check=True).stdout)
    inv = dict(text=[], extra="", headings=0, tables=0, images=0, equations=0, notes=[])
    def walk(x):
        if isinstance(x, list):
            for i in x: walk(i)
        elif isinstance(x, dict):
            t, c = x.get("t"), x.get("c")
            if t == "Str": inv["text"].append(c)
            elif t in ("Code", "CodeBlock"): inv["text"].append(c[1])
            elif t == "Math": inv["equations"] += 1; return
            elif t == "Header": inv["headings"] += 1
            elif t == "Table": inv["tables"] += 1
            elif t == "Image": inv["images"] += 1; return  # alt text repeats the caption
            elif t in ("RawBlock", "RawInline"):
                inv["tables"] += len(re.findall(r"<table\b", c[1]))
                inv["images"] += len(re.findall(r"<img\b", c[1]))
                inv["text"].append(re.sub(r"<[^>]+>", " ", c[1])); return
            for v in x.values(): walk(v)
    walk(ast["blocks"]); walk(ast.get("meta", {}))
    inv["text"] = " ".join(inv["text"])
    if fmt == "docx":  # read the raw XML too, so text pandoc skips (text boxes etc.) still counts as source
        z = zipfile.ZipFile(path)
        for n in z.namelist():
            if re.fullmatch(r"word/(document|footnotes|endnotes)\.xml", n):
                x = z.read(n).decode("utf8", "ignore")
                inv["extra"] += " " + " ".join(re.findall(r"<w:t(?: [^>]*)?>([^<]*)</w:t>", x))
                inv["checkboxes"] = inv.get("checkboxes", 0) + x.count("<w:checkBox>")
    return inv

def via_pdf(path, figs):
    import pymupdf
    doc = pymupdf.open(path)
    text, tables, images = [], 0, 0
    for i, page in enumerate(doc, 1):
        boxes = [pymupdf.Rect(f["rect"]) for f in figs if f["page"] == i]
        for x0, y0, x1, y1, word, *_ in page.get_text("words"):
            if not any(b.contains(pymupdf.Point((x0 + x1) / 2, (y0 + y1) / 2)) for b in boxes): text.append(word)
        images += len(page.get_images())
        try: tables += sum(1 for t in page.find_tables().tables
                           if not any(b.intersects(pymupdf.Rect(t.bbox)) for b in boxes))
        except Exception: pass
    text = " ".join(text)
    inv = dict(text=text, extra="", headings=None, tables=tables, images=images, equations=None, notes=[], pdf=True)
    if len(text.strip()) < 50 * len(doc):
        inv["notes"].append("scanned / image-only PDF: no usable text layer - check the page images by eye")
    elif shutil.which("pdffonts"):
        rows = subprocess.run(["pdffonts", path], capture_output=True, text=True).stdout.splitlines()[2:]
        bad = [r for r in rows if len(r.split()) >= 6 and r.split()[-3] == "no" and re.search(r"Custom|Identity", r)]
        if bad: inv["notes"].append(f"{len(bad)} font(s) have no Unicode map, so the PDF text layer may be garbled or "
                                    "missing whole scripts - compare extracted text with the page images")
    # PDFs hide structure: table/image counts are rough, so only large drops are worth a look.
    inv["images"] = None if figs or inv["notes"] else images
    return inv

def inventory(path, figs):
    p = path.lower()
    if p.endswith(".pdf"): return via_pdf(path, figs)
    if p.endswith(".txt"):
        return dict(text=open(path, encoding="utf-8-sig", errors="replace").read(), extra="", headings=None,
                    tables=None, images=None, equations=None, notes=[])
    return via_pandoc(path)

figs = json.load(open(sys.argv[3])) if len(sys.argv) > 3 else []
src, out = inventory(sys.argv[1], figs), inventory(sys.argv[2], [])
problems = [f"source: {n}" for n in src["notes"]] + [f"output: {n}" for n in out["notes"]]
a, b = tokens(src["text"] + " " + src["extra"]), tokens(out["text"] + " " + out["extra"])
# Distinct-word recall: a dropped section takes its unique words and numbers with it.
recall = sum(1 for w in a if w in b) / (len(a) or 1)
length = sum(tokens(out["text"]).values()) / (sum(tokens(src["text"]).values()) or 1)
limit = 0.92 if src.get("pdf") else 0.97
print(f"distinct words kept: {recall:.1%} of {len(a)}; length ratio: {length:.2f}")
boxes = lambda t: len(re.findall(r"[☐-☒]", t))
if src.get("checkboxes") and not out.get("pdf") and boxes(out["text"]) < boxes(src["extra"]) + src["checkboxes"]:
    problems.append(f"source has {src['checkboxes']} form checkboxes whose ticked/unticked state is missing from the output - run docx_prep.py first")
if not a:
    pass  # nothing readable in the source to compare against
elif recall < limit:
    gone = sorted((w for w in a if w not in b), key=lambda w: -a[w])[:25]
    problems.append(f"only {recall:.1%} of distinct words survived; missing: " + ", ".join(gone))
if length < 0.80:
    problems.append(f"output has only {length:.0%} of the source's word count - something may be truncated")
for key in ("headings", "tables", "images", "equations"):
    s, o = src[key], out[key]
    print(f"{key}: {s if s is not None else '?'} -> {o if o is not None else '?'}")
    if out.get("pdf") and key in ("tables", "images"): o = None  # cannot count these in a PDF reliably
    if s is not None and o is not None and o < s:
        problems.append(f"{key} dropped from {s} to {o}")
print("RESULT: " + ("OK" if not problems else "CHECK\n- " + "\n- ".join(problems)))
```

How to read it:
- The "missing" word list points at what was lost - search the source for those words to find the passage, table or reference that did not make it, and restore it. Missing numbers usually mean a table lost cells.
- When the source is a PDF, a little is expected to go (page numbers, running headers); that is why its threshold is lower. Pass `figures.json` when figures were cropped, otherwise every axis label counts as lost text.
- **The script only knows what its own readers can see in the source.** `?` marks what it cannot count: equations and tables inside a PDF, anything in a scan. A PDF with a broken text layer under-reports its own content, so the script can say OK while half the text is missing - which is why its font warning must be followed up by eye. Whenever a PDF is on either side, the script is not enough: look at the pages and confirm each equation, table and figure is present and correct.
- A PDF source reports tables and images as rough counts, so more in the output than in the source is normal.
- A warning that cannot be cleared (the source really is a scan, the fonts really have no Unicode map) is fine to finish on once the visual comparison has been done.

Only finish when the script is OK or every flagged item is explained, and the visual checks pass. If something truly cannot be carried over - an equation too degraded to read, a figure that cannot be cropped cleanly - keep it as an image of that region rather than dropping it, and tell the user.
