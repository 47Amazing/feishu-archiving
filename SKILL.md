---
name: feishu-archiving
description: Use when user wants to archive an article or URL to Feishu knowledge base. Triggers on keywords: 入库、归档、存进飞书、存飞书、知识整理, or when user sends a URL with intent to save/organize knowledge. Do NOT use for batch article fetching to ArticleInbox.
---

# Feishu Archiving

将文章归档入飞书知识库的 7 步工作流。

## 7 步执行顺序

**Step 1 — 抓正文**（使用 `web-access` skill）

| 来源 | 方式 |
|------|------|
| 普通文章、博客、文档 | Jina：`r.jina.ai/<url>` |
| 微信公众号、小红书等反爬平台 | CDP 浏览器直接访问 |
| 其他静态页面 | WebFetch |

内容仅保留在内存中，**不落盘 ArticleInbox**。

**Step 2 — 查重**
```bash
lark-cli docs +search --query "<文章标题关键词>"
```
确认飞书知识库中无重复文档再继续。

**Step 3 — 判断是否值得收录**

满足以下任一条才入库，否则告知用户并停止：
- 可复用方法论
- 稳定认知框架
- 长期参考价值

**Step 4 — 分类**

| 分类 | 适用内容 | wiki space_id |
|------|---------|--------------|
| AI 认知篇 | AI 原理、认知框架、思维模型 | `7619745043108088764` |
| AI 实践篇 | 工具使用、工程实践、操作方法 | `7620043164254014422` |
| Claude 与 Agent 工作流 | Claude Code、Agent 构建、Prompt 工程 | `7619761830163909585` |

完整决策树见 TOOLS.md：https://www.feishu.cn/wiki/TlJhwSe4biDCcJkcGyFcUNZinKa

**Step 5 — 创建知识库文档**
```bash
lark-cli docs +create \
  --wiki-space "<对应空间的 space_id>" \
  --title "<文档标题>" \
  --markdown "<Lark-flavored Markdown 正文>"
```
返回结果中记录 `node_token`，wiki URL 格式：`https://www.feishu.cn/wiki/<node_token>`

文档内容格式见下方「文档格式规则」。

**Step 6 — 更新精华文档**
```bash
lark-cli docs +update \
  --doc "<精华文档 doc_id>" \
  --mode append \
  --markdown "<追加内容>"
```

追加内容格式（必须遵守）：
```markdown
### [文档标题](https://www.feishu.cn/wiki/<node_token>)

1. 章节标题1（陈述句，取文档 ## 标题）
2. 章节标题2
3. 章节标题3

**直接记住：** 核心判断1 / 核心判断2 / 核心判断3

```

精华文档 doc_id 见下方「资源 ID」。

**Step 7 — 写入知识索引 bitable**
```bash
lark-cli base +record-upsert \
  --base-token "<base_token>" \
  --table-id "<table_id>" \
  --json '{"<文档标题字段>":"...","<日期字段>":<ms时间戳>,"<原文标题字段>":"...","<原文链接字段>":"...","<关键词字段>":"...","<打开字段>":"https://www.feishu.cn/wiki/<node_token>","<已放NotebookLM字段>":"否"}'
```
字段 ID 见下方「资源 ID」。日期字段传**毫秒时间戳**（Unix 秒 × 1000）。

---

## 文档格式规则

### 顶部 callout（严格执行）

每个字段独立一行，字段间空一行，原文链接写完整 URL：

```
原文标题：xxx

原文链接：https://完整URL（不要缩写）

来源：公众号名 / 平台名

关键词：xxx / xxx / xxx
```

### 正文结构

- `##` 分主题（2-4 个），`###` 分子主题，不用粗体代替标题
- 章节标题必须是**陈述句**，禁用反问句
  - ✅ 「AI 使用频率直接影响销售业绩」
  - ❌ 「为什么 AI 用得越多业绩越好？」
- 有编号列表/框架 → 用编号列表或表格，全部保留
- 有对比 → 用 grid 分栏
- 有推导链 → 单独一行（如「A → B → C」）
- 长段落（>2-3 句）拆分，段落间保留空行
- 并列判断块之间加 `---` 分隔线
- 禁止套话：「总结一下」「这节最该记住的话」等一律不写

---

## 资源 ID

### AI 认知篇

| 参数 | 值 |
|------|----|
| wiki space_id | `7619745043108088764` |
| 精华文档 doc_id | `VVU4dXFpOo7Pd5xcEYxcwFAsnCd` |
| bitable base_token | `Eua8bxN4WaDqBcs268gcLM8SnWh` |
| table_id | `tblevf0UmyKgHgJX` |
| 文档标题 | `fldjtFCPC1` |
| 日期（毫秒时间戳） | `fldKus7mj8` |
| 原文标题 | `fldEVXvlbh` |
| 原文链接 | `fldsbIJWI0` |
| 关键词 | `fld4VX6KM3` |
| 打开（wiki URL） | `fldYMsyVCx` |
| 已放NotebookLM | `fldNLL8AWp` |

精华文档：https://www.feishu.cn/wiki/WnQTwg4g0ieoUHk5yf8cGCN3nPq

### AI 实践篇

| 参数 | 值 |
|------|----|
| wiki space_id | `7620043164254014422` |
| 精华文档 doc_id | `ZzAhdevapoCj0jxAONOcKxobnGg` |
| bitable base_token | `JuuSbd5XFa1KzzsQMcVcM2UOndd` |
| table_id | `tblrDd4kpAuFQN2q` |
| 文档标题 | `fld0P1AMA3` |
| 日期（毫秒时间戳） | `fld2zO5l6u` |
| 原文标题 | `fldC2PRVPY` |
| 原文链接 | `fldLekKnpP` |
| 关键词 | `fld4VOaTCu` |
| 打开（wiki URL） | `fldrSyNAcq` |
| 已放NotebookLM | `flduV73SmH` |

精华文档：https://www.feishu.cn/wiki/Hc6pwhjdLiV0AZkIwTKc1vejndh

### Claude 与 Agent 工作流

| 参数 | 值 |
|------|----|
| wiki space_id | `7619761830163909585` |
| 精华文档 doc_id | 待建立 |
| bitable | 待建立 |

精华文档和 bitable 建立后补充此处。
