# 英语课程内容生成规则

这份规则用于把小学英语课本原文生成成可练习的课程内容。

核心方向：**课文是源数据，课程是生成结果**。以后优先维护课文原文，再由规则或工具生成拆句练习、对话练习、翻译、提示、词块和单词信息。

核心方法：**递进链原则**——将每个句子拆成一条从小片段逐步扩展到完整句的递进链，辅助学生高效背课文。

## 0. 最高优先级

执行任何课程内容修改前，先确认这 4 条：

- `## 对话列表` 是课本原文，默认绝不修改。
- 生成课程时，只能基于 `## 对话列表` 的原文内容。
- 不允许为了拆句方便改写大小写、标点、缩写、角色名、编号或句子顺序。
- 只有用户明确说“修改课文原文”时，才允许改 `## 对话列表`。

## 1. 推荐工作流

按下面顺序执行，不要跳步：

1. 读取课文源文件。
2. 固定 `## 对话列表` 为基准。
3. 读取当前单元词表（单元级 `wordlist.md`，或同目录 `semester2-wordlists.md`）。
4. 将每条对话先拆成一个个完整句子。
5. 对每个完整句子先做句内拆解：短语、单词、剩余部分、完整句。
6. 为完整句生成中文、语法、提示、词块。
7. 为单词和短语补全音标、词性、意思。
8. 生成课程结果。
9. 最后校验：课文原文不变、编号正确、翻译不是单词级误匹配。

## 2. 文件职责

### 2.1 课文源

课文源优先保存人工维护内容：

- `## 课程信息`
- `## 对话列表`
- 可选 `## 单词表`
- 可选 `## 翻译覆盖`
- 可选 `## 生成配置`

课文源可以包含 `## 拆解列表`，用于人工校对和迁移参考。长期目标是把拆解内容迁移到结构化 JSON 中，但当前阶段两者共存。

课文源可以包含 `## 图片素材`，用于人工指定单元配图和出题意图。图片素材只描述图片和它关联的课文内容，不替代 `## 对话列表`，也不能反向修改课文原文。

### 2.2 生成课程

生成课程负责保存程序可直接使用的内容：

- 对话练习列表
- 拆句练习列表
- 阅读理解图片题列表
- 中文翻译
- 语法说明
- 提示
- 词块
- 单词、音标、词性、意思

生成课程可以是 JSON、KVON 或其他结构化格式。生成课程可以被缓存、覆盖、重建，但不能反向改写课文源原文。

### 2.3 旧版 markdown 兼容

当前格式中 `## 拆解列表` 既可以作为人工校对参考，也可以作为生成 JSON 课程的 seed 数据。

新生成规则中，`## 拆解列表` 不再是必须手写的长期维护内容，优先由工具从对话列表自动生成。

### 2.4 manifest v2 契约

每个可安装的英语课程包都必须使用 v2 领域包裹。`manifest.json` 是课程包的入口，生成或修改课程时必须同时校验它：

- 外层必须包含 `schemaVersion: "2.0"`、`id`、`name`、`description`、`subject: "english"`、`publisher`、`generation`；`cover` 可选。
- 外层必须包含 `domain`，且 `domain.key` 固定为 `english`；`domain.planning` 和 `domain.package` 必须是对象。
- `domain.planning` 保存英语排课信息：`audience`、`grade` / `semester` / `unit`、`tags`、`language`、`wordlist` / `vocabulary`、`languageLevel`、`phonicsStage`、`courseType` 和 `growth`。不适用的字段写 `null`，不要移到外层。
- `domain.package` 保存执行数据：`source { type, path }` 和 `activities[]`。源文件与活动数据路径都必须是相对当前课程目录的路径，并且目标文件真实存在。
- 每个 activity 必须包含 `type`、`name`、`data`、`enabled`。基础活动类型包括 `sentence-practice`、`conversation-practice`、`phonics-practice`、`listening-practice`、`reading-qa`、`speaking-practice`、`dictation-practice`；成长活动类型包括 `word-family-practice`、`pattern-transfer-practice`、`context-output-practice`、`mistake-review-practice`。
- v1 的顶层 `language`、`targetAge`、`grade`、`semester`、`unit`、`tags`、`source`、`supportedActivities`、`audience` 不得继续使用；它们必须迁入 `domain.planning` 或 `domain.package`。

