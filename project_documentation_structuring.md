# Project Documentation Structuring

> **文档定位**：
>
> - 定义项目根目录的文档结构标准，将项目信息拆分为四份职责单一的文档
> - `README.md` 回答「是什么、怎么跑、项目结构」；`AGENTS.md` 约束 Agent 行为；`SPEC.md` 记录需求与待办；`PROMPT.md` 承载单次任务的精确指令
> - Agent 在参与项目前，应先检索这四份文档，再依据本模板生成或补全缺失内容
> - Agent 阅读顺序：`README.md` → `AGENTS.md` → `SPEC.md` → `PROMPT.md`（先懂项目，再懂规则，再懂目标，最后执行单次任务）
>
> **生成约束**：
>
> - `<项目名称>`、`<仓库地址>` 等尖括号标记为项目级占位符，需替换为实际值
> - 各文档严格遵守职责边界，同一信息只在一处维护，其他地方通过引用指向
> - 只填充项目中实际存在的信息；尚无内容的章节标记为「待补充」，禁止为完成模板而臆想内容
> - 说明性约束文字（如「待补充」「禁止臆想」）必须保留，不得删除或替换
> - 每个文件条目应包含：**职责说明**、**标准模板结构**、**参考标准**
> - `PROMPT.md` 随任务变化可频繁更新；`AGENTS.md` 变更需谨慎，因影响所有后续任务

---

## 文档结构树

```
<项目名称>/
    ├── .gitattributes
    ├── .gitignore
    ├── .editorconfig
    ├── AGENTS.md
    ├── README.md
    ├── SPEC.md
    └── PROMPT.md
```

> 树中文件排列顺序不代表阅读顺序。阅读顺序见文首「文档定位」：`README.md` → `AGENTS.md` → `SPEC.md` → `PROMPT.md`。

---

## 各文件职责与内容范围

以下每个文件均包含：职责说明、标准模板结构、参考标准。Agent 生成文档前应先检索对应参考标准。

> **裁剪说明**：模板中的章节为推荐结构。应根据项目实际特性保留或裁剪——尚不具备的部分标记为「待补充」，不得填充臆想内容。配置文件模板为多语言/多技术栈超集，按项目实际技术栈保留对应行，删除无关行。

---

### 根目录配置文件

#### `.gitattributes`

**职责**：定义仓库级别的 Git 属性配置。用于指定文件的行尾处理方式、diff 策略、语言标记、导出忽略规则等。

**标准模板结构**：

```
# 行尾处理
* text=auto

# 强制特定文件的行尾风格
# 注意：以下按语言分类的行尾规则为多语言超集，请根据项目实际使用的编程语言保留对应行，删除无关行
*.md text eol=lf
*.txt text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
*.json text eol=lf
*.xml text eol=lf
*.css text eol=lf
*.scss text eol=lf
*.sh text eol=lf
*.py text eol=lf
*.rb text eol=lf
*.go text eol=lf
*.rs text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
*.js text eol=lf
*.jsx text eol=lf
*.html text eol=lf
*.svg text eol=lf
*.conf text eol=lf
*.cfg text eol=lf
*.toml text eol=lf
*.ini text eol=lf
*.sql text eol=lf
*.java text eol=lf
*.kt text eol=lf
*.scala text eol=lf
*.c text eol=lf
*.cpp text eol=lf
*.h text eol=lf
*.hpp text eol=lf

# Windows 脚本
*.bat text eol=crlf
*.ps1 text eol=crlf
*.cmd text eol=crlf

# 二进制文件标记
# 注意：binary 标记作用于已被 Git 追踪的文件（禁止行尾转换和文本 diff）
# 与 .gitignore 的互补关系：.gitignore 阻止新文件被追踪，.gitattributes 处理已追踪文件的属性
# 已被追踪的二进制文件（如历史提交的 .dll/.exe）仍需此处的 binary 标记
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.ico binary
*.pdf binary
*.zip binary
*.gz binary
*.tar binary
*.7z binary
*.dll binary
*.exe binary
*.so binary
*.dylib binary
*.woff binary
*.woff2 binary
*.ttf binary
*.eot binary

# 自定义 diff 驱动
# 注意：以下 diff driver 需要在 Git 配置中定义实际行为才能生效
# 例如：git config diff.markdown.textconv "pandoc -f markdown -t plain"
# 参见：https://git-scm.com/docs/gitattributes#_defining_a_custom_diff_driver
*.md diff=markdown
*.ipynb diff=jupyternotebook

# 导出时忽略的文件
.gitattributes export-ignore
.gitignore export-ignore
```

