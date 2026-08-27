# AGENTS.md 模板与自动生成

## 何时生成

启动检查发现项目根目录没有 AGENTS.md 时，自动扫描项目并生成。以下情况也应更新：
- 用户新增了依赖或改变了技术栈
- 项目目录结构发生重大变化
- 用户明确要求更新

## 自动生成流程

1. 扫描项目根目录文件列表
2. 读取关键配置文件识别技术栈：
   - Python：requirements.txt / pyproject.toml / setup.py / Pipfile
   - Node：package.json / pnpm-lock.yaml / yarn.lock
   - Go：go.mod
   - Rust：Cargo.toml
   - Java：pom.xml / build.gradle
   - 其他：README.md、Makefile、Dockerfile、docker-compose.yml
3. 读取 .gitignore 识别敏感文件和产物目录
4. 扫描目录结构（一级 + 关键二级目录，跳过 .gitignore 覆盖的路径）
5. 从配置文件中提取常用命令（scripts、Makefile targets 等）
6. 观察现有代码风格（缩进、命名约定）
7. 填充下方模板，展示给用户确认后写入项目根目录

## 模板

```markdown
# {{项目名称}}

## 项目简介

{{一句话描述项目做什么。根据 README、代码内容、项目结构自动推断}}

## 技术栈

- 语言：{{如 Python 3.10+ / Node 20 / Go 1.22}}
- 框架/核心库：{{如 FastAPI + SQLAlchemy / React + Vite}}
- 包管理：{{如 pip + requirements.txt / pnpm / go mod}}
- 其他关键依赖：{{从依赖配置中识别并列出，标注用途}}

## 目录结构

{{扫描目录生成，每个文件/目录加一行注释说明用途}}

示例格式：
  project/
  ├── src/           # 源代码
  │   ├── api/       # 接口层
  │   └── core/      # 核心逻辑
  ├── tests/         # 测试
  ├── docs/          # 文档
  └── README.md

## 常用命令

{{从 package.json scripts / Makefile / pyproject.toml / README 中提取}}

至少包含（如适用）：
- 安装依赖：{{命令}}
- 开发运行：{{命令}}
- 构建：{{命令}}
- 测试：{{命令}}
- 代码检查/格式化：{{命令}}

## 代码约定

- 缩进：{{如 4空格 / 2空格}}
- 命名：{{如 Python snake_case / JS camelCase / Go MixedCaps}}
- 注释/文档：{{如 中文 docstring / JSDoc 英文}}
- {{其他从 .editorconfig / eslint / pyproject.toml 或现有代码中观察到的约定}}

## 环境变量与配置

{{列出需要的环境变量/配置项，参考 .env.example / config.example.json 等模板文件}}
- {{变量名}}：{{用途}}{{是否必填，默认值}}

## 测试方式

{{怎么验证改动是对的：运行测试命令，或手动验证步骤}}

## Git 约定

- 主分支：main
- 远程仓库：{{仓库 URL}}
- 流程模式：简单（直接提交到 main）/ 完整（建分支 + PR）
- 提交规范遵循全局 github-collab skill
- {{项目特有的分支或发布约定，如无则删除此行}}

## GitHub 协作设置

- 流程：简单（默认，直接提交推送）/ 完整（建分支走 PR）
- 自动打 tag：否（仅无版本号文件的项目需要；有 version 字段的项目改版本号即自动打 tag）
- 说明：简单流程适合个人项目快速迭代；完整流程适合多人协作或正式项目
- 切换方式：对 AI 说"以后这个项目走完整流程""以后简单点""以后自动打 tag"即可

## 注意事项

- {{从 .gitignore、配置文件、代码中识别出的敏感文件、特殊配置、环境要求}}
- 不要提交：{{具体文件名/模式，如 .env、config.json、*.key}}
- {{已知限制或技术债，如能从代码/TODO中识别到}}
```

## 维护规则

- AGENTS.md 应随项目演进更新，改了架构/依赖/命令时同步修改
- AGENTS.md 可以提交到 GitHub（不含敏感信息），团队共享
- 如果项目有特殊的 AI 协作约定（如特定的代码生成规则），也写在里面
- 保持简洁，只写 AI 助手需要知道的信息，不写人类开发者文档（那是 README 的事）