manifest 迁移不能只改版本号。每次生成后都要检查：`domain.package.source.path` 存在；每个 activity 的 `data` 存在；`growthAvailable` 为 `false` 时不注册成长 activity；所有 JSON 文件可解析。未知 activity type 不得被悄悄改名或替换成其他练习。

### 2.5 生成文件与 manifest 的对应关系

- 生成 JSON 的 `manifestId` 必须与当前 `manifest.json` 的 `id` 一致。
- 普通活动的 `activityType` 必须与 `domain.package.activities[].type` 一致；`growth-content.json` 可以被多个已声明的成长活动共享，但必须同时提供这些活动所需的 `wordFamilies`、`patterns` 或 `transferPrompts` 数据。
- `sourceHash`、`generatorVersion`、`generatedAt` 用于追踪生成来源；重新生成内容时更新生成信息，不改写课文源文件。
- 活动缺少数据、路径越界、JSON 损坏或 manifestId 不匹配时，课程包视为不完整，应修复或移除对应 activity 声明，不能生成空占位活动。

## 3. 对话列表规则

处理 `## 对话列表` 时：

- 编号必须保留。
- 角色名必须保留。
- 英文原文必须保留。
- 旁白不能擅自改成角色对话。
- 角色对话不能擅自改成旁白。
- 一条对话中有多个英文句子时，不要改成多条对话。

示例：

```md
3. Su Hai: You can take the metro. You can get on the metro at Park Station and get off at City Library Station.
```

这里第 `3` 条仍然是一条对话，只是在生成课程时拆成多个句子。

## 4. 句子拆分流程

对每条对话按以下流程处理：

1. 去掉角色前缀，只处理英文正文。
2. 按句号 `.`、问号 `?`、感叹号 `!` 拆成独立句子。
3. 保留每个句子的原标点。
4. 保留缩写，例如 `There's`、`I'm`、`don't`。
5. 每个句子独立生成拆解内容。

示例：

```text
Hello! Nice to meet you.
```

拆为：

```text
Hello!
Nice to meet you.
```

## 5. 每个句子的生成顺序

每个句子都按这个顺序生成：

1. 先把句子拆成一条完整、连续、不重叠的片段链。
2. 片段链中的每一段都必须来自原句的连续文本。
3. 片段链拼起来后，必须正好还原整句。
4. 每个片段在当前句子里只出现一次。
5. 最后生成完整句拆解。

注意：

- 先看句子本身怎么拆，不是先看 wordlist。
- 拆句的目标是形成一条"从片段到整句"的背诵链，不是把句子切成一堆重叠小题。
- **递进链原则（最高优先级）**：片段链中，每个片段必须**紧跟在包含它的最小更大片段之前**。这不是只针对单词的规则，而是适用于所有片段的根本原则。
  - 例如 `very much` 被包含在 `I like it very much,` 中，所以 `very much` 必须紧跟在 `I like it very much,` 之前。
  - 例如 `far from` 被包含在 `but it's far from` 中，所以 `far from` 必须紧跟在 `but it's far from` 之前。
  - 例如 `school` 被包含在 `but it's far from school.` 中，所以 `school` 必须紧跟在 `but it's far from school.` 之前。
  - "单词紧贴在包含它的短语之前"只是递进链原则的一个特例。
