# code-walkthrough Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个对话式代码讲解/确认 skill,帮助用户理解 AI 生成的代码,并沉淀理解笔记与存疑清单。

**Architecture:** 单一 skill `code-walkthrough`:`SKILL.md` 是不认识任何具体技术栈的引擎(对话流程 + 确认机制 + 笔记规则);`references/packs/**` 是可插拔燃料(按探测到的技术栈按需加载,支持多 pack 叠加);理解笔记产出到用户项目内。首发只实现 Spring 一个 pack。

**Tech Stack:** Markdown(skill 内容)。无代码运行时;验证靠"干跑一遍讲解流程"与结构核对。

> 参考 spec:`docs/superpowers/specs/2026-05-26-code-walkthrough-design.md`

---

## File Structure

```
skills/code-walkthrough/
├── SKILL.md                              # 引擎
└── references/
    ├── note-template.md                  # 理解笔记 + 存疑清单模板
    └── packs/
        ├── _pack-format.md               # pack 编写规范(给未来扩展者看)
        ├── languages/   (空,占位说明)
        ├── frameworks/
        │   └── spring.md                 # 首发唯一 pack
        ├── systems/     (空,占位说明)
        └── projects/    (空,占位说明)
```

职责划分:
- `SKILL.md`:流程引擎,不含任何技术栈知识。
- `references/note-template.md`:两份产出物(理解笔记、存疑清单)的格式样板,引擎在沉淀时读取。
- `references/packs/_pack-format.md`:统一的 pack 结构说明,保证未来扩展一致。
- `references/packs/frameworks/spring.md`:首个 pack。

---

### Task 1: 搭建 skill 骨架与目录

**Files:**
- Create: `skills/code-walkthrough/references/packs/languages/.gitkeep`
- Create: `skills/code-walkthrough/references/packs/systems/.gitkeep`
- Create: `skills/code-walkthrough/references/packs/projects/.gitkeep`

- [ ] **Step 1: 创建目录占位**

```bash
mkdir -p skills/code-walkthrough/references/packs/{languages,frameworks,systems,projects}
touch skills/code-walkthrough/references/packs/languages/.gitkeep
touch skills/code-walkthrough/references/packs/systems/.gitkeep
touch skills/code-walkthrough/references/packs/projects/.gitkeep
```

- [ ] **Step 2: 验证目录结构**

Run: `find skills/code-walkthrough -type d`
Expected: 列出 `skills/code-walkthrough`、`.../references`、`.../references/packs` 及四个子目录。

- [ ] **Step 3: Commit**

```bash
git add skills/code-walkthrough
git commit -m "chore: scaffold code-walkthrough skill directories"
```

---

### Task 2: 编写 note-template.md

**Files:**
- Create: `skills/code-walkthrough/references/note-template.md`

- [ ] **Step 1: 写入模板内容**

文件内容(完整):

````markdown
# 产出物模板

引擎在 [沉淀] 阶段使用这两份模板。产出到**用户项目内** `docs/walkthrough/`,
不要写进 skill 目录。

---

## 1. 理解笔记(每模块一份)

路径:`docs/walkthrough/<模块名>.md`
规则:**增量更新**——同一模块二次讲解时合并字段,不整体覆盖;更新"最后更新"日期。

```markdown
# <模块名> 理解笔记
> 最后更新:YYYY-MM-DD | 技术栈:<探测到的栈,多个用逗号分隔>

## 一句话职责
<这个模块是干什么的>

## 对外接口 / 入口
<别人怎么用它,从哪进>

## 依赖
<它依赖谁、谁依赖它>

## 主链路
1. 入口 X → 调用 Y → 落到 Z
   - 关键实现:<用了什么策略,为什么>
   - ✅ 已确认意图:<用户确认过的设计预期>

## 扫盲记录
- <这次搞懂的陌生技术/模式,一句话留存>
```

---

## 2. 存疑清单(全局一份)

路径:`docs/walkthrough/_open-questions.md`
规则:每标记一个存疑点就**追加一行**;状态字段随后续讨论流转。
状态取值:`开放` / `已确认无碍` / `待追改` / `已解决`。
类型取值:`风险` / `待确认` / `不符常规` / `疑问`。