**参考标准**：

| 标准/框架                    | 说明                         | 链接                                                                            |
| ---------------------------- | ---------------------------- | ------------------------------------------------------------------------------- |
| Git 官方文档 - gitattributes | Git 属性配置完整参考         | https://git-scm.com/docs/gitattributes                                          |
| GitHub - Customizing Git     | GitHub 关于 Git 自定义的指南 | https://docs.github.com/en/get-started/getting-started-with-git/customizing-git |
| GitHub Linguist 文档         | 语言检测与标记规则说明       | https://github.com/github-linguist/linguist/blob/main/docs/overrides.md         |

---

#### `.gitignore`

**职责**：定义 Git 版本控制应忽略的文件与目录模式。典型内容包括构建产物、依赖目录、本地配置覆盖、编辑器临时文件、日志输出等不应纳入版本管理的内容。

**标准模板结构**：

```
# ====================
# 依赖目录
# ====================
# 注意：以下按技术栈分类的忽略规则为多语言超集，请根据项目实际使用的技术栈保留对应行，删除无关行
node_modules/
vendor/
.bundle/
.jbang/
.venv/
venv/
env/
__pycache__/
.pytest_cache/
.m2/
.gradle/
.cargo/
target/
obj/

# ====================
# 构建产物
# ====================
dist/
build/
out/
/bin/
*.o
*.so
*.dylib
*.dll
*.exe
*.class
# 注意：以上二进制文件扩展名与 .gitattributes 中的 binary 标记互补
# .gitignore 阻止新文件被追踪，.gitattributes 处理已追踪文件的属性
*.pyc
*.pyo

# ====================
# 本地配置覆盖
# ====================
.env
.env.local
.env.*.local
config.local.*
*.local.conf

# ====================
# 编辑器与 IDE
# ====================
.idea/
.vscode/
*.swp
*.swo
*~
.project
.classpath
.settings/

# ====================
# 操作系统文件
# ====================
.DS_Store
Thumbs.db
desktop.ini

# ====================
# 日志与临时文件
# ====================
*.log
logs/
tmp/
temp/
.cache/

# ====================
# 测试与覆盖率
# ====================
coverage/
.nyc_output/
*.cover
test-results/

# ====================
# 包管理产物
# ====================
*.tgz
*.tar.gz
```

**参考标准**：

| 标准/框架                | 说明                                  | 链接                                        |
| ------------------------ | ------------------------------------- | ------------------------------------------- |
| GitHub gitignore 模板库  | 官方维护的各语言/框架 .gitignore 模板 | https://github.com/github/gitignore         |
| Git 官方文档 - gitignore | Git 忽略规则完整参考                  | https://git-scm.com/docs/gitignore          |
| gitignore.io             | 在线生成器，支持按技术栈组合生成      | https://www.toptal.com/developers/gitignore |

---

#### `.editorconfig`

**职责**：跨编辑器的代码风格统一配置。定义缩进风格、行尾处理、字符编码等基础格式化规则，确保不同编辑器和 IDE 的开发者遵循一致的代码风格。

**标准模板结构**：

```
# EditorConfig: https://editorconfig.org
# 顶级配置，阻止向上查找父级 .editorconfig
root = true

# 所有文件的默认设置
[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

# Markdown 文件
[*.md]
trim_trailing_whitespace = false

# 缩进为 2 空格的类型
[*.{yml,yaml,json,ts,tsx,js,jsx,css,scss,html,vue,svelte}]
indent_size = 2

# 使用 Tab 缩进的语言
[*.go]
indent_style = tab

[Makefile]
indent_style = tab

# Windows 脚本使用 CRLF
[*.{bat,cmd,ps1}]
end_of_line = crlf
```

**参考标准**：

| 标准/框架             | 说明                                | 链接                                 |
| --------------------- | ----------------------------------- | ------------------------------------ |
| EditorConfig 官方文档 | EditorConfig 规范与完整属性说明     | https://editorconfig.org/            |
| Google Style Guides   | Google 编码规范中对格式化风格的定义 | https://github.com/google/styleguide |

---

### 根目录文档