- 短语优先于单词，例如优先 `a party`、`the prince's house`，而不是先拆 `party`、`prince`。
- 但 wordlist 单词如果需要单独出题，必须放在该单词所在短语**之前**（这是递进链原则的自然结果）。
- 功能词按类型区别对待（详见 §8）：
  - **介词**（`at`、`to`、`from`、`in`、`on`、`before` 等）：当它连接两个不同语义块时应**单独成片段**，保证背诵链的小步推进。
  - **纯连接词**（`and`、`but`、`or`、`so`）：默认不单独拆，除非它是该句语法结构的关键转折/连接点。
  - **冠词**（`a`、`an`、`the`）：不单独拆，附在相邻片段中。
  - **be 动词和助动词**（`is`、`are`、`am`、`do`、`does`）：不单独拆，附在相邻片段中。
  - **代词**（`I`、`you`、`he`、`she`、`it`）：不单独拆，附在相邻片段中。
- `wordlist` 只用于补单词信息和确认重点，不再强制把命中单词单独再出一题。
- 完整句必须放在这个句子的最后。
- 处理完当前句子后，才能开始下一句。
- 不要把下一句的短语提前放到当前句子前面。

示例 1：

```text
There is a party at the prince's house.
```

正确拆法：

```text
There is | a party | at | prince | the prince's house | There is a party at the prince's house.
```

错误拆法：

```text
prince | There is | a party | at | the prince's house | There is a party at the prince's house.
```

原因：

- `prince` 是 wordlist 单词，属于短语 `the prince's house`，必须紧贴在 `the prince's house` **正前方**（递进链原则）。
- 不能把单词提到整个句子的最前面，打断自然语序。

示例 2：

```text
I like it very much, but it's far from school.
```

正确拆法：

```text
very much | I like it very much, | far from | but it's far from | school | but it's far from school. | I like it very much, but it's far from school.
```

错误拆法：

```text
very much | far from | school | I like it very much, | but it's far from | but it's far from school. | I like it very much, but it's far from school.
```

原因：

- `very much` 被包含在 `I like it very much,` 中，所以 `very much` 必须紧跟 `I like it very much,`。
- `far from` 被包含在 `but it's far from` 中，所以 `far from` 必须紧跟 `but it's far from`。
- `school` 被包含在 `but it's far from school.` 中，所以 `school` 必须紧跟 `but it's far from school.`。
- 错误拆法把所有小片段堆在前面，违反了递进链原则。

## 6. 编号规则

如果仍然生成 markdown 拆解题，编号按下面规则：

- `## 对话列表` 的第 `n` 条，对应拆解题 `#### n.m. ...`。
- `n` 是对话编号。
- `m` 是该对话下生成项的顺序编号。
- 同一条对话包含多个句子时，仍然使用同一个 `n`，只递增 `m`。
- 拆解题标题必须来自第 `n` 条对话原文的连续片段。
- 拆解题不需要角色字段。

示例：

```md
#### 3.1. take the metro
#### 3.2. You can take the metro.
#### 3.3. get on
#### 3.4. You can get on the metro at Park Station.
```

## 7. Wordlist 规则

生成前必须读取词表。优先级：单元级 `wordlist.md` > 同目录 `semester2-wordlists.md` > 共享词库（§17）。

执行顺序：

1. 找到当前单元的词表（`wordlist.md` 中的 `## Unit n`，或 `semester2-wordlists.md` 中对应部分）。
2. **先按句子自然结构拆解**，形成连续的片段链（短语、介词等），不参考 wordlist。
3. **再参考 wordlist**，检查句子中哪些单词出现在词表中。
4. **调整拆分内容**：如果 wordlist 词未被任何片段单独覆盖且是核心学习点，补一道单独单词题。
5. **调整顺序**：如果补了单词题，将其**紧贴在包含该单词的短语正前方**（递进链原则），保持自然语序不变。
6. 命中 wordlist 的单词，要优先补齐音标、词性、意思和提示。
7. 如果 wordlist 词已自然包含在片段链中，不强制单独出题。
8. 句子中没有出现的 wordlist 词，不要强行塞进当前对话拆解。
9. wordlist 后面的索引区只能作为补充参考，不能覆盖当前单元词表。

短语示例：

- `far from`
- `on foot`
- `get on`
- `get off`
- `next to`

