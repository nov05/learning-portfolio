市面上的通用大模型不适合作为系统性的 Startup Discovery 工具。ChatGPT、Claude、Gemini、Perplexity 可以作为搜索入口、候选扩展和信息整理工具，但它们无法保证市场覆盖率。具体问题包括搜索结果受搜索引擎索引限制、单次检索结果数量有限、网页抓取深度有限、动态页面和 LinkedIn 等受限数据源难以访问、数据库信息存在滞后、同一家公司在不同来源中的名称和信息不一致。对于高度具体的筛选条件，返回 0 个结果不能解释为“市场上没有符合条件的 Startup”，只能解释为“当前 Discovery Pipeline 没有发现符合条件的 Startup”。

更可靠的架构应该采用 **Discovery → Crawling → Extraction → Agent Research → Verification → Filtering**。Discovery 层负责最大化 Recall，不让 LLM 直接承担完整的公司发现任务。数据源可以包括 Google/Bing 等搜索 API、Crunchbase、Dealroom、Tracxn、Wellfound、Product Hunt、G2、VC Portfolio、Accelerator Portfolio、招聘网站、行业目录、公司官网、融资新闻、政府注册信息等。不同来源产生 Candidate Set 后进行 Entity Resolution，将 “OpenAI, Inc.”、“OpenAI” 等不同名称归并到同一个公司实体，同时记录 domain、LinkedIn URL、数据库 ID、公司名称、创始人等唯一标识。

Crawler 层负责补充搜索结果无法直接提供的信息。不要设计成简单的 URL crawler，而应该针对 Startup Research 建立 **targeted crawler**：输入公司 domain 后抓取 `/about`、`/careers`、`/jobs`、`/pricing`、`/customers`、`/blog`、`/press` 等页面，提取成立年份、团队规模、产品类别、客户、招聘岗位、融资事件、地理位置等字段。对于 JS-rendered 页面使用 browser automation；对于重复访问页面使用缓存；对 robots.txt、rate limits、authentication 和网站 ToS 设置约束。Crawler 的输出不是原始 HTML，而应该进入统一的 Evidence Store。

Agent 层负责针对每个 Candidate 动态决定下一步研究路径。例如公司官网没有成立年份，Agent 查询融资数据库和新闻；官网显示 25 人，但 LinkedIn 显示 45 人，Agent 标记冲突并寻找第三个来源；公司符合行业条件，但无法确认是否 B2B，Agent 搜索产品页面、客户案例和定价页面。Agent 应该使用工具调用，而不是依赖自身知识直接回答。每个结论都保存 `claim → evidence → source → timestamp → confidence`，避免最后只留下一个无法审计的 “符合条件”。

筛选逻辑应该与 LLM 判断分离。硬条件使用 deterministic rule engine，例如 `founded_year >= 2022`、`employee_count BETWEEN 10 AND 50`、`country = US`。语义条件才交给 LLM，例如“是否属于 AI-native sales infrastructure”“是否真正服务 B2B customers”。最终输出可以采用：

```text
Company
├── Structured attributes
├── Evidence
│   ├── Source
│   ├── URL
│   ├── Extracted claim
│   └── Timestamp
├── Must-have checks
├── Optional criteria
├── Confidence
└── Research status
```

Agent 的核心价值不是“比 ChatGPT 知道更多公司”，而是能够根据当前证据自动决定 **下一步查什么、查哪里、什么时候停止**。因此可以设置 Research Policy，例如每家公司最多查询 8 个来源；核心条件必须有至少 2 个独立来源验证；来源冲突时进入 manual review；连续 3 个高质量来源都无法验证某字段时标记为 `unknown`，而不是让模型猜测。

整个系统应该优化 **Recall → Precision**，而不是一开始追求精准结果。第一阶段尽可能获得 500–5,000 个候选公司，第二阶段进行去重和基础字段抽取，第三阶段根据 Must-have criteria 降到 100–500 家，第四阶段使用 Agent 深度研究降到 20–100 家，最后人工验证 Top 10–30 家。这样即使某一个搜索引擎、数据库或 LLM 漏掉部分公司，也不会直接导致整个研究结果失效。

因此，“AI 返回 0 个 Startup”这个情况不应该通过简单地从 ChatGPT 切换到 Claude、Gemini、Perplexity 来解决。更合理的 fallback 是扩大 **Discovery Sources**、改变 **Search Strategy**、增加 **Crawler Coverage**、增加 **Candidate Pool**，再让 Agent 对候选公司进行深度验证。不同 LLM 可以作为不同的 Research/Verification Model，但不应该被当作四个独立的 Startup Database。最终系统的目标不是得到“AI 推荐的 10 家公司”，而是建立一个能够解释 **为什么这些公司被发现、为什么其他公司被排除、每个结论来自什么证据、信息截至什么时候有效** 的可审计 Startup Research Pipeline。

