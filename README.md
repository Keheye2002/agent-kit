# agent-kit

**把常用 Skills 接到你的 Agent，把工程规范带进新项目。**

一个个人开发工具箱：整理跨 Agent 的 skills，也为 Gin 和 React 项目提供携带编码规范、测试约定和提交检查的启动模板。

[快速开始](#快速开始) · [Skills](#skills) · [启动模板](#启动模板) · [维护与扩展](#维护与扩展)

<p align="center">
  <img src="assets/readme/hero.svg" width="100%" alt="agent-kit 的两条路径：skills 和 vendor 经 install.sh 链接到 Agent；stacks 经 new.sh 复制到新项目。">
</p>

**给 Agent 装工具，用软链；给新项目带规范，用文件。** Skills 面向 Cursor、Codex CLI、Claude Code 等遵循 [Agent Skills](https://github.com/anthropics/skills) 约定的工具；模板把规范留在项目里，随代码一起维护。

## 快速开始

先克隆到稳定的本地路径，预览将安装的 skills，再创建软链：

```bash
git clone --recurse-submodules https://github.com/b1ngsha/agent-kit.git ~/Documents/code/agent-kit
cd ~/Documents/code/agent-kit
./install.sh dry-run
./install.sh link
```

安装器使用已有的 Agent 配置目录，跳过同名真实文件或目录。由 `npx skills` 管理的副本继续保留，其余按仓库来源建立软链。详细行为见下方[安装与更新](#安装与更新)。

然后在支持 skill 调用的 Agent 中按任务选择，例如用 `/think` 推敲方案，完成实现后用 `/check` 自检。这些步骤由你手动切换。

新项目则走另一条入口：

```bash
./new.sh gin my-service
./new.sh react my-app
```

Gin 需要 Go；React 需要 pnpm，模板脚本也会使用 Python 3。脚手架会下载依赖，目标目录必须尚不存在。

## Skills

从一个明确任务开始，按需选择 skill。

| 你在做什么 | 入口 |
| --- | --- |
| 推敲需求与方案 | [think](vendor/Waza/plugins/waza/skills/think/) |
| 交付前审查代码 | [check](vendor/Waza/plugins/waza/skills/check/) |
| 排查报错和异常 | [hunt](vendor/Waza/plugins/waza/skills/hunt/) |
| 设计页面与组件 | [ui](vendor/Waza/plugins/waza/skills/ui/) |
| 阅读、研究、写作 | [read](vendor/Waza/plugins/waza/skills/read/) · [learn](vendor/Waza/plugins/waza/skills/learn/) · [write](vendor/Waza/plugins/waza/skills/write/) |
| 制作 PDF、简历与幻灯片 | [kami](vendor/kami/plugins/kami/skills/kami/) |

### 自有 skills

- **[find-simplifications](skills/find-simplifications/)**：扫描死代码、重复和过度设计，提出可落地的简化方案。
- **[my-doc-style](skills/my-doc-style/)**：按个人写作习惯编写中文 README、测试指引、操作手册和 wiki。
- **[release-tag](skills/release-tag/)**：分析 diff、选择版本、生成 changelog，并交互式完成 tag 与推送。

<details>
<summary>查看 Waza 与 Kami 的使用说明</summary>

[Waza](vendor/Waza/) 围绕工程习惯提供独立 skills。除上表入口外，[health](vendor/Waza/plugins/waza/skills/health/) 用于排查 Agent 规则、hook 和 MCP 配置问题。每个 skill 有明确触发场景，用户决定何时调用，下面是手动组合示例：

```text
/think → 实施 → /check → 合并
/read → /learn → /write
/hunt → 修复 → /check
```

[Kami](vendor/kami/) 为简历、一页纸、白皮书、信件、作品集、幻灯片、研报和更新日志提供「暖米色 + 墨蓝」排版。可用「做 PDF」「排版」「简历」「PPT」等短语触发。

字体与 WeasyPrint、Python 等构建依赖按 [Kami 上游说明](https://github.com/tw93/kami) 安装；字体缺失时，上游脚本会尝试从 jsDelivr 补齐。

</details>

<details>
<summary>查看完整第三方来源</summary>

第三方 skills 通过 git submodule 收录。`vendor/.allowlists/<submodule>` 决定安装范围；没有白名单的子模块收录其全部 skills。

| 来源 | 收录方向 | 上游 |
| --- | --- | --- |
| [Waza](vendor/Waza/) | 方案、界面、审查、排错、阅读与写作 | [tw93/Waza](https://github.com/tw93/Waza) |
| [kami](vendor/kami/) | 文档排版 | [tw93/kami](https://github.com/tw93/kami) |
| [mattpocock-skills](vendor/mattpocock-skills/) | code-review、tdd、research、prototype、domain-modeling 等 | [mattpocock/skills](https://github.com/mattpocock/skills) |
| [anthropics-skills](vendor/anthropics-skills/) | frontend-design | [anthropics/skills](https://github.com/anthropics/skills) |
| [yutto-dev-yutto](vendor/yutto-dev-yutto/) | bilibili-video-download | [yutto-dev/yutto](https://github.com/yutto-dev/yutto) |
| [humanizer-zh](vendor/humanizer-zh/) | humanizer-zh | [op7418/Humanizer-zh](https://github.com/op7418/Humanizer-zh) |
| [larksuite-meegle-cli](vendor/larksuite-meegle-cli/) | meegle | [larksuite/meegle-cli](https://github.com/larksuite/meegle-cli) |
| [larksuite-cli](vendor/larksuite-cli/) | 白名单内的 lark-* skills | [larksuite/cli](https://github.com/larksuite/cli) |
| [vercel-labs-skills](vendor/vercel-labs-skills/) | find-skills | [vercel-labs/skills](https://github.com/vercel-labs/skills) |
| [brianlovin-claude-config](vendor/brianlovin-claude-config/) | simplify | [brianlovin/claude-config](https://github.com/brianlovin/claude-config) |
| [humanlayer-skills](vendor/humanlayer-skills/) | show-me | [humanlayer/skills](https://github.com/humanlayer/skills) |
| [cursor-plugins](vendor/cursor-plugins/) | thermo-nuclear-code-quality-review | [cursor/plugins](https://github.com/cursor/plugins) |
| [yetone-kill-ai-slop](vendor/yetone-kill-ai-slop/) | kill-ai-slop | [yetone/kill-ai-slop](https://github.com/yetone/kill-ai-slop) |
| [native-feel-skill](vendor/native-feel-skill/) | 单 skill，全量收录 | — |

</details>

## 启动模板

`new.sh` 先运行官方工具创建项目，再复制对应技术栈的规范和配置。两种模板都有 `AGENTS.md`、`CODING_STANDARDS.md`、按需阅读的测试规范和 pre-commit 配置。

### Gin · HTTP 服务

```bash
./new.sh gin my-service
```

包含 `GET /healthz`、golangci-lint、Makefile，以及 [Go 测试](stacks/gin/docs/standards/go-testing.md)、[API 契约](stacks/gin/docs/standards/api-contract.md)和[数据库](stacks/gin/docs/standards/database.md)规范。

### React · Web 应用

```bash
./new.sh react my-app
```

使用 Vite、React、TanStack Router 和 Query，包含 pnpm、oxlint / oxfmt、Vitest，以及[代码](stacks/react/docs/standards/web-code.md)、[路由](stacks/react/docs/standards/web-routing.md)和[测试](stacks/react/docs/standards/web-testing.md)规范。

新项目中的规范可以直接阅读、修改和提交：

```text
my-project/
├── AGENTS.md
├── CODING_STANDARDS.md
├── .pre-commit-config.yaml
├── scripts/check-file-tokens.sh
└── docs/standards/
```

提交检查包含基础文件检查、格式与 lint，以及手写文本文件的 **5000 token 估算上限**（按字节数 ÷ 4）；React 还会运行类型检查和测试。安装 [pre-commit](https://pre-commit.com/) 及对应栈的检查工具后，在生成项目里执行 `make setup-hooks`（Gin）或 `pnpm setup-hooks`（React）启用 hooks。

模板仅收录从 x-data 提炼的通用规范，背景见 [ADR](docs/adr/)。已有目标目录会被拒绝覆盖，Go 模块路径使用目标目录名。

## 维护与扩展

### 安装与更新

```bash
./install.sh              # 更新仓库和 vendor，再重链
./install.sh link         # 仅重链，不联网
./install.sh dry-run      # 查看计划，不写入
./install.sh uninstall    # 移除本脚本管理的软链
```

从旧目录结构更新后，先运行 `./install.sh link`，将自有 skill 的旧软链修复到 `skills/`。

<details>
<summary>安装行为、Agent 安装 prompt 与更新细节</summary>

安装器处理 `~/.agents/skills/`、`~/.claude/skills/`、`~/.cursor/skills/` 和 `~/.codex/skills/`，仅在对应 Agent 的父目录已存在时创建目标目录。

- 自有 skills 优先于同名 vendor skills；vendor 按 allowlist 筛选。
- `npx skills` 锁文件中记录的副本继续由 npx 管理，其他 Agent 优先链接到 `~/.agents/skills/<name>`。
- 未由 npx 管理的 skills 链接到本仓库源文件；更新这些文件后，软链指向的内容随之更新。
- 同名真实目录或文件会跳过。卸载只移除脚本识别的软链。

也可以把以下 prompt 交给目标 Agent：

```text
Install agent-kit skills for me:
1. Clone https://github.com/b1ngsha/agent-kit with --recurse-submodules
   to a stable path such as ~/Documents/code/agent-kit.
2. Run ./install.sh dry-run and inspect the planned links.
3. Run ./install.sh link to create links using its vendor allowlists
   and npx ownership rules. Preserve existing real files and directories.
4. Report the install path and installed skills.
```

默认 `./install.sh` 执行 `git pull --ff-only`、刷新 vendor 子模块并重链。离线重链用 `./install.sh link`，或 `SKIP_UPDATE=1 ./install.sh`。

忘记递归克隆时，运行：

```bash
git submodule update --init --recursive
```

更新不会自动提交 vendor 指针。需要固定上游版本时，检查并提交相应子模块改动；调整收录范围后，编辑 allowlist，再运行 `./install.sh link`。

</details>

<details>
<summary>目录结构与新增 skill</summary>

```text
agent-kit/
├── skills/                    # 自有 skills
├── vendor/                    # 第三方 submodules 与白名单
├── stacks/                    # Gin、React 模板和工程规范
├── assets/readme/             # 首页视觉素材
├── scripts/sync-to-remote.sh   # 远程同步
├── docs/adr/                  # 设计决策
├── install.sh                 # 安装入口
└── new.sh                     # 新建项目入口
```

新增自有 skill 放在 `skills/<name>/SKILL.md`，包含 YAML frontmatter（`name`、`description`）与正文；按需添加 `references/`。参考 [find-simplifications](skills/find-simplifications/SKILL.md)，然后运行 `./install.sh link`。

远程同步使用 `./scripts/sync-to-remote.sh`，配置示例为 [sync-remote.conf.example](sync-remote.conf.example)，实际配置仍放在仓库根目录的 `sync-remote.conf`。

</details>

## License

MIT
