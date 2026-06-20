---
name: douban-best-edition
version: 1.1.0
description: Find the best edition or version of ANY book using Douban scores and reviews, then automatically check WeRead (微信读书) availability for immediate reading — covers translations, different publishers, reprints, and formats. Use when the user asks "哪个版本最好", "best edition/version", "哪个译本最好", "哪个出版社的好", "哪一版值得买", "台版还是大陆版", "原版还是译本", or compares different editions of the same book. Triggers for both translated works AND original-language works with multiple publishers/reprints. Also use when the user wants to know which specific edition to buy or read.
---

# Douban Best Edition

Compares all editions/versions of a book on Douban, recommends the best one, then checks if that edition is available on WeRead (微信读书) for immediate reading. Works for:
- **Translated works**: compares translators, completeness, censorship
- **Original works**: compares publishers, print quality, errata, reprint improvements
- **Mixed**: original vs translation (e.g. "read English or Chinese?")

## Dependencies

| Skill | 必需？ | 用途 | 安装 |
|-------|--------|------|------|
| [weread-skills](https://github.com/flow67bro/weread-skills) | 可选 | Phase 5 微信读书上架匹配 + 生成 `weread://` 直达链接 | 见该仓库 README |

> 缺少 `weread-skills` 时，Phase 5 自动跳过，不影响豆瓣版本对比的核心功能。

## Tool requirement

Uses **Jina** (`r.jina.ai`) to fetch Douban pages as Markdown — no browser, no CDP, no login needed. Requires only `curl`.

Jina rate limit: ~20 RPM. Parallel requests are fine but don't fire more than 5 at once.

## Fetching Douban pages

Every Douban URL is fetched through Jina with one pattern:

```bash
curl -s -L "https://r.jina.ai/<DOUBAN_URL>" -H "Accept: text/markdown"
```

Add `| head -N` to limit output when you only need the top portion of a page.

## Workflow

### Phase 1: Locate editions via search

1. **Search Douban general search** (always start here — it shows editions inline with scores, ratings, publisher, year, and translator):
   ```bash
   curl -s -L "https://r.jina.ai/https://www.douban.com/search?cat=1001&q=<URL-ENCODED-TITLE>" \
     -H "Accept: text/markdown"
   ```
   Add `| head -200` to keep output manageable.

2. **Parse search results**: Each result contains:
   - Subject ID (extracted from `subject/XXXXXXX/` in the URL)
   - Score (e.g. `9.1`)
   - Rating count (e.g. `274000人评价`)
   - Meta line (translator / publisher / year)
   
   **Filter out irrelevant results**: items that share keywords but aren't the target book (e.g. same-title novels by different authors, academic papers, comic adaptations). Only keep items where the title and author/translator reasonably match.

3. **Multi-name search** (critical for translated works): Taiwan editions often use a completely different Chinese title. Always run additional searches:
   - The **original title** (e.g. English/French title)
   - Common **Taiwan edition names** (e.g. "異鄉人" for 局外人 in Taiwan, or search "加缪 台版" / "加缪 繁体")
   - If you don't know these, search `"<author> 台版"` or `"<author> 繁体"` to discover them
   
   Run these searches **in parallel** as separate curl commands.

4. **Extracting subject IDs from Jina output**: Douban search results rendered by Jina show URLs like `https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F4908885%2F`. Extract the numeric ID after `subject/` from these URLs — that's the canonical subject ID. Use it to construct direct URLs:
   ```
   https://book.douban.com/subject/<ID>/
   ```

5. If the search doesn't show enough editions, also check the "其他版本" (other editions) section on a subject page.

### Phase 2: Collect edition data in parallel

Pick the top 5-6 most relevant candidate editions (different translators/publishers; skip near-duplicates like same translator/same publisher with different year). For each, fetch the subject page **in parallel**:

```bash
# Example for subject 4908885
curl -s -L "https://r.jina.ai/https://book.douban.com/subject/4908885/" \
  -H "Accept: text/markdown" | head -80 &
# Example for subject 1052203  
curl -s -L "https://r.jina.ai/https://book.douban.com/subject/1052203/" \
  -H "Accept: text/markdown" | head -80 &
# ...etc, then wait
```

From each subject page, extract:
- Score and star distribution (5★/4★/3★/2★/1★ percentages)
- Rating count (评价人数)
- Translator / author
- Publisher, year, series info
- Binding and price (if visible)

Jina renders the star distribution block as inline text in Markdown, usually near the rating section.

### Phase 3: Collect review evidence

For each major edition, fetch short comments and edition-specific reviews **in parallel**:

**Short comments** (these are always edition-specific, not aggregated):
```bash
curl -s -L "https://r.jina.ai/https://book.douban.com/subject/<ID>/comments/?sort=new_score&status=P" \
  -H "Accept: text/markdown" | head -100
```

**Edition-specific long reviews** (the `?version=1` parameter is critical — without it, Douban aggregates reviews across all editions):
```bash
curl -s -L "https://r.jina.ai/https://book.douban.com/subject/<ID>/reviews?version=1" \
  -H "Accept: text/markdown" | head -120
```

Also search for **cross-edition comparison** discussions:
```bash
curl -s -L "https://r.jina.ai/https://www.douban.com/search?q=<译本A译者>+<译本B译者>+<书名>" \
  -H "Accept: text/markdown"
```

**Scan for quality signals** (adapt to book type):

**For translated works**, look for reviews/comments mentioning:
- 译本 / 翻译 / 译者 / 译得 / 译文 / 信达雅
- 删减 / 删节 / 缺失 / 阉割 / 和谐
- 台版 / 大陆版 / 简体 / 繁体 / 原版对比
- 流畅 / 生硬 / 机翻 / 错误 / 不通顺
- Direct comparisons: "对比了A译本和B译本" or "读过台版和大陆版"

**For original works**, look for:
- 印刷 / 纸质 / 装帧 / 排版 / 字体 / 插图
- 错字 / 错误 / 校对 / 勘误 / 修订
- 删减 / 删节 / 和谐 / 完整版 / 未删节
- 精装 / 平装 / 轻型纸 / 收藏 / 便携

**Quote decisive evidence** verbatim — reviewer name, useful-count, and exact quote. Multi-edition comparison quotes are the most valuable evidence.

### Phase 3.5: Book content quality (highly recommended)

When the user asks a general question like "最好的版本" or "值不值得读", also evaluate the book's content — not just edition differences.

Scan short comments and reviews for content opinions (exclude pure translation/physical-quality mentions):
- 啰嗦 / 注水 / 干货 / 深度 / 核心观点
- 收获 / 启发 / 醍醐灌顶 / 值得一读
- 文风 / 难读 / 失望 / 不推荐
- 适合谁读 / 推荐给

Extract 3-5 representative quotes showing the range of opinions (positive, mixed, negative). Present as a **separate section** before the edition comparison.

### Phase 4: Rank and recommend

Rank editions on these axes, weighted by book type:

| Priority | Translated work | Original work |
|----------|----------------|---------------|
| 1 | **Completeness** — any censored/deleted content? Hard disqualifier | **Completeness** — any cuts from original? |
| 2 | **Translation quality** — reviewer testimony about accuracy and readability | **Print/production quality** — errors, paper, binding |
| 3 | **Score × volume** — high score with many ratings | **Score × volume** |
| 4 | **Translator credentials** | **Edition improvements** — revised? new content? |
| 5 | Price/availability | Price/availability |

Rules:
- If one edition has confirmed deletions/censorship and another doesn't, the complete one WINS regardless of score
- For translated works: if reviews say "台版更完整/更好", prioritise Taiwan edition
- For original works: favour the edition reviewers say has better production quality; prefer the most recent revision when scores are comparable
- English/foreign-language original is a valid recommendation — mention it if significantly higher quality
- If no clear winner from review evidence, default to highest-scored edition with >100 ratings

**Rating count reliability tiers:**
- <10 ratings: "数据严重不足，无法纳入比较" — skip this edition
- 10–50 ratings: "数据偏少，仅供参考" — mention but don't rely on
- 50–200 ratings: "可参考，置信度一般"
- >200 ratings: "数据充分" — reliable

### Phase 5: Cross-reference with WeRead (微信读书匹配)

After determining the best edition(s), automatically search WeRead to find the recommended edition so the user can start reading immediately.

**Prerequisites**: Requires `$WEREAD_API_KEY` environment variable. If not set, skip this phase and note that WeRead availability could not be checked.

**WeRead search endpoint**:

```bash
curl -s -X POST "https://i.weread.qq.com/api/agent/gateway" \
  -H "Authorization: Bearer $WEREAD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"api_name": "/store/search", "keyword": "<书名> <译者>", "scope": 10, "skill_version": "1.0.3"}'
```

> Request body rules: Parameters must be flat at top level (not nested in `params`). Always include `"skill_version": "1.0.3"`.

**Matching strategy** (two-round):

1. **Precise match**: Search with `{书名} {译者名}` — e.g. `"社会性动物 邢占军"`. This targets the exact translator from the Douban recommendation.
2. **Fallback broad match**: If round 1 returns 0 results or no translator match, search with `{书名}` only to find all available editions on WeRead.

**Cross-reference logic**:

From the WeRead response (`results[].books[].bookInfo`), compare `title`, `author`, `translator`, `publisher` against the Douban-recommended edition:

| Match level | Criteria | Icon |
|-------------|----------|------|
| **Exact match** | Same title, same translator (or same author for originals) | ✅ |
| **Likely match** | Same title, different translator/edition | ⚠️ |
| **Not found** | No matching title on WeRead | ❌ |

**Deep link generation**:
- Extract `bookId` from the matched WeRead result
- Generate: `weread://reading?bId={bookId}`

**Fallback recommendation**: If the recommended edition is not on WeRead, list all available editions from the broad search as alternatives, with their bookId and deep link.

**Scoring on WeRead**: WeRead scores are 0-100 (divide by 10 for comparable scale). Note the WeRead score alongside the Douban score for additional context.

### Output format

```
### 这本书到底怎么样（内容评价）

> @reviewer1（力荐）：...（正面观点）
> @reviewer2（还行）：...（批评观点）

**适合**：...；**不适合**：...

## 《<title>》版本对比

| | 版A | 版B | ... |
|---|---|---|---|
| 书名 | | | |
| 作者/译者 | | | |
| 出版社 | | | |
| 年份 | | | |
| 豆瓣评分 | | | |
| 评价人数 | | | |
| 5星占比 | | | |
| 装帧/定价 | | | |

### 版本质量证据
> @reviewer1：...
> @reviewer2：...

### 结论
**推荐：<edition name>**
理由：...

### 📖 微信读书可用性

✅ 已找到：[书名](weread://reading?bId=XXXXX) — 豆瓣推荐版本可直接阅读
（或）
❌ 微信读书暂无豆瓣推荐版本
⚠️ 替代：以下版本可在微信读书阅读：
- [版本A](weread://reading?bId=XXXXX) — 译者/出版社不同
- [版本B](weread://reading?bId=XXXXX)
```

Add clear emoji markers for the final picks: 🥇首选 🥈风格之选 🥉新锐之选 / 🔰入门之选.

## Notes

- Douban general search (`douban.com/search?cat=1001`) is the primary discovery method — it shows MORE editions than the "其他版本" section on a single subject page.
- **Reviews aggregation trap**: Douban's long reviews page (`/subject/<ID>/reviews`) aggregates across all editions at the works level. Always use `/subject/<ID>/reviews?version=1` to see only that specific edition's reviews. Short comments (短评) are NOT aggregated and are always edition-specific.
- Taiwan editions use traditional characters and often have completely different Chinese titles than mainland editions. Always search original title + common Taiwan translations + "台版" separately.
- For web novels / 网络小说, editions are different physical publishers of the same digital content — compare print quality, completeness, and extras (bonus chapters, illustrations).
- Jina output preserves the page's information hierarchy in Markdown. Scores, star distribution, and review text are rendered inline — no DOM traversal needed.
- If Jina returns a login wall or very incomplete data for a page, try the same URL once more. If it still fails, note the limitation and skip that source.
- **Replaces old CDP-based version**: An earlier version of this skill used CDP/browser automation. This Jina-based rewrite is the current and only version — simpler, faster, no browser needed.
- **WeRead API integration**: After ranking editions, automatically call WeRead Agent Gateway to check availability. Use precise match (书名+译者) first, fall back to broad match (书名 only). Always generate `weread://` deep links for matched books. If `$WEREAD_API_KEY` is not set, skip this step and inform the user.
- **WeRead score scale**: WeRead scores are 0-100 (e.g. 835 = 8.35). When comparing with Douban scores, divide by 10.