```markdown
# 存疑清单

| # | 模块 | 位置 | 疑点 | 类型 | 状态 |
|---|------|------|------|------|------|
| 1 | auth | TokenService:42 | 并发下乐观锁可能丢更新 | 风险 | 待追改 |
| 2 | order | 全局 | 没看到超时处理 | 待确认 | 开放 |
```
````

- [ ] **Step 2: 验证 markdown 表格与代码块闭合**

Run: `grep -c '```' skills/code-walkthrough/references/note-template.md`
Expected: 偶数(代码围栏成对闭合)。

- [ ] **Step 3: Commit**

```bash
git add skills/code-walkthrough/references/note-template.md
git commit -m "feat: add walkthrough note + open-questions templates"
```

---

### Task 3: 编写 pack 格式规范 _pack-format.md

**Files:**
- Create: `skills/code-walkthrough/references/packs/_pack-format.md`

- [ ] **Step 1: 写入内容**

文件内容(完整):

````markdown
# Pack 编写规范

Pack 是"可插拔燃料":给引擎补充某个技术栈"重点看什么 / 常见存疑点 / 怎么扫盲"。
引擎本身不认识任何技术栈——所有栈相关知识都在 pack 里。

## 放哪个目录

| 目录 | 放什么 | 例 |
|------|--------|-----|
| `languages/` | 编程语言 | go.md, python.md |
| `frameworks/` | 框架/库 | spring.md, react.md |
| `systems/` | 基础设施/中间件/数据系统 | es.md, volcano.md |
| `projects/` | 具体开源项目的脉络导览 | opencompass.md |

## 多 pack 叠加

一个项目通常匹配多个 pack(如 Python + PyTorch + OpenCompass)。
引擎会**全部加载并合并**它们的清单,而非只选一个。
写 pack 时只管自己这一层,不要重复其它层的内容。

## 统一结构(每个 pack 必须包含这四节)

```markdown
# <栈名> 讲解包

## 触发探测
<引擎据此判断是否加载本 pack:文件特征 / 依赖声明 / 注解 等>

## 重点看什么(检查清单)
- <理解该栈代码时优先关注的点,用于 [建全景] 和下钻取舍>

## 常见存疑点(供"③标记存疑"参考)
- <该栈典型的坑/反模式,引擎据此主动提示用户判断>

## 扫盲套路(供"④扫盲")
- <该栈难点的大白话讲法>
```
````

- [ ] **Step 2: 验证代码围栏闭合**

Run: `grep -c '```' skills/code-walkthrough/references/packs/_pack-format.md`
Expected: 偶数。

- [ ] **Step 3: Commit**

```bash
git add skills/code-walkthrough/references/packs/_pack-format.md
git commit -m "docs: add pack authoring format guide"
```

---

### Task 4: 编写 Spring pack

**Files:**
- Create: `skills/code-walkthrough/references/packs/frameworks/spring.md`

- [ ] **Step 1: 写入内容**

文件内容(完整),必须严格遵循 `_pack-format.md` 的四节结构:

```markdown
# Spring Boot 讲解包

## 触发探测
- `pom.xml` 或 `build.gradle` 含 `spring-boot`
- 存在 `@SpringBootApplication` 注解的类

## 重点看什么(检查清单)
- Bean 生命周期与注入方式(构造器注入 vs 字段注入)
- 事务边界:`@Transactional` 的传播级别、自调用失效陷阱
- AOP 切面影响了哪些方法(`@Aspect` / 代理)
- 配置来源:`application.yml` / `@ConfigurationProperties` / profile
- 请求链路:Controller → Service → Repository 分层是否清晰
- 异常处理:`@ControllerAdvice` / `@ExceptionHandler` 统一在哪

## 常见存疑点(供"③标记存疑"参考)
- `@Transactional` 加在 private 方法或被自调用 → 不生效
- 字段注入(`@Autowired` 字段)导致循环依赖被悄悄吞掉
- JPA 懒加载触发 N+1 查询
- `@Async` / 事务在同类自调用下失效(都走代理)
- 把业务逻辑写进 Controller,Service 形同虚设

## 扫盲套路(供"④扫盲")
- 依赖注入:用大白话讲"谁负责 new 对象、谁把它塞进来"
- 代理机制:Spring 在外面包了一层代理,所以"自己调自己"绕过了代理,事务/切面就失效
- Bean 作用域:singleton 默认全局一个,别在里面存请求级状态
```

