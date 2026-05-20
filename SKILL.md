---
name: bilingual
description: Convert English EPUB/PDF/text into bilingual English-Chinese editions. Each paragraph appears in English followed by Chinese translation. Use when the user mentions "双语", "bilingual", "中英对照", "translate this book", or provides an EPUB/PDF to convert.
---

# Bilingual Book Generator

Convert English content into bilingual English-Chinese editions — one paragraph English, one paragraph Chinese. Output as EPUB (or PDF) with TOC navigation.

## When to Use

- User provides an EPUB / PDF / text / URL and wants a bilingual version
- User says "双语", "bilingual", "英汉对照", "中英对照", "translate this book"

## Step 0: Check Dependencies

Before starting, verify and install if missing:

```bash
which pandoc || brew install pandoc
python3 -c "import ebooklib" 2>/dev/null || python3 -m pip install --break-system-packages ebooklib
```

## Step 1: Extract

| Input | Command |
|-------|---------|
| `.epub` | `pandoc input.epub -t plain --wrap=none > /tmp/book_full.txt` |
| `.pdf` | `pdftotext input.pdf /tmp/book_full.txt` or pdfplumber |
| `.txt` | Copy directly to `/tmp/book_full.txt` |
| URL | WebFetch → save to `/tmp/book_full.txt` |

Then count: `wc -l /tmp/book_full.txt`

## Step 2: Smart Chunking

**Do NOT split blindly by line count.** Instead:

1. Scan the text for chapter/part boundaries (lines starting with "Chapter", numbered headers like "1.", "2.", or all-caps section titles)
2. Group chapters into chunks of roughly 400 lines each, never splitting mid-chapter
3. If a single chapter exceeds 400 lines, it becomes its own chunk

This preserves context for better translation quality.

## Step 3: Translate via Parallel Subagents

Dispatch one subagent per chunk. Each creates `/tmp/bilingual_part_{N}.py`.

**Subagent prompt template:**

```
Read /tmp/book_full.txt lines {START}-{END}.

You are translating a book into bilingual format. Translate EVERY paragraph into natural, fluent Chinese (简体中文). Prioritize readability and faithfulness to meaning over literal word-for-word translation.

Create /tmp/bilingual_part_{N}.py with variable content_part_{N} — a Python list of tuples.

Format: (type, english, chinese) where type is:
- 'part' — major divisions: "Part I", "Chapter One", "Chapter Two" etc.
- 'chapter' — section headers within a chapter
- 'highlight' — 1-2 sentence key insights worth emphasizing
- 'text' — all other paragraphs

Rules:
- Include EVERY paragraph. Do not skip, merge, or summarize.
- Skip only blank lines, [] markers, and TOC listings.
- Keep proper nouns in English within Chinese text (e.g. "Karl Popper" stays as-is).
- Escape quotes: use 「」 for Chinese quotes, or escape \" for ASCII double quotes.
- The file MUST be valid Python. Verify with: compile(open(f).read(), f, 'exec')
- Write the ENTIRE list in ONE create operation (do not use insert/append).
```

**Dispatch rules:**
- ≤400 lines per chunk (prefer chapter-aligned boundaries)
- After ALL agents return, validate every file:
  ```bash
  python3 -c "compile(open('/tmp/bilingual_part_{N}.py').read(), 'f', 'exec')"
  ```
- If any file fails validation or is missing, retry that chunk once

## Step 4: Generate Output

Generate both bilingual and Chinese-only versions:

**Bilingual (中英对照):**
```bash
python3 {SKILL_DIR}/generate.py epub \
  --content-files /tmp/bilingual_part_1.py,/tmp/bilingual_part_2.py,... \
  --title "Book Title" \
  --author "Author" \
  --mode bilingual \
  --output /path/to/output_双语对照.epub
```

**Chinese-only (纯中文翻译版):**
```bash
python3 {SKILL_DIR}/generate.py epub \
  --content-files /tmp/bilingual_part_1.py,/tmp/bilingual_part_2.py,... \
  --title "Book Title" \
  --author "Author" \
  --mode chinese \
  --output /path/to/output_中文翻译版.epub
```

Both versions include a cover page and translator's preface automatically.

For PDF output (requires reportlab):
```bash
python3 -m pip install --break-system-packages reportlab pypdf
python3 {SKILL_DIR}/generate.py pdf \
  --content-files ... \
  --title "Book Title" \
  --author "Author" \
  --output /path/to/output.pdf
```

`{SKILL_DIR}` = directory containing this SKILL.md.

**Default behavior:** Generate BOTH versions (bilingual + chinese) unless user specifies otherwise.

## Step 5: Cleanup

Remove temp files after confirming output:
```bash
rm -f /tmp/bilingual_part_*.py /tmp/book_full.txt
```

## Content Types Reference

| Type | Use for | EPUB Style |
|------|---------|------------|
| `part` | Part I, Chapter One... | Large centered, page break before |
| `chapter` | Section headers | Colored heading (red-orange) |
| `highlight` | Key quotes/insights | Bold, accent color |
| `text` | Everything else | EN black, CN gray |

## Translation Style Guide

- **信达雅**: 优先"达"（通顺自然），兼顾"信"（忠实原意），适当追求"雅"
- **术语**: 保留英文专有名词，括号注中文（首次出现时）
- **语气**: 保持原文的正式/非正式程度
- **长句**: 可适当拆分以符合中文表达习惯，但不改变段落结构

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Subagent returns empty | Retry with smaller chunk (≤300 lines) |
| Python syntax error | Usually unescaped quotes — fix and revalidate |
| Missing paragraphs | Compare `wc -l` vs total entry count across all parts |
| Chapter split mid-sentence | Use smart chunking (Step 2) |
| Large book (>3000 lines) | May need multiple rounds; save progress between rounds |
