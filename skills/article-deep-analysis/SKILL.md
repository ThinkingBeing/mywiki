---
name: article-deep-analysis
description: 将用户提供的文章/帖子/文档进行中文深度解读，保存为 Jekyll Markdown 文章，并发布到 ThinkingBeing/mywiki GitHub Pages。
version: 1.0.0
author: ThinkingBeing / Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [article-analysis, jekyll, github-pages, markdown, publishing]
    repo: https://github.com/ThinkingBeing/mywiki
---

# Article Deep Analysis / 文章深度解读发布 Skill

这个 Skill 用于把用户感兴趣的文章、博客、帖子、文档或网页，转化为一篇中文深度解读，并发布到 `ThinkingBeing/mywiki` 的 GitHub Pages 文章库，方便后续检索和复用。

目标产物：

- 一篇中文 Markdown 解读文章
- 带 Jekyll front matter
- 提交到 `https://github.com/ThinkingBeing/mywiki`
- 在 `https://thinkingbeing.github.io/mywiki/` 可访问

---

## 适用场景

当用户说类似以下内容时使用：

- “解读这篇文章”
- “把这篇文章做成一篇分析发到我的 wiki / mywiki / GitHub Pages”
- “深入分析这个帖子/博客/论文”
- “保存一篇文章解读”
- “把这个链接整理成中文笔记”

默认输出语言：中文。

---

## 仓库与发布位置

仓库地址：

```text
https://github.com/ThinkingBeing/mywiki.git
```

常用本地路径：

```bash
/home/user/hermes/mywiki
```

Jekyll 文章目录：

```text
_posts/
```

推荐文件位置：

```text
_posts/YYYY-MM-DD-slug.md
```

如果当前仓库已有分类子目录约定，也可以放到：

```text
_posts/<category>/YYYY/MM/YYYY-MM-DD-slug.md
```

但必须在最终发布后访问 GitHub Pages 验证文章能被 Jekyll 收录。

---

## 工作流程

### 1. 获取原文内容

优先使用网页抽取工具获取正文、标题、作者、发布日期：

1. 用 `web_extract` 抓取 URL。
2. 如果失败，使用浏览器或 `curl` + Python 提取正文。
3. 如果原始链接不可访问，先尝试：
   - 官方 changelog / docs
   - 作者博客归档
   - Internet Archive
   - GitHub README / issue / release
   - 搜索文章标题或核心关键词
4. 如果仍然无法获取正文，明确告诉用户无法访问，并说明使用了哪些替代来源；不要凭空编造原文内容。

HTML 提取兜底示例：

```python
import re
import urllib.request
from html import unescape

url = "https://example.com/article"
req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
html = urllib.request.urlopen(req, timeout=30).read().decode("utf-8", errors="ignore")

patterns = [
    r'<article[^>]*>(.*?)</article>',
    r'<main[^>]*>(.*?)</main>',
    r'<div[^>]*class="[^"]*(?:post|content|markdown|article|entry)[^"]*"[^>]*>(.*?)</div>',
]

for pattern in patterns:
    m = re.search(pattern, html, re.S | re.I)
    if not m:
        continue
    text = re.sub(r'<(script|style)[^>]*>.*?</\\1>', '', m.group(1), flags=re.S | re.I)
    text = re.sub(r'<[^>]+>', '\n', text)
    text = unescape(text)
    text = re.sub(r'\n{3,}', '\n\n', text).strip()
    if len(text) > 500:
        print(text)
        break
```

---

### 2. 生成中文深度解读

解读不是翻译，也不是摘要。要回答：这篇文章为什么重要、作者在解决什么问题、论据是否可信、对用户有什么启发。

推荐结构：

```markdown
# 文章标题：中文副标题

> 来源: 来源名称 | [原文链接](URL) | 发布日期: YYYY-MM-DD

## 1. 文章基本信息

- **标题**: 原文标题
- **来源**: 来源/网站/博客
- **链接**: URL
- **作者**: 作者，如未知则写“未注明”
- **发布日期**: YYYY-MM-DD，如未知则写“未注明”
- **主题**: 关键词列表

## 2. 文章要解决的核心问题

用一两句话说清楚文章试图回答什么问题。

一句话概括：**……**

## 3. 背景上下文

说明这个问题为什么出现、为什么现在重要、业界现状是什么。

## 4. 文章核心观点

按 3-6 个小节展开。每个观点都要包含：

- 作者主张
- 关键论据
- 该观点的隐含前提
- 与当前 AI / 工程 / 产品实践的关系

## 5. 观点对应的论据与案例

把文章中的数据、案例、引用、代码、产品变化等整理出来；不要只复述结论。

## 6. 我的评价和启发

需要有判断，不要只说“很有启发”。至少覆盖：

- 哪些观点可信，为什么
- 哪些地方可能过度乐观或证据不足
- 对 AI Agent / 工程实践 / 产品策略的启发
- 如果我是实践者，下一步会怎么做

## 7. 延伸阅读 / 相关链接

列出原文提到或分析中自然关联的链接，如官方文档、GitHub 项目、论文、同主题文章。
```

质量要求：

