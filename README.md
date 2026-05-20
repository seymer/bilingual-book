# bilingual-book 📖

[中文说明](README_CN.md)

Turn any English book (EPUB/PDF/text) into a bilingual English-Chinese edition — one paragraph English, one paragraph Chinese — using AI as the translator. No external API needed when used as a Kiro/Claude skill.

## Two Ways to Use

### 1. As a Kiro CLI Skill (Recommended)

Kiro does the translation directly — no API key needed.

**Install:**

```bash
# Clone into your Kiro skills directory
git clone https://github.com/Seymer/bilingual-book.git ~/.kiro/skills/bilingual
```

**Use:**

```
# In Kiro CLI, just say:
> 把这本书做成双语对照版 /path/to/book.epub
> Make this article bilingual /path/to/article.pdf
```

Kiro will:
1. Extract content from your EPUB/PDF/text
2. Smart-chunk by chapter boundaries (not arbitrary line counts)
3. Translate in parallel using subagents
4. Generate EPUB with full TOC navigation

### 2. As a Standalone CLI Tool

For automation or use outside Kiro. Calls Anthropic/OpenAI API for translation.

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...  # or OPENAI_API_KEY

python bilingual_book.py book.epub                   # → bilingual EPUB
python bilingual_book.py book.epub --format pdf      # → bilingual PDF
python bilingual_book.py book.epub --provider openai # Use GPT-4o
```

## Output Format

Each paragraph appears as:

> **English original text here.**
>
> 中文翻译在这里。

- English in black, Chinese in gray
- Key quotes highlighted in accent color
- Full chapter navigation / bookmarks

## Architecture

```
bilingual-book/
├── SKILL.md             # Kiro CLI skill instructions
├── generate.py          # EPUB/PDF generator
├── bilingual_book.py    # Standalone CLI tool (with API translation)
├── requirements.txt     # Python dependencies
└── README.md
```

**Skill mode** (Kiro CLI):
```
User input → Kiro extracts text → subagents translate in parallel
           → generate.py produces EPUB/PDF
```

**CLI mode** (standalone):
```
User input → bilingual_book.py extracts text → API translates
           → bilingual_book.py produces EPUB/PDF
```

## Requirements

- Python 3.8+
- `pandoc`: `brew install pandoc` / `apt install pandoc`
- `pip install ebooklib` (for EPUB output)
- `pip install reportlab` (for PDF output, optional)
- For CLI mode: `pip install anthropic` or `pip install openai`

## Translation Quality

The skill uses a translation style guide:
- Prioritizes fluency (达) while maintaining faithfulness (信)
- Preserves English proper nouns in Chinese text
- Maintains the formality level of the original
- May split long sentences for natural Chinese expression

## License

MIT