<br><br><br>

---  

这个案例值得保留，而且很适合说明为什么 **Discovery Pool 不能只依赖当前数据库状态或单次 AI 搜索**。

OpenArt 可以作为一个 **“已超出目标融资阶段，但仍然是高质量补充样本”**。目前公开信息显示，公司成立于 2022 年，最初由前 Google 员工 Coco Mao 和 John Qiao 创办，位于旧金山/Redwood City；2026 年 1 月完成 **$30M Series A，由 Canaan Partners 领投**，Basis Set 和 DCM 等参与。Canaan 对外披露的融资时点，公司约 20 人、8M MAU、$70M+ ARR；OpenArt 自己后续发布的公司故事也披露了 2025 年 ARR 从约 $10.5M 增长到 $70M+。([OpenArt][1])

这个案例尤其有价值，因为它说明了**同一家公司的公开信息会随着时间发生剧烈变化**。2025 年 8 月 TechCrunch 报道时，OpenArt 对外披露的是累计融资约 $5M、正向现金流、预计 ARR 超过 $20M；到 2026 年 1 月已经完成 $30M Series A。([TechCrunch][2]) 如果研究系统只依赖某一个数据库，很可能得到过时的融资状态；如果只看最新数据库，又会丢失公司在早期阶段符合筛选条件时的重要信号。

因此在系统里应该把 **company state 做成时间序列，而不是一个静态字段**：

```text
OpenArt

2022
Founded

2023
Seed / early funding

2024
~$10.5M revenue
Small team

May 2025
~$10M ARR
~8 employees

Aug 2025
> $20M ARR trajectory
~$5M total funding

Jan 2026
$30M Series A
~20 employees
$70M+ ARR
8M MAU
```

([Basis Set][3])

这直接影响你的 MVP 数据模型。不要只存：

```text
company.funding_stage = Series A
company.employee_count = 20
```

应该至少保存：

```text
company
    ↓
claims
    ↓
claim_history
    ↓
source
    ↓
retrieved_at
```

例如：

```text
claim:
    field = funding_stage
    value = Seed
    observed_at = 2025-08-08
    source = TechCrunch

claim:
    field = funding_stage
    value = Series A
    observed_at = 2026-01-30
    source = Canaan

claim:
    field = employee_count
    value = 8
    observed_at = 2025-05
    source = Basis Set

claim:
    field = employee_count
    value = 20
    observed_at = 2026-01-30
    source = Canaan
```

这样你的筛选条件可以变成：

> **Find companies that matched criteria at any point during the target period**

而不是：

> **Find companies that match criteria right now**

这是 Startup Research 系统里非常重要的区别。

OpenArt 还暴露了另一个 Discovery 信号：**融资不是唯一的搜索入口**。如果从“AI creative platform / AI image generation / AI storytelling / generative media”这些产品类别、创始人背景、VC portfolio、招聘增长、ARR 增长等方向反向搜索，都可能发现 OpenArt。Canaan 和 Basis Set 的 portfolio 页面实际上就是非常好的 Discovery Source。Basis Set 甚至记录了 OpenArt 从 2025 年约 8 人、约 $10M ARR 到 2026 年 Series A 的增长过程。([Basis Set][3])

所以这个样本建议在 MVP 中标记成：

```text
OpenArt
────────────────────────
Discovery:        Valid
Founded:          2022
Location:         US
Sector:           AI / Generative Media
Stage:            Series A
Latest Funding:  $30M
Latest Funding:  2026-01-30
Employees:        ~20 at Series A
ARR:              $70M+
MAU:              8M+

Original target:
Early-stage startup
        ↓
No longer qualifies

Research value:
★★★★★
```

尤其不要因为它现在已经 Series A 就从数据库里删除。**应该保留为 historical candidate，并记录 `qualified_at`、`disqualified_at` 和 `disqualification_reason`。** 以后如果研究目标改成“过去 24 个月内曾经处于 Seed / pre-Series-A 阶段、且后来表现优秀的公司”，OpenArt 就会从“排除项”变成非常有价值的样本。

这也进一步支持前面那个架构：**Candidate Pool 应该是长期积累的 Research Dataset，而不是一次搜索的临时结果。**

[1]: https://openart.ai/about?utm_source=chatgpt.com "About"
[2]: https://techcrunch.com/2025/08/08/former-googlers-ai-startup-openart-now-creates-brainrot-videos-in-just-one-click/?utm_source=chatgpt.com "Former Googlers' AI startup OpenArt now creates ‘brain rot’ videos in just one click | TechCrunch"
[3]: https://www.basisset.com/insights/openarts-series-a-and-the-acceleration-of-infrastructure-first-consumer?utm_source=chatgpt.com "OpenArt’s Series A and the Acceleration of Infrastructure-First Consumer"