- 中文表达自然，避免机器翻译腔。
- 不要堆砌空泛形容词。
- 引用原文观点时要忠实；自己的判断要明确标注。
- “核心观点”部分可以深入，但要有结构，不要流水账。
- 对 AI Agent、LLM、工程架构类文章，要重点分析其系统设计、边界条件和实践风险。

---

### 3. 创建 Jekyll 文章文件

文件命名：

```text
YYYY-MM-DD-short-slug.md
```

- `YYYY-MM-DD`：优先使用原文发布日期；未知时用解读创建日期。
- `slug`：英文小写、数字、连字符，不要使用中文或空格。

Front matter 必须包含：

```yaml
---
layout: post
title: "中文标题"
description: "一句话说明这篇解读的核心观点"
date: 2026-06-20
created_at: 2026-06-20 14:30:00 +0800
categories: AI Agents
source: "来源名称"
author: "作者或未注明"
original_url: "https://example.com/article"
---
```

关键注意事项：

- `layout: post` 必须存在。
- `title`、`description`、`source`、`author`、`original_url` 建议加引号。
- `date` 用原文日期，格式为 `YYYY-MM-DD`，不要加引号。
- `created_at` 用解读创建时间，格式如 `2026-06-20 14:30:00 +0800`，不要加引号。
- front matter 结束的 `---` 后必须空一行再写正文。
- 首页通常按 `created_at` 倒序展示，因此必须填写 `created_at`。

---

### 4. 本地检查

进入仓库：

```bash
cd /home/user/hermes/mywiki
```

检查文件是否存在：

```bash
git status --short
```

检查 front matter 常见问题：

```bash
python3 - <<'PY'
from pathlib import Path
p = Path('_posts/YYYY-MM-DD-slug.md')
s = p.read_text(encoding='utf-8')
assert s.startswith('---\n'), 'front matter must start with ---'
head, body = s.split('---', 2)[1:]
assert 'layout: post' in head
assert '\ndate: "' not in head, 'date must not be quoted'
assert '\ncreated_at: "' not in head, 'created_at must not be quoted'
assert body.startswith('\n\n'), 'blank line required after closing ---'
print('OK')
PY
```

如果本地装了 Ruby/Jekyll，可运行：

```bash
bundle exec jekyll build
```

如果没有安装，不要为了单篇文章强行安装大量依赖；可以依赖 GitHub Actions 构建，但必须做线上验证。

---

### 5. 提交并推送

```bash
cd /home/user/hermes/mywiki

git config user.name "ThinkingBeing"
git config user.email "thinkingbeing@gmail.com"

git add _posts skills
git commit -m "feat: 添加文章解读 - <标题>"
git push origin main
```

如果只是更新这个 Skill，可使用：

```bash
git add skills/article-deep-analysis/SKILL.md
git commit -m "docs: add article deep analysis skill"
git push origin main
```

---

### 6. GitHub Pages 验证

推送后必须验证，不要只停在 `git push` 成功。

1. 查看 GitHub Actions：

```bash
gh run list --repo ThinkingBeing/mywiki --limit 5
```

或访问：

```text
https://github.com/ThinkingBeing/mywiki/actions
```

2. 等待 Pages 构建完成。

3. 检查首页是否出现文章：

```bash
curl -sL "https://thinkingbeing.github.io/mywiki/" | grep -i "标题关键词"
```

4. 打开文章 URL 验证不是 404。

常见文章 URL 形式：

```text
https://thinkingbeing.github.io/mywiki/YYYY/MM/DD/slug.html
```

如果首页或文章不存在：

- 检查 `_posts` 路径是否正确。
- 检查文件名是否符合 `YYYY-MM-DD-slug.md`。
- 检查 front matter 是否有 `layout: post`。
- 检查 `date` 和 `created_at` 是否被错误加引号。
- 检查 `---` 后是否有空行。
- 查看 GitHub Actions 构建日志。
- 必要时推送空提交触发重建：

```bash
git commit --allow-empty -m "chore: trigger pages rebuild"
git push origin main
```

---

## 返回给用户的格式

完成后回复：

```markdown
已完成：

- 解读文章：<标题>
- 已提交到：ThinkingBeing/mywiki
- GitHub Pages：<文章访问链接>
- 验证状态：首页已出现 / 文章页可访问
```

如果原文无法访问但使用了替代来源，必须说明：

```markdown
注意：原始链接无法直接访问，我改用 <替代来源> 作为依据完成解读。
```

---

## 常见坑

1. **把文章放错仓库**  
   目标是 `ThinkingBeing/mywiki` 仓库，不是 GitHub Wiki 独立仓库。

2. **没有 `created_at`**  
   首页按 `created_at` 排序。缺失时排序可能不符合预期。

3. **front matter 格式错**  
   Jekyll 对 front matter 很敏感。特别注意 `date` 不要加引号，结束 `---` 后要有空行。

4. **只 push 不验证**  
   必须访问 GitHub Pages 或用 `curl` 验证文章已经发布。

5. **把摘要当解读**  
   用户要的是深度解读，必须包含背景、观点、论据、评价和启发。

6. **原文不可访问时硬编**  
   不要编造。找替代来源，并在文章和最终回复中注明。
