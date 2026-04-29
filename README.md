# 英语课程内容制作指南

## 概述

本项目的核心流程是：**准备课文源文件 → 按规则生成结构化课程内容**。

课文源文件由人工维护，课程内容由规则或工具自动生成。生成结果为 JSON 格式，可直接供应用程序使用。

## 文件结构

一个完整的课程单元文件夹如下（文件夹名称可自由命名）：

```
my-unit-folder/
├── textbook.md                          ← 必需：课文源文件
├── wordlist.md                          ← 可选：单元单词表
├── manifest.json                        ← 生成：单元元数据
├── assets/                              ← 可选：单元图片、音频等素材
│   └── images/
│       └── cleaning-car.png
└── content/
    ├── sentence-practice.json           ← 生成：拆句练习
    ├── conversation-practice.json       ← 生成：对话练习
    └── image-qa.json                    ← 可选：看图问答
```

## 第一步：准备课文源文件

### 创建文件夹

创建一个文件夹，名称随意，例如 `cinderella/` 或 `unit1/`。

### 编写 textbook.md

这是唯一必需的源文件。模板如下：

```markdown
# 五年级第一单元 - Cinderella

学习关于灰姑娘故事的表达

## 课程信息

- **难度**: 初级
- **级别**: 1

## Story time

1. There is a party at the prince's house, but Cinderella cannot go.
2. Sister: Cinderella, come and help me!
3. A fairy comes.
```

**Story time 规则：**

- 标题固定为 `## Story time`
- 每行一条对话，以数字编号开头
- 对话格式：`编号. 角色名: 英文内容`
- 旁白格式：`编号. 英文内容`（无角色名）
- 一条对话中可以有多个句子（用 `.` `?` `!` 分隔），不要拆成多条
- 编号、角色名、英文原文、标点、缩写都必须保留原样

**可选章节：**

```markdown
## 翻译覆盖        ← 可选：人工指定的中文翻译
## 图片素材        ← 可选：人工指定的配图和出题意图
## 生成配置        ← 可选：生成参数
```

### 配置图片素材（可选）

如果希望通过图片出题，在单元目录下创建 `assets/images/`，把图片放进去，并在 `textbook.md` 里添加 `## 图片素材`：

```markdown
## 图片素材

- id: morning-cleaning-car
  file: assets/images/morning-cleaning-car.png
  alt: 爸爸早上在门口洗车
  relatedLines: [1]
  targets:
    - My father is cleaning the car.
    - cleaning the car
```

字段说明：

| 字段 | 说明 |
|---|---|
| `id` | 图片唯一标识，建议用英文短横线 |
| `file` | 图片相对当前单元目录的路径 |
| `alt` | 给老师和无障碍场景看的图片描述，不直接给学生当答案 |
| `relatedLines` | 图片关联的课文编号 |
| `targets` | 希望图片触发练习的英文句子、短语或词 |

图片出题的目标是让学生先观察场景，再回忆课文表达。不要把答案写进图片文件名、题干或 `alt` 中。

### 编写 wordlist.md（可选）

如果单元有重点单词表，创建 `wordlist.md`：

```markdown
# Unit 1 Wordlist

* prince 王子
* fairy 仙女
* why 为什么
* because 因为
* clothes 衣服
* put on 穿上
* try on 试穿
* fit 合适，合身
```

格式：每行一个词或短语，用 `*` 开头，英文和中文之间用空格分隔。

词表的作用：
- 标记哪些词是本单元的学习重点
- 生成时为这些词补齐音标、词性、意思
- 已包含在片段链中的词不强制单独出题

## 第二步：交给 AI 生成课程内容

准备好 `textbook.md`（和可选的 `wordlist.md`）后，就可以交给 AI 自动生成课程内容了。

**推荐工具：TRAE SOLO**

目前大部分课程使用 TRAE SOLO 生成。操作步骤：

1. 打开 TRAE SOLO
2. 将课程文件夹（包含 `textbook.md`）作为工作目录
3. 上传 `learn-english-course-content-rules.md` 作为规则上下文
4. 告诉 AI："按规则生成课程内容"
5. AI 会自动完成拆句、生成 JSON、校验

