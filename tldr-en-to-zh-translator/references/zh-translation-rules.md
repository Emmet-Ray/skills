# tldr 简体中文翻译规则

本文档提炼简体中文翻译阶段的核心规则，并导航到当前 tldr 仓库中的权威资料。它不替代项目贡献规范。

## 目录

- [适用范围](#适用范围)
- [权威来源优先级](#权威来源优先级)
- [每个翻译批次固定必读](#每个翻译批次固定必读)
- [根据页面内容条件读取](#根据页面内容条件读取)
- [核心翻译契约](#核心翻译契约)
- [页面路径和结构](#页面路径和结构)
- [标准中文表达](#标准中文表达)
- [占位符](#占位符)
- [简体中文排版](#简体中文排版)
- [术语和不确定内容](#术语和不确定内容)
- [翻译阶段完成条件](#翻译阶段完成条件)

## 适用范围

- 目标目录为 `pages.zh`。
- 第一版只新增缺失的简体中文页面，不同步、更新或润色已有中文页面。
- 每个中文页面必须以当前英文 `pages` 目录中的对应页面为翻译源。
- 一次 skill 调用中的所有命令构成一个翻译批次；先完成整个批次的初稿，再统一进入校验阶段。

## 权威来源优先级

发生冲突时，按以下顺序决定：

1. 当前英文源页面决定需要翻译的内容和页面结构。
2. 当前仓库的 `CONTRIBUTING.md`。
3. 当前仓库的英文 `contributing-guides/style-guide.md`。
4. 当前仓库的 `contributing-guides/translation-templates/`。
5. `contributing-guides/style-guide.zh.md`。
6. 现有 `pages.zh` 页面。
7. agent 的一般知识和翻译习惯。

本文档是规则摘要和资料导航。如果本文档与更高优先级来源冲突，使用更高优先级来源，并在后续维护中修正本文档。

## 每个翻译批次固定必读

### 英文源页面

完整读取本批次的所有英文源页面。英文源页面决定：

- 页面标题和类型。
- 描述、补充说明及其顺序。
- 示例数量和顺序。
- 示例命令及参数结构。
- More information、See also 和子命令说明。

### CONTRIBUTING.md

读取：

- `Guidelines`
- `Markdown format`
- `Translations`

### 英文 style guide

读取 `contributing-guides/style-guide.md` 中与以下主题相关的章节：

- 页面布局、页面类型和通用写作。
- 标题与程序描述。
- More information 和 See also。
- 示例描述和示例命令。
- 选项、占位符和按键语法。
- 帮助和版本命令。
- 通用翻译规则和 Chinese-Specific Rules。

### 通用简体中文模板

读取：

- `contributing-guides/translation-templates/common-arguments.md` 中的 `zh` 行。
- `contributing-guides/translation-templates/common-descriptions.md` 中的 `zh` 行。
- `contributing-guides/translation-templates/more-info-link.md` 中的 `zh` 条目。

## 根据页面内容条件读取

- 英文页面是 alias：读取 `contributing-guides/translation-templates/alias-pages.md` 的 `zh` 条目。
- 页面包含子命令说明：读取 `contributing-guides/translation-templates/subcommand-mention.md` 的 `zh` 条目。
- 页面包含 See also：读取 `contributing-guides/translation-templates/see-also-mentions.md` 的 `zh` 条目。
- 页面属于 Windows 或 PowerShell：读取英文和中文版 style guide 中对应的 Windows 或 PowerShell 规则。
- 遇到不确定术语或表达：搜索同一命令族或相似的现有 `pages.zh` 页面。
- 需要辅助理解中文规范：读取 `contributing-guides/style-guide.zh.md` 的对应章节。

只读取适用于当前页面的条件资料和其中的简体中文内容，不加载无关页面类型或其他语言条目。

## 核心翻译契约

### 严格对应英文页面

- 保持英文页面的整体结构。
- 保持描述和补充说明的顺序与语义。
- 保持示例数量和顺序。
- 不自行增加、删除、合并、拆分或重新排序示例。
- 不借翻译之机修正或扩展英文页面；发现英文页面可能存在问题时，记录问题但不改变本次翻译范围。

### 需要翻译的内容

- 页面描述。
- 补充说明。
- 示例描述。
- 标准提示语。
- See also 和子命令说明中的自然语言。
- 可以翻译且项目规范允许翻译的占位符。

翻译应准确、简洁、自然，避免机械逐词对应，同时不能改变英文原意。

### 保持不变的内容

- 命令名和子命令名。
- 命令选项及其大小写。
- 命令语法和参数结构。
- URL。
- 固定协议名、产品名、项目名和官方专有名称。
- 具有固定含义的字面值。

除翻译占位符外，不因中文表达需要改变命令块的行为或参数结构。

## 页面路径和结构

- 将 `pages/<platform>/<command>.md` 映射到 `pages.zh/<platform>/<command>.md`。
- 保持平台目录和文件名不变。
- 保持页面标题符合原命令的大小写约定。
- 保持 Markdown 段落、列表和命令块的结构。

## 标准中文表达

项目翻译模板中已有标准表达时，优先使用模板，不自行创造变体。例如：

- `Display help`：`显示帮助`
- `Display version`：`显示版本`
- More information：`更多信息：<https://example.com>。`
- See also：使用 `see-also-mentions.md` 的 `zh` 模板。
- Alias：使用 `alias-pages.md` 的 `zh` 模板。
- 子命令说明：使用 `subcommand-mention.md` 的 `zh` 模板。

模板中的 `example` 仅代表实际命令或 URL 的替换位置。

## 占位符

- 按项目中文规范尽量翻译西文占位符。
- 优先使用 `common-arguments.md` 已提供的标准翻译。
- 保持 `{{...}}` 语法、可选项结构、分组选项和省略号结构不变。
- 路径占位符保持平台所需的路径分隔符。
- 不翻译命令选项、固定值或必须原样传递给程序的内容。

常用示例：

- `{{path/to/file}}` → `{{路径/到/文件}}`
- `{{path/to/directory}}` → `{{路径/到/目录}}`
- `{{path/to/file_or_directory}}` → `{{路径/到/文件或目录}}`
- `{{package}}` → `{{软件包}}`
- `{{username}}` → `{{用户名}}`
- `{{password}}` → `{{密码}}`
- `{{command}}` → `{{命令}}`
- `{{port}}` → `{{端口}}`
- `{{value}}` → `{{值}}`

## 简体中文排版

遵循当前 `style-guide.md` 的 Chinese-Specific Rules，重点包括：

- 中文与西文单词、阿拉伯数字之间通常保留一个空格。
- 数字和单位之间通常保留一个空格，度数和百分比除外。
- 全角标点前后不添加额外空格。
- 中文句子通常使用全角标点。
- 句子末尾是半角内容时，按项目规则选择句末标点。
- 技术术语和专有名称使用准确、官方的写法，不使用非官方简称。
- 不翻译 `example.com`。

若标准翻译模板对具体标点已有明确写法，使用模板写法。

## 术语和不确定内容

- 优先参考当前仓库中同一命令族或相似 `pages.zh` 页面的用词。
- 现有中文页面只作为表达参考，不能覆盖当前项目规范。
- 遇到多种合理表达时，选择与上下文和现有页面最一致的最佳初稿。
- 记录所有不确定术语、表达和选择理由，在“用户审核与迭代”阶段集中提示用户。
- 除非无法确定英文原意、因而无法可靠继续，否则不在翻译阶段频繁暂停询问用户。

## 翻译阶段完成条件

- 本批次的所有目标中文页面都已创建。
- 每个页面均完整翻译了应翻译内容。
- 页面结构和示例顺序与英文源页面对应。
- 命令语义和参数结构未被主动改变。
- 所有不确定项均已记录。
- 整个批次进入 `TRANSLATED` 状态，并统一交给“翻译质量校验”阶段。

翻译阶段只负责生成尽可能正确的初稿。结构核对、格式检查、lint 和系统性规则验证属于后续“翻译质量校验”阶段。