- [ ] **Step 2: 验证四节齐全**

Run: `grep -E '^## (触发探测|重点看什么|常见存疑点|扫盲套路)' skills/code-walkthrough/references/packs/frameworks/spring.md | wc -l`
Expected: `4`

- [ ] **Step 3: Commit**

```bash
git add skills/code-walkthrough/references/packs/frameworks/spring.md
git commit -m "feat: add Spring Boot walkthrough pack"
```

---

### Task 5: 编写引擎 SKILL.md

**Files:**
- Create: `skills/code-walkthrough/SKILL.md`

- [ ] **Step 1: 写入内容**

文件内容(完整)。frontmatter 的 `description` 决定触发准确度,需覆盖"理解/讲解 AI 写的代码、逐函数/模块对齐、看懂陌生代码库"等场景:

````markdown
---
name: code-walkthrough
description: Use when the user wants to UNDERSTAND existing code (especially AI-generated or unfamiliar code) rather than fix or review it for bugs. Triggers include "带我理解这个模块/项目"、"讲讲这段代码怎么实现的"、"我看不懂 AI 写的这块"、"逐个函数对齐一下实现是否符合预期"、"帮我梳理这个链路"、"walk me through this code/module". Drives a structured, one-step-at-a-time dialogue: builds an overview, suggests a route, drills down段-by-段 and stops to confirm intent, flag risks, and explain unfamiliar tech; persists understanding notes and an open-questions list. Do NOT use for pure bug-hunting code review (use a code-review skill) or for generating new code.
---

# Code Walkthrough(代码讲解 / 逐项确认)

帮用户**理解**代码(尤其 AI 生成的、陌生的),不是替他判断对错。目标是建立 mental model。
本质:苏格拉底式的讲解 + 确认教练。

## 铁律(决定本 skill 成败)

1. **一次只下钻一段,讲完必停**。绝不一口气讲完整个模块。这是"逐项确认"的命根子。
2. **用户随时可打断 / 跳转 / 追问**。放下当前路线去响应,响应完回到原路线并提示"我们回到刚才的 X"。
3. **确认严格按优先级提问:① 对齐意图 > ③ 标记存疑 > ④ 扫盲 > ②(可选)反向检验**。
4. **引擎不认识任何具体技术栈**。栈相关知识全部来自 `references/packs/**`(按需加载)。

## 流程

创建 TodoWrite 跟踪以下阶段。

### [开场] 确定范围
- 用户已指定模块/文件 → 直接进入 [加载 pack]。
- 用户说"整个项目" → 探测入口与技术栈;若是大项目(文件多、不知从何看起),
  **可选**地派一个 subagent 跑全景静态扫描,产出"项目地图"(模块清单 + 依赖关系 + 入口),
  再请用户选一个模块开始。先征求用户是否要这一步。

### [加载 pack]
- 读取 `references/packs/_pack-format.md` 了解结构(若不熟)。
- 探测技术栈,匹配 `references/packs/**` 下所有相关 pack(可多个:语言+框架+系统+项目)。
- **全部加载并合并**它们的"重点看什么 / 常见存疑点 / 扫盲套路"。
- 无匹配 pack → 退化为纯通用讲解,照常进行。

### [建全景]
对目标模块给出:
- **一句话职责**
- **对外接口 / 入口**
- **依赖**(依赖谁、谁依赖它)
- **主链路入口**
然后给一条**建议讲解路线**:按主链路排序的函数/步骤清单,告诉用户"我打算依次讲这几步,可随时改"。

