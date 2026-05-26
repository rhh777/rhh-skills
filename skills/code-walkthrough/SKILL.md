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
