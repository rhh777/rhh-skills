# rhh-skills

个人 Claude Code skills 集合。

## Skills

### code-walkthrough —— 代码讲解 / 逐项确认

帮你**理解**代码(尤其 AI 生成的、陌生的),而不是替你判断对错。通过结构化、一次一步的对话,
按模块/函数粒度建立心智模型:先建全景给出讲解路线,再逐段下钻、讲完必停,跟你确认实现是否符合
预期、标记存疑点、对陌生技术扫盲;并把理解沉淀成笔记与存疑清单。

- 引擎:`skills/code-walkthrough/SKILL.md`(不认识任何具体技术栈)
- 可插拔技术栈包:`skills/code-walkthrough/references/packs/`(languages / frameworks / systems / projects 四类,支持多包叠加)
- 产出物:理解笔记 + 存疑清单,落到被讲解项目的 `docs/walkthrough/`

新增技术栈包:照 `references/packs/_pack-format.md` 往对应子目录丢一个 md 即可,引擎零改动。