### [下钻循环] —— 沿路线逐段
每一段:
1. **讲这段做什么**。遇到用户可能不懂的技术 → 先按【④扫盲】用大白话讲清(pack 的扫盲套路供料)。
2. **讲完必停**,按优先级提问(通常一次聚焦 1-2 项,别堆问题):
   - **① 对齐意图(主线)**:"这里用了 <策略>,你期望的是这个吗?" / "这块的设计目标是 <X>,对吗?"
   - **③ 标记存疑(补充)**:对照 pack 的"常见存疑点"+ 你自己的判断,主动指出可疑/有风险/不符常规处,请用户裁决。
   - **④ 扫盲**:遇陌生技术时已在第 1 步处理。
   - **②(可选)反向检验**:想确认用户真懂了,可反问一句。
3. **用户响应** → 确认(记入笔记)/ 追问(展开)/ 跳转(放下路线去别处,之后回来)。

### [沉淀]
每聊完一个模块:
- 按 `references/note-template.md` 写/**增量更新** `docs/walkthrough/<模块名>.md`,
  把"已确认意图"落到主链路对应步骤下。
- 把本轮标记的存疑点**追加**到 `docs/walkthrough/_open-questions.md`。
- 告诉用户笔记已更新,以及存疑清单里新增了哪几条。

## 提问风格
- 一次别堆多个问题;聚焦当前这一段。
- 优先用"是不是 / 是 A 还是 B"这类好回答的确认式问法。
- 用户表示"懂了 / 符合预期"就推进,别过度盘问。
````

- [ ] **Step 2: 验证 frontmatter 与四阶段齐全**

Run: `grep -E '^### \[(开场|加载 pack|建全景|下钻循环|沉淀)\]' skills/code-walkthrough/SKILL.md | wc -l`
Expected: `5`

Run: `head -3 skills/code-walkthrough/SKILL.md`
Expected: 第一行 `---`,第二行 `name: code-walkthrough`。

- [ ] **Step 3: Commit**

```bash
git add skills/code-walkthrough/SKILL.md
git commit -m "feat: add code-walkthrough engine SKILL.md"
```

---

### Task 6: 干跑验证(dry-run walkthrough)

**Files:**
- 无新增;在本仓库或一个示例 Spring 模块上空跑流程。

- [ ] **Step 1: 准备一个被讲解的目标**

挑一段真实代码(可用本仓库任意文件,或临时找一个小 Spring 类)。确认 `docs/walkthrough/` 不存在或为空。

- [ ] **Step 2: 按 SKILL.md 流程空跑一轮**

人工核对引擎是否产生以下行为:
- [建全景] 是否给出"一句话职责 / 接口 / 依赖 / 主链路 + 建议路线"四要素
- [下钻] 是否**每段停下**,且提问顺序符合 ①③④② 优先级
- 是否在打断后能回到原路线
- [沉淀] 是否生成 `docs/walkthrough/<模块>.md` 且字段与模板一致
- 存疑点是否追加进 `docs/walkthrough/_open-questions.md`

Expected: 以上每项都满足;不满足则回到对应文件修订。

- [ ] **Step 3: 验证产出物结构**

Run: `test -f docs/walkthrough/_open-questions.md && grep -E '^\| # \| 模块' docs/walkthrough/_open-questions.md`
Expected: 表头存在(若本轮产生了存疑点)。

- [ ] **Step 4:(如有修订)Commit**

```bash
git add -A
git commit -m "fix: refine code-walkthrough based on dry-run"
```

---

## Self-Review(已执行)

- **Spec 覆盖**:开场/加载 pack/建全景/下钻/沉淀 → Task 5;笔记+存疑清单 → Task 2;pack 格式与分类叠加 → Task 3;Spring pack → Task 4;可选全景扫描 → Task 5 [开场];退化为通用讲解 → Task 5 [加载 pack]。无遗漏。
- **占位符**:每个文件均给出完整内容,无 TBD/TODO。
- **一致性**:目录名(packs 四子目录)、状态/类型取值、四节 pack 结构、五阶段流程在 Task 2/3/4/5 间用词一致。
