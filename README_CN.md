# bilingual-book 📖

将任何英文书籍（EPUB/PDF/文本）转换为中英双语对照版——一段英文、一段中文——使用 AI 作为翻译器，无需外部 API。

## 使用方式

### 作为 Kiro CLI Skill（推荐）

无需 API key，Kiro 自己完成翻译。

**安装：**

```bash
git clone https://github.com/Seymer/bilingual-book.git ~/.kiro/skills/bilingual
```

**使用：**

```
# 在 Kiro CLI 中直接说：
> 把这本书做成双语对照版 /path/to/book.epub
> 翻译这本书为中英对照 /path/to/book.pdf
```

**工作流程：**
1. 提取文本内容
2. 按章节边界智能分块（不会把一章切成两半）
3. 并行翻译（多个 subagent 同时工作）
4. 生成带目录导航的 EPUB

### 作为独立命令行工具

需要 API key，适合自动化场景。

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...

python bilingual_book.py book.epub                   # → 双语 EPUB
python bilingual_book.py book.epub --format pdf      # → 双语 PDF
python bilingual_book.py book.epub --provider openai # 使用 GPT-4o
```

## 输出效果

> **English original text here.**
>
> 中文翻译在这里。

- 英文黑色，中文灰色
- 关键语句高亮显示
- 完整章节目录导航
- 适合微信读书、Apple Books、Kindle

## 翻译风格

- 优先"达"（通顺自然），兼顾"信"（忠实原意）
- 保留英文专有名词
- 保持原文正式/非正式程度
- 长句可适当拆分以符合中文表达习惯

## 依赖

- Python 3.8+
- pandoc: `brew install pandoc`
- ebooklib: `pip install ebooklib`

## 许可证

MIT