#### `README.md`

**定位**：项目名片

**职责**：项目的入口文档，面向首次接触项目的读者。涵盖项目概述、快速上手、项目结构与文档导航。

**内容范围**：

- 项目是什么
- 怎么跑起来
- 项目结构

**不应包含**：技术栈（归 `AGENTS.md`）、功能需求与待办（归 `SPEC.md`）、编码规范（归 `AGENTS.md`）、单次任务步骤（归 `PROMPT.md`）。

**标准模板结构**：

````markdown
# <项目名称>

> <一句话描述项目的核心价值>

## 目录

- [文档导航](#文档导航)
- [项目概述](#项目概述)
- [快速上手](#快速上手)
- [项目结构](#项目结构)

> 以上为示例条目。实际生成时，应根据文档中包含的顶层章节调整目录。

## 文档导航

| 文档 | 说明 |
|------|------|
| [行为准则](AGENTS.md) | 技术栈、代码规范、禁止事项与偏好设置 |
| [任务清单](SPEC.md) | 功能需求、验收标准与待办事项 |
| [精确指令](PROMPT.md) | 当前单次任务的详细步骤与约束 |

## 项目概述

<项目名称> 是一个 <项目类型>，用于 <解决什么问题>。

### 核心功能

> 只列出项目中实际存在的核心功能，禁止为填满模板而臆想功能。尚无明确功能时，标记为「待补充」。详细需求参见 [SPEC.md](SPEC.md)。

- <功能>：<简要说明>

## 快速上手

### 前置条件

> 只列出项目中实际依赖的前置条件，禁止臆想。尚无明确要求时，标记为「待补充」。

| 依赖 | 最低版本 | 说明 |
|------|----------|------|
| <依赖> | <版本号> | <用途说明> |

### 安装与运行

```bash
# 克隆仓库
git clone <仓库地址>
cd <项目名称>

# 安装依赖
<安装依赖命令>

# 配置环境变量
<环境变量配置命令或说明>

# 启动项目
<启动命令>
```

### 验证安装

```bash
# 运行测试
<运行测试命令>

# 访问服务
<访问地址或验证方式>
```

## 项目结构

> 根据项目的实际文件结构填写。使用树状图描述各目录与关键文件的职责。
> 根目录文档文件：`AGENTS.md`、`README.md`、`SPEC.md`、`PROMPT.md`。
> 以下仅为格式示例，`<项目名称>`、`<...>` 等所有占位符必须替换为项目实际目录与文件。

```
<项目名称>/
├── <目录>/
│   ├── <子目录>/
│   └── <文件名>
├── <文件名>
└── ...
```
````

**参考标准**：

| 标准/框架              | 说明                                  | 链接                                                                                                                              |
| ---------------------- | ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| GitHub README 最佳实践 | GitHub 官方关于编写优秀 README 的指南 | https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes |
| Make a README          | README 写作指南与模板                 | https://www.makeareadme.com/                                                                                                      |
| standard-readme 规范   | 标准化 README 规范，定义应包含的章节  | https://github.com/RichardLitt/standard-readme                                                                                    |
| awesome-readme         | 社区维护的优秀 README 范例合集        | https://github.com/matiassingers/awesome-readme                                                                                   |

---

#### `AGENTS.md`

**定位**：行为准则

**职责**：面向 AI Agent 的项目级行为约束。定义技术栈、编码规范、禁止事项与偏好设置，使 Agent 在与项目交互时遵循统一标准。

**内容范围**：

- 技术栈
- 代码规范
- 禁止事项
- 偏好设置

**不应包含**：功能需求与验收标准（归 `SPEC.md`）、单次任务步骤（归 `PROMPT.md`）、环境搭建命令（归 `README.md`）。

**标准模板结构**：

```markdown
# <项目名称> 行为准则

## 技术栈

> 只列出项目中实际使用的技术，禁止臆想。尚无明确技术栈时，标记为「待补充」。

| 类别 | 技术 | 用途 |
|------|------|------|
| <类别> | <技术名称> | <用途说明> |

### 关键入口

- 项目简述：<一句话描述项目是什么>
- 关键入口文件：<主入口文件路径>
- 文档索引：参见 [README.md](README.md)、[SPEC.md](SPEC.md)、[PROMPT.md](PROMPT.md)

## 代码规范

> 根据项目实际使用的编程语言裁剪以下章节，不适用的条目标记为「待补充」。Agent 应检索对应语言的官方编码规范（如 PEP 8、Effective Go 等），而非套用空泛模板。

### 命名与格式

- 文件/目录命名：<规则>
- 类/函数/变量命名：<规则>
- 缩进与行宽：<规则>
- Import 排序：<规则>

### 注释与文档

- 语言约束：代码使用英文，注释使用中文
- 代码定位：文档优先使用概念性描述（如 `<模块名称>` 的认证逻辑），严禁依赖行号定位代码
- 禁止编写开发过程式说明（如"此处修复了 xxx"）
- 严禁在代码、注释、Commit Message 或 PR Body 中出现开发进度词汇（`FIXED`、`Step`、`Phase` 等）或 AI 工具名称
- 验收标准编号（如 `AC-001`）仅在 `SPEC.md` 与 `PROMPT.md` 中使用，禁止出现在代码与 Git 记录中

### 错误处理与测试

- 异常/错误处理策略：<规则>
- 日志规范：<规则>
- 测试命名与组织：<规则>

### 质量底线

- 项目早期保留最小必要质量标准——可运行、可验证、可回滚
- Bug 修复闭环：严格遵循"先复现 → 再修复 → 后验证"

## 禁止事项

- 永久禁止使用 `<禁止命令>`
- `<AI 上下文文件名>` 必须基于项目实际定制，禁止套用空泛模板
- 禁止在 Git 记录中暴露 AI 协作痕迹
- <项目特定的架构红线、安全红线>

## 偏好设置

- 回复语言：<回复语言要求>
- 方案先行：写代码前先描述方案，等用户批准再动手
- 需求澄清：需求模糊时，先提问澄清再写代码
- 防御性代码：不能写兼容性代码，除非用户主动要求
- 任务拆分：修改超过 3 个文件时，先拆成小任务
- Bug 处理：出 bug 时，先写重现的测试再修复
- 边界思考：写完代码后，列出边缘情况和测试用例
- 反思机制：每次被纠正后，反思并制定不再犯的计划
- <其他项目特定偏好>
```

**参考标准**：

| 标准/框架                | 说明                             | 链接                                                        |
| ------------------------ | -------------------------------- | ----------------------------------------------------------- |
| AGENTS.md 完整指南       | 跨工具 AI Agent 上下文文件标准   | https://vibecoding.app/blog/zh/agents-md-wanzheng-zhinan    |
| AGENTS.md 内容策略       | "给 AI 一张地图，而不是一本手册" | https://blog.csdn.net/ID314846818/article/details/161839666 |
| Context Engineering 入门 | 上下文工程的核心理念与实践       | https://github.com/coleam00/context-engineering-intro       |
| Google Style Guides      | 多语言编码规范参考               | https://github.com/google/styleguide                        |

---

#### `SPEC.md`

**定位**：任务清单

**职责**：记录项目的需求目标与进度状态。定义做什么、做到什么程度算完成，以及当前待办事项。

**内容范围**：

- 功能需求
- 验收标准
- 待办事项

**不应包含**：单次任务执行步骤（归 `PROMPT.md`）、编码规范（归 `AGENTS.md`）、环境搭建（归 `README.md`）。

**与 `PROMPT.md` 的边界**：`SPEC.md` 管「做什么、怎么算完成」（相对稳定）；`PROMPT.md` 管「当前这一次怎么做」（随任务更新）。

**标准模板结构**：

```markdown
# <项目名称> 任务清单

## 功能需求

> 按模块或特性列出，标注优先级。只记录项目中实际确认的需求，禁止臆想。编号列（如 `FR-001`）对应下方「验收标准」中的同名标题锚点，供 `PROMPT.md` 链接引用。

| 编号 | 模块/特性 | 描述 | 优先级 |
|------|-----------|------|--------|
| FR-001 | <模块> | <需求描述> | P0 / P1 / P2 |

### 非功能需求

> 性能、安全、可用性等质量目标。尚无明确要求时，标记为「待补充」。

| 编号 | 类别 | 要求 |
|------|------|------|
| NFR-001 | <类别> | <要求描述> |

## 验收标准

> 每条标准必须可测试、可判定，禁止模糊描述（如"性能良好"）。推荐使用 Given-When-Then 格式。

### FR-001

<特性名称>

- [ ] **AC-001**：Given <前置条件>，When <操作>，Then <预期结果>
- [ ] **AC-002**：<可判定的完成条件>

## 待办事项

> 状态：待办 / 进行中 / 已完成。完成后勾选并注明日期。编号列（如 `TODO-001`）对应下方同名标题锚点，供 `PROMPT.md` 链接引用。

| 编号 | 任务 | 关联需求 | 状态 | 备注 |
|------|------|----------|------|------|
| TODO-001 | <任务描述> | FR-001 | 待办 | |

### TODO-001

<任务详细说明（可选）>
```

**参考标准**：

| 标准/框架             | 说明                            | 链接                                                |
| --------------------- | ------------------------------- | --------------------------------------------------- |
| User Story 与验收标准 | 用户故事与 Given-When-Then 写法 | https://martinfowler.com/bliki/UserStory.html       |
| INVEST 原则           | 优秀用户故事的评判标准          | https://en.wikipedia.org/wiki/INVEST_%28mnemonic%29 |

---

#### `PROMPT.md`

**定位**：精确指令

**职责**：承载当前单次任务的可执行指令。定义详细步骤、输入输出与约束条件，供 Agent 直接执行。

**内容范围**：

- 单次任务的详细步骤
- 输入输出
- 约束条件

**不应包含**：全局编码规范（归 `AGENTS.md`）、长期功能需求（归 `SPEC.md`）、项目介绍（归 `README.md`）。

**生命周期**：一个任务一份指令。执行前必须已读 `AGENTS.md`；任务目标必须对应 `SPEC.md` 中的条目。任务完成后可归档或清空，准备下一次任务。

**标准模板结构**：

```markdown
# 任务：<任务标题>

> 关联需求：[FR-001](SPEC.md#fr-001) / [TODO-001](SPEC.md#todo-001)
> 遵循：[AGENTS.md](AGENTS.md)

## 任务描述

<本次要完成什么，一句话说明>

## 详细步骤

> 按任务实际需要列出有序步骤，可增可减，禁止为凑满模板而拆分或合并步骤。

1. <动作> → <预期中间结果>
<!-- 按需继续添加后续步骤 -->

## 输入

### 前置条件

- <环境、分支、依赖等前置要求>

### 上下文文件

- `<文件路径>`：<该文件提供的上下文说明>

### 依赖信息

- <接口定义、数据结构、配置项等>

## 输出

### 交付物

> 按任务实际产出列出，可增可减。

- [ ] <交付物：文件 / 功能 / 文档等>

### 验收点

- [ ] <本任务完成后可立即验证的检查项，对应 [SPEC.md 中的 AC-001](SPEC.md#fr-001)>

## 约束条件

> 继承 `AGENTS.md` 的全局约束。以下为本任务特有的额外限制。

- <禁止修改的范围>
- <必须使用的技术或模式>
- <性能/兼容性等任务级限制>
```

**参考标准**：

| 标准/框架                | 说明                     | 链接                                                  |
| ------------------------ | ------------------------ | ----------------------------------------------------- |
| Context Engineering 入门 | 最小上下文与精确指令实践 | https://github.com/coleam00/context-engineering-intro |
| Prompt Engineering Guide | 结构化指令编写参考       | https://www.promptingguide.ai/                        |

---

## 四文档职责边界

### 归属规则

| 信息类型                             | 归属                                                    | 禁止出现在                             |
| ------------------------------------ | ------------------------------------------------------- | -------------------------------------- |
| 项目介绍、启动命令、项目结构         | `README.md`                                           | AGENTS / SPEC / PROMPT                 |
| 功能概览（一两句简述）               | `README.md`                                           | SPEC（详细需求）                       |
| 技术栈、编码规范、全局禁止、交互偏好 | `AGENTS.md`                                           | README / PROMPT（完整复述）            |
| 功能需求（详细）、验收标准、待办     | `SPEC.md`                                             | README（详细列表）/ PROMPT（完整复述） |
| 单次任务步骤、输入输出、任务级约束   | `PROMPT.md`                                           | README / SPEC                          |
| Git/编辑器配置                       | `.gitattributes` / `.gitignore` / `.editorconfig` | 四个`.md`                            |

> **说明**：「禁止完整复述」指不得复制另一文档的全文章节；通过 Markdown 链接引用，或为执行任务而提及编号（如 `FR-001`、`AC-001`），均属允许。

### 交叉引用规则

- `README.md` → 链接到 `AGENTS.md`、`SPEC.md`、`PROMPT.md`
- `PROMPT.md` → 引用 `SPEC.md` 中的需求/待办编号，并声明遵循 `AGENTS.md`
- `SPEC.md` → 可引用 `README.md` 中的项目概述，但不重复其内容
- 同一事实只在一处维护，其余通过 Markdown 链接指向

### 联动更新规则

| 变更来源                         | 需同步检查                                 |
| -------------------------------- | ------------------------------------------ |
| `SPEC.md` 需求或验收标准变更   | `PROMPT.md` 是否过期                     |
| `AGENTS.md` 规范或禁止事项变更 | 进行中的`PROMPT.md` 是否需要调整         |
| `README.md` 项目结构变更       | 项目结构章节是否与实际一致                 |
| 配置文件变更                     | 是否影响`README.md` 快速上手中的相关说明 |

---

## Agent 文档生成工作流

Agent 在生成或更新文档时，应遵循以下流程。核心原则：只填充项目中实际存在的信息，禁止为完成模板而臆想内容。

**推荐生成顺序**：配置文件 → `README.md` → `AGENTS.md` → `SPEC.md` → `PROMPT.md`（最后仅在执行具体任务时生成）

### 阶段一：准备

1. **识别目标文件**：确定需要生成或更新的文件属于结构树中的哪一项。
2. **检索参考标准**：查阅本文档中对应条目的参考标准，提取必须遵循的规范与推荐结构。
3. **检索项目实际内容**：从代码库、现有文档、对话记录中提取与目标文件相关的事实信息。

### 阶段二：分析与对照

4. **提取关键实体**：模块名称、技术名称、目录路径、需求编号等。
5. **对照模板结构**：检查哪些章节已有内容、哪些尚缺。
6. **检查职责边界**：确认内容不会错放到其他文件中。

### 阶段三：生成

7. **按模板填充**：将已知信息填入对应章节；不具备的部分标记为「待补充」。
8. **建立交叉引用**：在文档间添加正确的 Markdown 链接。
9. **输出完整度摘要**：列出每个文件的已完成 / 待补充 / 不适用章节，并给出补充建议。

### 阶段四：质量验证

10. **结构树一致性**：确认生成文件与文档结构树定义完全一致，无多余或缺失文件。
11. **文件边界检查**：确认各文档内容未越界。
12. **交叉引用验证**：链接指向正确的目标文件。
13. **清单逐项检查**：对照[附录 文档质量检查清单](#附录-文档质量检查清单)逐项验证。

---

## 附录 文档质量检查清单

Agent 在完成文档生成后，应逐项检查以下清单：

| 检查项                | 说明                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------- |
| 占位符与待补充标记    | 已知项目信息的`<占位符>` 已全部替换；信息不足的章节标记为「待补充」，不存在臆想内容 |
| 交叉引用正确          | 文档间的链接指向正确的目标文件与章节锚点                                              |
| 参考标准已检索        | 各文件条目的参考标准已被实际查阅                                                      |
| 技术栈归属正确        | 技术栈仅在`AGENTS.md` 维护；`README.md` 不出现技术栈表格或详细技术说明            |
| 术语统一              | 全文使用统一的术语命名                                                                |
| 格式规范              | Markdown 格式符合通用规范（标题层级、列表缩进、代码块语言标记）                       |
| 内容无重复            | 同一信息只在一处维护，其他地方通过引用指向                                            |
| 结构树一致性          | 生成的文件集合与文档结构树定义完全一致                                                |
| 文件边界遵守          | 各文档内容严格遵守[四文档职责边界](#四文档职责边界)，无内容错放                        |
| AGENTS 编码规范已适配 | `AGENTS.md` 编码规范已根据项目实际编程语言裁剪                                      |
| SPEC 验收标准可测试   | 每条验收标准可判定通过或失败，无模糊描述                                              |
| PROMPT 步骤可执行     | `PROMPT.md` 步骤可逐步执行，每步有明确的中间结果                                    |
| PROMPT 任务可追溯     | `PROMPT.md` 任务可对应 `SPEC.md` 中的需求或待办条目                               |
| 参考链接有效          | 参考标准表格中的链接可访问且指向正确资源                                              |