AI 将按以下流程执行：

```
1. 读取 textbook.md，锁定 Story time 原文（不可修改）
2. 读取 wordlist.md（如果存在）
3. 逐条对话拆句：
   a. 去掉角色前缀，按 .?! 拆成独立句子
   b. 每句按自然结构拆成连续、不重叠的片段链
   c. 为每个片段生成中文、语法、提示、单词信息
   d. 完整句放在最后，必须带词块
   e. 完整句必须列出每一个单词的详细信息
4. 生成 JSON 文件
5. 校验
```

### 拆句核心规则

**片段链要求：**
- 片段必须来自原句的连续文本
- 片段拼起来必须正好还原整句
- 每个片段在当前句子里只出现一次
- 短语优先于单词（如 `a party` 优于 `party`）

**功能词处理：**

| 类型 | 规则 | 示例 |
|---|---|---|
| 介词（at/to/from/before） | 连接语义块时**单独拆** | `a party \| at \| the prince's house` |
| 连接词（and/but/or/so） | 默认**不单独拆** | 附在相邻片段中 |
| 冠词（a/an/the） | **不单独拆** | `a party`、`the prince's house` |
| be动词/助动词（is/are/do） | **不单独拆** | 附在相邻片段中 |
| 代词（I/you/he/she/it） | **不单独拆** | 附在相邻片段中 |

**完整句要求：**
- 必须列出该句**每一个单词**的详细信息（音标、词性、意思）
- 必须带词块（structureBlocks）
- 放在当前句子的最后

### 拆句示例

原文：`There is a party at the prince's house.`

```
片段链：There is → a party → at → the prince's house → 完整句
```

## 第三步：生成输出文件

### manifest.json

单元元数据文件，模板如下：