单词示例：

- `metro`
- `taxi`
- `bookshop`
- `station`

## 8. 重要内容拆解规则

先按句子自然结构拆，再用 wordlist 校验单词信息。

优先拆（常见类型，实际以句子自然结构为准）：

- 核心名词和名词短语
- 核心动词和动词短语
- 形容词
- 副词
- 地点
- 交通方式
- 方位表达
- 固定搭配
- 人名和地名

补充原则：

- 如果一个短语已经足够承载记忆点，就优先出短语题，不必先把里面每个单词都拆散。
- 句子的拆分必须是一条完整连续链，例如：
  - `There is`
  - `a party`
  - `at`
  - `prince`
  - `the prince's house`
  - `There is a party at the prince's house.`
- "剩余部分"必须仍然是原句中的连续片段，不能乱拼，也不能和前一个片段重叠。
- 句子里的连接词、介词、there be 结构等可以单独出片段，但每个结构片段在当前句子里只出现一次。
- `wordlist` 重点词如果已经自然包含在完整片段里，就不要再单独抽成重复题。
- 非 wordlist 的人名（如 `Cinderella`、`Yang Ling`）不单独出题，附在相邻片段中。

### 功能词拆分规则

功能词按类型区别对待：

**A. 介词 — 连接语义块时应单独拆**

当介词连接两个不同语义块（如"事件 + 地点"、"动作 + 方向"、"动作 + 时间"）时，应单独成片段，保证背诵链的小步推进。

- `at`：`a party | at | the prince's house`（事件 → 地点）
- `to`：`go | to | the party`（动作 → 目的地）
- `from`：`far | from | school`（状态 → 地点）
- `before`：`Come back | before | 12 o'clock`（动作 → 时间）
- `on`、`in`、`next to` 等：同理

**B. 纯连接词 — 默认不单独拆**

- `and`、`but`、`or`、`so`

硬性例外：**当连接词连接两个独立分句（各自有主语+谓语）时，必须单独拆出**。例如：
- `..., but Cinderella cannot go.` — `but` 两侧各有独立分句 → 必须拆
- `..., but it does not fit.` — 同上 → 必须拆
- `come and help me` — `and` 连接两个动词，不是独立分句 → 不拆

**C. 冠词 — 不单独拆**

- `a`、`an`、`the`

附在相邻片段中，例如 `a party`、`the prince's house`。

**D. be 动词和助动词 — 不单独拆**

- `is`、`are`、`am`、`do`、`does`

附在相邻片段中。例外：`There is` 作为 there be 句型结构可以单独成片段。

**E. 代词 — 不单独拆**

- `I`、`you`、`he`、`she`、`it`、`me`、`my`、`your`

附在相邻片段中。

**F. 完整句和短语的单词列表**

- **完整句拆解题必须列出该句每一个单词的详细信息**（单词、音标、词性、意思），不允许只列关键词而跳过功能词。
- 短语拆解题的单词列表应覆盖该短语所有单词。
- 短语片段可以只列核心单词的详细信息，但完整句不允许省略。

## 9. 重复规则

同一个单元内：

- wordlist 词在不同句子中重复出现时，可以重复生成，方便当前句子学习。
- 非 wordlist 的人名不单独出题，附在相邻片段中。
- 非 wordlist 的普通重要词，优先首次出现时生成独立拆解，后续不重复生成。
- 同一个拆解题内部，同一个单词只列一次。
- 同一个句子里，每个片段只能出现一次，不允许为了强调某个 wordlist 词再额外复制一个重叠片段。

## 10. 单词信息规则

每个单词或短语至少包含：

- `单词`
- `音标`
- `词性`
- `意思`

说明要求：

- 适合小学生理解。
- 音标尽量准确。
- 词性用常见说法，例如名词、动词、形容词、介词短语。
- 意思要结合当前课文语境。
- 人名必须完整，例如 `Yang Ling` 不要拆成只有 `Yang`。
- 地名必须完整，例如 `City Library Station` 不要只写 `Station`。

## 11. 中文翻译规则

中文翻译按优先级生成：

1. 人工翻译覆盖。
2. 已缓存的整句翻译。
3. 规则或 AI 生成的整句翻译。
4. 短语或单词翻译只用于短语题、单词题。

禁止：

- 不能把整句对话翻译成单个词。
- 不能把 `You can take the metro...` 这种长句只翻译成“搭乘”。
- 不能因为某个单词先匹配到，就用它的中文当整句中文。

## 12. 语法和提示规则

语法说明：

- 必须短。
- 必须精炼。
- 面向小学生。
- 不写长篇理论。
- 优先解释当前句子怎么表达意思。

提示：

- 用于帮助回忆，不直接泄露完整答案。
- 可以提示语义、场景、句型或关键词。
- 人名题可以提示“人名”。
- 地名题可以提示“地点”。

## 13. 词块规则

词块用于给小学生展示句子结构。

格式：

```md
- **词块**: 标签: 英文片段
```

要求：

- 完整句必须尽量生成词块。
- 单词题通常不需要词块。
- 短语题只有在有学习价值时才写词块。
- 英文片段必须来自当前题目原文，不能改写。
- 不强行套固定“主谓宾”。
- 标签要灵活、容易懂。

推荐标签：

- 时间
- 人物
- 主语
- 动作
- 正在做
- 宾语
- 地点
- 方式
- 位置
- 疑问词
- 称呼
- 连接
- 原因
- 转折

示例：

```md
#### 1.15. She is sweeping the floor.

- **中文**: 她正在扫地。
- **语法**: She + is sweeping + the floor
- **提示**: 描述正在做的家务
- **词块**: 主语: She
- **词块**: 正在做: is sweeping
- **词块**: 宾语: the floor
```

```md
#### 1.18. What is he doing now?

- **中文**: 他现在正在做什么？
- **语法**: What + is + he + doing + now
- **提示**: 询问某人现在正在做什么
- **词块**: 疑问词: What
- **词块**: 主语: he
- **词块**: 正在做: doing
- **词块**: 时间: now
```

## 14. 生成后检查清单

每次生成或修改课程内容后，按顺序检查：

- `## 对话列表` 是否和修改前完全一致。
- 对话编号是否没有新增、删除、错乱。
- 角色名是否没有被修改。
- 每条对话是否都能生成对应对话练习内容。
- 每个完整句是否都有整句中文翻译。
- 整句翻译是否没有退化成单个单词翻译。
- wordlist 中出现在句子里的重点单词是否已覆盖（单独题或包含在片段中均可）。
- 拆句顺序是否是"当前句单词题 -> 短语题 -> 当前完整句 -> 下一句 ..."。
- **递进链原则**：每个片段是否紧跟在包含它的最小更大片段之前（不能把所有小片段堆在前面）。
- 完整句词块是否存在且英文片段来自原文。
- 单词信息是否包含音标、词性、意思。
- 完整句拆解题是否列出了该句**每一个单词**的详细信息（不允许跳过功能词）。
- 如果有图片题，图片路径是否存在，且使用相对单元目录的路径。
- 图片题答案是否能追溯到 `## 对话列表` 原文或明确标注的目标句。
- 图片题干、图片文件名、`alt` 是否没有直接泄露答案。
- manifest 是否为 v2 领域包裹，并显式声明 `subject: "english"` 与 `domain.key: "english"`。
- `domain.package.activities` 中声明的数据文件是否全部存在。
- 只有存在 `growth-content.json` 且资源声明成长能力时，才注册成长 activity。

## 15. 图片素材和阅读理解图片题规则

当用户希望“配图片”“看图出题”“通过图片来出题”时，按本节规则处理。

### 15.1 图片放置规则

图片放在当前单元目录下：

```text
assets/images/
```

图片路径必须写成相对当前单元目录的路径，例如：

```text
assets/images/morning-cleaning-car.png
```