```json
{
  "schemaVersion": "1.0",
  "id": "grade5-semester2-unit1",
  "name": "五年级下册 Unit 1 - Cinderella",
  "description": "学习关于灰姑娘故事的表达",
  "publisher": {
    "name": "Learn with daddy's love"
  },
  "language": "en",
  "targetAge": "primary-grade-5",
  "grade": 5,
  "semester": 2,
  "unit": 1,
  "tags": ["灰姑娘", "故事", "童话"],
  "source": {
    "type": "textbook-markdown",
    "path": "textbook.md",
    "wordlist": "wordlist.md"
  },
  "supportedActivities": [
    {
      "type": "sentence-practice",
      "name": "拆句练习",
      "data": "content/sentence-practice.json",
      "enabled": true
    },
    {
      "type": "conversation-practice",
      "name": "对话练习",
      "data": "content/conversation-practice.json",
      "enabled": true
    },
    {
      "type": "image-qa",
      "name": "看图问答",
      "data": "content/image-qa.json",
      "enabled": true
    }
  ],
  "generation": {
    "sourceHash": "",
    "generatorVersion": "learn-language-generator-2",
    "generatedAt": "2026-04-18"
  }
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| `id` | 唯一标识，格式 `grade{年级}-semester{学期}-unit{单元号}` |
| `name` | 显示名称，中文 |
| `grade` / `semester` / `unit` | 年级、学期（1=上册 2=下册）、单元号 |
| `tags` | 标签数组，用于分类和搜索 |
| `supportedActivities` | 支持的练习类型列表 |

### content/sentence-practice.json

拆句练习数据，结构如下：

```json
{
  "manifestId": "grade5-semester2-unit1",
  "activityType": "sentence-practice",
  "sections": [
    {
      "title": "Breakdown",
      "sentences": [
        {
          "sequenceNumber": 1,
          "segmentNumber": 1,
          "english": "There is",
          "speaker": "narrator",
          "chinese": "有",
          "difficulty": 1,
          "hint": "存在片段",
          "grammarAnalysis": "There be 句型",
          "words": [
            {
              "text": "There",
              "phonetic": "/ðeə(r)/",
              "partOfSpeech": "副词",
              "meaning": "引导 there be 句型"
            }
          ],
          "structureBlocks": [],
          "category": "phrase",
          "id": "1-1-1"
        }
      ]
    }
  ]
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| `sequenceNumber` | 对话编号（对应 textbook.md 中的编号） |
| `segmentNumber` | 片段编号（同一对话内递增） |
| `english` | 英文片段（来自原文的连续文本） |
| `speaker` | 说话者（角色名或 `narrator`） |
| `chinese` | 中文翻译 |
| `difficulty` | 难度级别 |
| `hint` | 学习提示（帮助回忆，不泄露答案） |
| `grammarAnalysis` | 语法说明（短、精炼、面向小学生） |
| `words` | 单词数组，每个单词含 `text`/`phonetic`/`partOfSpeech`/`meaning` |
| `structureBlocks` | 词块数组，完整句必须有，格式 `{"label": "标签", "text": "英文片段"}` |
| `category` | 类型：`phrase`（短语）/ `word`（单词）/ `sentence`（完整句） |

**category 规则：**
- `phrase`：短语片段，words 列核心单词即可
- `word`：单个单词片段
- `sentence`：完整句，**必须**列出每一个单词的详细信息，**必须**有 structureBlocks

### content/conversation-practice.json

对话练习数据，结构如下：

```json
{
  "manifestId": "grade5-semester2-unit1",
  "activityType": "conversation-practice",
  "sections": [
    {
      "title": "Story time",
      "lines": [
        {
          "sequenceNumber": 1,
          "speaker": "narrator",
          "english": "There is a party at the prince's house, but Cinderella cannot go.",
          "chinese": "王子家有一个派对，但灰姑娘不能去。"
        }
      ]
    }
  ]
}
```

对话练习直接来自 `## Story time` 原文，不做任何修改。

### content/image-qa.json

看图问答数据，结构如下：

```json
{
  "manifestId": "grade5-semester2-unit1",
  "activityType": "image-qa",
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

图片题推荐类型：

| 类型 | 用途 |
|---|---|
| `multiple-choice` | 看图选择正确句子，适合低年级和首次练习 |
| `fill-blank` | 看图补全关键词，如 `He is ____ the car.` |
| `speak` | 看图说句子，适合口语和背诵检查 |
| `order-words` | 看图后给词排序，适合句型巩固 |

生成图片题时，答案必须来自课文原文或课文原文的自然变体；如果是自然变体，要在 `targetEnglish` 里保留对应课文原句，方便追溯。

## 生成后检查清单

每次生成后，逐项检查：

- [ ] `## Story time` 原文完全一致（未改任何字符）
- [ ] 对话编号连续、无新增或删除
- [ ] 角色名未被修改
- [ ] 每个完整句都有整句中文翻译（不是单词翻译）
- [ ] 每个完整句都列出了**每一个单词**的详细信息
- [ ] 每个完整句都有 structureBlocks（词块）
- [ ] wordlist 中的重点词已覆盖（单独题或包含在片段中）
- [ ] 片段链连续、不重叠、拼起来还原整句
- [ ] 介词在连接语义块时已单独拆出
- [ ] 功能词（and/but/or/so/a/the）未单独拆出
- [ ] 图片路径存在且是相对单元目录的路径
- [ ] 图片题的答案能追溯到课文原文或明确的目标句
- [ ] 图片题干、文件名和 `alt` 没有直接泄露答案

## 快速开始

```
1. 创建文件夹
2. 编写 textbook.md（必须）
3. 编写 wordlist.md（可选）
4. 打开 TRAE SOLO，上传规则文件和课程文件夹
5. 发送提示词，等待生成
6. 检查生成结果
```

### 推荐提示词

准备好文件后，在 TRAE SOLO 中发送以下提示词：

**初次生成：**

```
请按照 learn-english-course-content-rules.md 的规则，读取 textbook.md（和 wordlist.md），生成 manifest.json 和 content/ 下的 JSON 文件。
```

**仅重新生成拆句练习（不改对话原文）：**

```
请按照规则重新生成 sentence-practice.json，对话原文保持不变。
```

**校验已有内容：**

```
请按照 learn-english-course-content-rules.md 的检查清单，校验当前单元的 JSON 文件是否合规。
```