不要把本机绝对路径写进课程 JSON。不要把答案直接写进文件名，例如避免 `he-is-cleaning-the-car-answer.png`。

### 15.2 `## 图片素材` 格式

`textbook.md` 可以添加：

```md
## 图片素材

- id: morning-cleaning-car
  file: assets/images/morning-cleaning-car.png
  alt: 爸爸早上在门口洗车
  relatedLines: [1]
  targets:
    - My father is cleaning the car.
    - cleaning the car
```

字段含义：

- `id`：图片唯一标识，使用英文、数字和短横线。
- `file`：图片相对当前单元目录的路径。
- `alt`：图片内容描述，给老师、审核和无障碍场景使用，不直接给学生当答案。
- `relatedLines`：关联的课文编号，必须来自 `## 对话列表`。
- `targets`：希望图片触发练习的英文句子、短语或词，优先来自课文原文。

### 15.3 阅读理解图片题 JSON

图片题生成到：

```text
content/reading-qa.json
```

结构示例：

```json
{
  "manifestId": "grade5-semester2-unit1",
  "activityType": "reading-qa",
  "sourceHash": "",
  "generatorVersion": "learn-language-generator-2",
  "generatedAt": "2026-04-29",
  "images": [
    {
      "id": "morning-cleaning-car",
      "src": "assets/images/morning-cleaning-car.png",
      "alt": "爸爸早上在门口洗车",
      "relatedLines": [1]
    }
  ],
  "questions": [
    {
      "id": "img-u5-q1",
      "imageId": "morning-cleaning-car",
      "type": "multiple-choice",
      "prompt": "图中爸爸正在做什么？",
      "options": [
        "He is cleaning the car.",
        "He is cooking breakfast.",
        "He is sweeping the floor.",
        "He is sleeping."
      ],
      "correctIndex": 0,
      "answer": "He is cleaning the car.",
      "targetEnglish": "My father is cleaning the car.",
      "targetChinese": "我爸爸正在洗车。",
      "explanation": "图片对应课文表达：My father is cleaning the car.",
      "difficulty": 1
    }
  ]
}
```

### 15.4 图片题类型

优先支持这些类型：

- `multiple-choice`：看图选择正确句子，适合首次练习。
- `fill-blank`：看图补词，例如 `He is ____ the car.`，适合强化核心词。
- `speak`：看图说句子，适合口语和背诵检查。
- `order-words`：看图后给词排序，适合句型巩固。

### 15.5 出题原则

- 图片题的答案必须能追溯到课文原文，或者是课文原文的自然问答变体。
- 如果使用自然问答变体，必须在 `targetEnglish` 中保留对应课文原句。
- 低年级优先用选择题；掌握后再生成填空、排序、口语题。
- 同一张图片可以出多道题，但每道题只考一个清晰目标。
- 干扰项要来自同单元或相邻单元的相近表达，不能明显离谱。
- 题干不要直接复述答案；图片文件名和 `alt` 也不要直接泄露答案。
- 不要为了图片题改写 `## 对话列表` 原文。

### 15.6 manifest 注册

图片题不再注册独立活动；如果生成图片题，必须合并进 `content/reading-qa.json`，并在 `manifest.json` 的 `domain.package.activities` 中保留：

```json
{
  "type": "reading-qa",
  "name": "阅读理解",
  "data": "content/reading-qa.json",
  "enabled": true
}
```

## 16. 错误示例和正确示例

对话：

```text
I like it very much, but it's far from school.
```

错误：

```md
#### 4.1. school
#### 4.2. I like it very much, but it's far from
#### 4.3. I like it very much, but it's far from school.
```

问题：

- 没先按句子自然结构拆。
- 把单个 wordlist 单词直接抽出来后，没有保留有学习价值的短语和剩余部分。
- 孩子很难通过这种顺序形成背诵链。

正确：

```md
#### 4.1. very much
#### 4.2. I like it very much,
#### 4.3. far from
#### 4.4. but it's far from
#### 4.5. school
#### 4.6. but it's far from school.
#### 4.7. I like it very much, but it's far from school.
```

说明：

- `school` 虽然已包含在完整句片段中，但它是本单元 wordlist 核心词且是句末地点词，单独抽出有助于强化记忆。
- 每个片段都紧跟在包含它的最小更大片段之前（递进链原则）：
  - `very much` → `I like it very much,`
  - `far from` → `but it's far from`
  - `school` → `but it's far from school.`
  - `but it's far from school.` → 完整句

## 17. 一句话执行口令

当用户说"按规则重新生成某单元"时，默认执行：

1. 保护 `## 对话列表` 原文不变。
2. 读取当前单元 wordlist（优先 `wordlist.md`）。
3. 按对话编号逐条拆句。
4. 每句先按自然结构拆解片段链（不参考 wordlist），再参考 wordlist 调整拆分内容和补单词题，最后按**递进链原则**调整顺序（每个片段紧跟在包含它的最小更大片段之前）。
5. 为完整句补中文、语法、提示、词块。
6. 为词和短语补音标、词性、意思。
7. 生成后检查：课文原文不变、编号正确、翻译未退化、wordlist 词已覆盖、词块存在。
8. 生成的 JSON 文件放在本单元文件夹的 `content/` 目录下（如 `content/sentence-practice.json`），如果文件已存在则直接覆盖。

当用户说"给某单元配图出题"时，默认执行：

1. 保护 `## 对话列表` 原文不变。
2. 读取 `## 图片素材` 和 `assets/images/`。
3. 校验图片路径存在且是相对单元目录路径。
4. 根据 `relatedLines` 和 `targets` 生成 `content/reading-qa.json`。
5. 在 `manifest.json` 中只保留 `reading-qa` 活动，不再添加独立图片题活动。
6. 生成后检查：题目答案可追溯、题干不泄露答案、干扰项合理、图片路径可用。

## 18. 教材掌握线与语言成长线

课程资源分成两条互不替代的线：

- **教材掌握线**使用课本原句、核心短语和单词，所有内容必须可追溯到 `## 对话列表` 或当前单元词表。
- **语言成长线**只在资源明确声明并且学习记录稳定后出现，用于词族、句型迁移和情境表达；它不能改写、覆盖或替代教材原文练习。

英语课程 manifest 使用 v2 领域包裹。外层包含市场元数据与 `subject: "english"`；英语字段只放在 `domain` 内：

```json
{
  "schemaVersion": "2.0",
  "id": "grade5-semester2-unit5",
  "subject": "english",
  "domain": {
    "key": "english",
    "planning": {
      "audience": { "stage": "primary" },
      "grade": 5,
      "semester": 2,
      "unit": 5,
      "growth": {
        "growthAvailable": true,
        "growthTypes": ["word-family", "pattern-transfer"],
        "foundationActivityTypes": ["sentence-practice"],
        "growthActivityTypes": ["word-family-practice", "pattern-transfer-practice"]
      }
    },
    "package": {
      "source": { "type": "textbook-markdown", "path": "textbook.md" },
      "activities": []
    }
  }
}
```

上例只突出 v2 的领域边界和成长声明；实际课程仍须按 §2.4 补齐外层元数据、英语 planning 字段、源文件和可执行 activities，不能把这个缩略示例当成可直接安装的最小文件。

当 `growthAvailable` 为真，成长活动可引用 `content/growth-content.json`。文件中的内容必须遵循：

1. `wordFamilies` 的根词和至少一个词形来自当前单元原文或词表；派生内容必须标记 `isSourceText: false`。
2. 每个词族节点保留词、作用、例句、中文、难度与来源标记；不以“为了扩展”修改原句。
3. `patterns` 与 `transferPrompts` 从本单元已出现的表达出发；迁移句必须显示与原句的关联。
4. 没有成长数据时不得声明对应成长 activity；UI 不显示词族树，也不把成长任务替换成其他内容。
