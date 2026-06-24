---
name: github-skill-publisher
description: |
  WorkBuddy Skill 发布管理工具 - 将本地 WorkBuddy Skill 发布到 GitHub、更新版本、查询已发布仓库、删除仓库。
  触发词：发布skill、上传skill到github、更新skill版本、删除skill仓库、查看我的skill仓库、publish skill。
version: "1.3.0"
author: honghaoxiang
---

# GitHub Skill Publisher

管理 WorkBuddy Skill 在 GitHub 上的完整生命周期：发布、更新、查询、删除。

## 前置条件

每次执行前，**必须先运行上传前自查清单**（见下方「上传前自查清单」章节）。

核心依赖只有一个：**`gh` CLI 已安装并已登录 `hugohung` 账号**。

> ⚠️ WorkBuddy 内置了 `gh` CLI（路径见下方跨平台说明），通常不需要手动安装。
> 如果内置 `gh` 未登录，执行 `gh auth login` 按提示完成浏览器授权即可。

## 环境自动识别（双机协作）

用户有两台机器，agent 每次执行前必须先判断当前平台，再选对应路径和命令。

### 判断当前平台

```bash
python -c "import platform; print(platform.system())"
# 输出 Windows → 当前是公司Win PC
# 输出 Darwin  → 当前是个人Mac
```

或直接检测特征路径：

```bash
# 如果存在 .exe → Windows
ls ~/.workbuddy/bin/gh/bin/gh.exe 2>/dev/null && echo "Windows" || echo "macOS"
```

### 两台机器的环境差异

| 项目 | Windows（公司PC）| macOS（个人Mac）|
|------|-----------------|-----------------|
| 用户名 | honghaoxiang | honghaoxiang |
| HOME | `C:\Users\honghaoxiang` | `/Users/honghaoxiang` |
| gh 路径 | `~/.workbuddy/bin/gh/bin/gh.exe` | `~/.workbuddy/bin/gh/bin/gh` |
| git换行 | 注意 LF↔CRLF 问题，建议 `core.autocrlf=input` | 无问题 |
| skills 目录 | `C:\Users\honghaoxiang\.workbuddy\skills\` | `/Users/honghaoxiang/.workbuddy/skills/` |

### 跨平台 push 常见报错

| 报错信息 | 原因 | 修复 |
|---------|------|------|
| `403 Forbidden` | credential helper 未配置 | 按「上传前自查清单第二步」配置 |
| `Permission denied (publickey)` | 使用了 SSH 而非 HTTPS | 改用 HTTPS remote |
| `LF will be replaced by CRLF` | Windows 行尾符警告 | 正常，不影响功能；可设置 `git config core.autocrlf input` |
| `gh: command not found` | 系统 gh 未安装，但内置 gh 存在 | 使用绝对路径 `~/.workbuddy/bin/gh/bin/gh[.exe]` |
| `non-fast-forward` | 本地落后远端 | 先 `git pull --rebase origin main`，再 push |
| 冲突（rebase conflict）| 两台机器都做了修改 | 先 `git fetch && git reset --hard origin/main`，再重新应用本地改动 |

---

## Skill 同步清单（双机管理）

用户的所有 skill 都托管在 `hugohung` GitHub 账号下，两台机器共用同一套 GitHub 仓库。

### 从 GitHub 同步到本地（Mac→Win 或 Win→Mac 初始化）

```bash
# 在目标机器上，针对每个 skill 执行：
GH_PATH="~/.workbuddy/bin/gh/bin/gh[.exe]"
cd ~/.workbuddy/skills/<skill-name>
# 如果已有 .git
git pull origin main
# 如果全新机器，先 clone：
git clone https://github.com/hugohung/workbuddy-skill-<name>.git ~/.workbuddy/skills/<name>
```

### 我的 Skill 列表（所有需要管理的）

| Skill 名称 | GitHub 仓库 | 最新版本 | 备注 |
|---|---|---|---|
| drama-topic-research | workbuddy-skill-drama-topic-research | v2.1.0 | 系统自动管理，author=honghaoxiang |
| github-skill-publisher | workbuddy-skill-github-skill-publisher | v1.3.0 | 系统自动管理，author=honghaoxiang |
| frame-anim-tool | workbuddy-skill-frame-anim-tool | v2.6.0 | agent_created，author=honghaoxiang |
| jd-ai-short-drama-helper | workbuddy-skill-jd-ai-short-drama-helper | v1.0.0 | agent_created，author=honghaoxiang |
| jd-ai-short-drama-libtv | workbuddy-skill-jd-ai-short-drama-libtv | v1.1.0 | agent_created，author=honghaoxiang |
| jianying-rough-cut | workbuddy-skill-jianying-rough-cut | v1.0.0 | agent_created，author=honghaoxiang |
| skill-intro-writer | workbuddy-skill-skill-intro-writer | v1.0.0 | agent_created，author=honghaoxiang |
| lottie-anim-extension | lottie-anim-extension（⚠️ 非标命名）| v2.0.0 | agent_created，author=honghaoxiang |

> ⚠️ `lottie-anim-extension` 的 GitHub 仓库名为 `lottie-anim-extension`（不含 `workbuddy-skill-` 前缀），是历史命名，后续可考虑迁移。

### 哪些 Skill 需要上传到 GitHub？

- `author: honghaoxiang` + `agent_created: true` → **我创建的，需要上传并维护**
- `agent_created` 缺失或 `author` 非本人 → **系统内置或他人 skill，不需要 push**
- 判断命令：`grep -l "author: honghaoxiang" ~/.workbuddy/skills/*/SKILL.md`

### 哪些 Skill 需要从 GitHub 同步到本地？

换机器或新装 WorkBuddy 时，运行以下命令检查缺少哪些 skill：

```bash
# 列出 GitHub 上我所有的 skill 仓库
~/.workbuddy/bin/gh/bin/gh[.exe] repo list hugohung --limit 50 \
  --json name,url --jq '.[] | select(.name | startswith("workbuddy-skill-")) | .name'
```

然后逐一确认本地 `~/.workbuddy/skills/` 下是否存在对应目录。

---

## 跨平台说明

用户在两台设备上工作，执行前先检测当前平台：

| 平台 | 内置 `gh` 路径 | 检测命令 |
|------|----------------|----------|
| **Windows（公司PC）** | `~/.workbuddy/bin/gh/bin/gh.exe` | `ls ~/.workbuddy/bin/gh/bin/gh.exe` |
| **macOS（个人Mac）** | `~/.workbuddy/bin/gh/bin/gh` | `ls ~/.workbuddy/bin/gh/bin/gh` |

**路径策略：**
- 优先使用内置 `gh`（`~/.workbuddy/bin/gh/bin/gh*`），它已随 WorkBuddy 安装，且 token 由 WorkBuddy 管理
- 如果内置 `gh` 存在但未登录，提示用户执行 `~/.workbuddy/bin/gh/bin/gh auth login`
- 如果内置 `gh` 不存在，再尝试系统 `gh`（Windows: `gh` / macOS: `/usr/local/bin/gh`）

**`~` 在不同平台的实际路径：**
- Windows: `C:\Users\honghaoxiang`
- macOS: `/Users/honghaoxiang`

## 上传前自查清单 ⚠️ 必读

每次上传/更新 skill 到 GitHub 之前，必须按顺序完成以下检查。**跳过任何一步都可能导致 push 失败。**

### 第一步：确认 `gh` CLI 可用

```bash
# Windows
~/.workbuddy/bin/gh/bin/gh.exe auth status

# macOS
~/.workbuddy/bin/gh/bin/gh auth status
```

预期输出：`Logged in to github.com account hugohung ...`

如果未登录：执行 `~/.workbuddy/bin/gh/bin/gh.exe auth login`，选择 GitHub.com → HTTPS → 浏览器授权。

### 第二步：确认 git credential helper 已配置

这是最容易出问题的地方。`git push` 需要通过 credential helper 获取 token，如果没配置会报 `403` 或要求输入密码。

**检查当前配置：**
```bash
git config --global --list | grep credential
```

**如果输出为空或不是 `gh auth git-credential`，执行以下命令配置：**

```bash
# Windows（Git Bash 中使用正斜杠路径）
git config --global credential.helper '!"~/.workbuddy/bin/gh/bin/gh.exe" auth git-credential'

# macOS
git config --global credential.helper '!"~/.workbuddy/bin/gh/bin/gh" auth git-credential'
```

**验证配置是否生效：**
```bash
# Windows
echo -e "protocol=https\nhost=github.com\n" | ~/.workbuddy/bin/gh/bin/gh.exe auth git-credential get

# macOS
echo "protocol=https\nhost=github.com\n" | ~/.workbuddy/bin/gh/bin/gh auth git-credential get
```

预期输出包含 `username=hugohung` 和 `password=gho_...`。

### 第三步：确认 Skill 目录的 git remote 指向正确

```bash
cd ~/.workbuddy/skills/<skill-name>
git remote -v
```

预期输出：`https://github.com/hugohung/workbuddy-skill-<name>.git`

如果不是，重新设置：
```bash
git remote set-url origin https://github.com/hugohung/workbuddy-skill-<name>.git
```

### 第四步：先 pull 再 push（避免冲突）

```bash
cd ~/.workbuddy/skills/<skill-name>
git pull --rebase origin main
git push origin main
```

---

## 仓库命名规则

格式：`workbuddy-skill-<skill-name>`

例如：`drama-topic-research` → `workbuddy-skill-drama-topic-research`

## 仓库描述格式规则

所有 GitHub 仓库的 description 必须使用以下格式，让别人一眼看懂用途：

```
<emoji> <中文标题> | <中文功能说明>
```

**示例：**
- `🎬 京东AI短剧剧本助手 | 通过对话生成符合京东规范的短剧剧本、人物设定、场景设定和封面提示词`
- `🚀 GitHub Skill发布管理工具 | 管理WorkBuddy Skill在GitHub上的完整生命周期：发布、更新、查询、删除`
- `🔥 热门话题调研专家 | 自动搜集18个平台热榜，分析短剧适配度，生成题材推荐报告`

**常用 emoji 参考：**
- 🎬 影视/短剧相关
- 🛠️ 工具/开发相关
- 🚀 发布/部署相关
- 🔥 热门/趋势相关
- 📊 数据/分析相关
- 🎨 设计/图片相关
- 📝 文档/写作相关

创建仓库时直接使用此格式设置 `--description` 参数。如果已创建的仓库描述不符合此格式，使用 `gh repo edit` 更新。

## 操作流程

### 0. 上传前必做（每次都要执行）

**完整执行「上传前自查清单」（见上方），确认全部通过后再继续。**

### 1. 发布新 Skill（publish）

将本地 Skill 发布到 GitHub 公开仓库。

**步骤：**

1. 读取 `~/.workbuddy/skills/<skill-name>/SKILL.md` 获取 skill 的 name、description、version
2. 确认该 Skill 目录下有 `.git`，如果没有则 `git init`
3. 检查 GitHub 上是否已存在同名仓库（`gh repo view hugohung/workbuddy-skill-<name>`）
   - 如果已存在 → 进入更新流程
4. 在 Skill 目录下生成 `README.md`（如果不存在），模板见下方
5. 在 Skill 目录下生成 `LICENSE`（MIT），如果不存在
6. 配置 git 用户信息（如未配置）：
   ```bash
   git config --global user.name "honghaoxiang"
   git config --global user.email "hugohung@users.noreply.github.com"
   ```
7. 提交代码并推送：
   ```bash
   cd ~/.workbuddy/skills/<skill-name>
   git add .
   git commit -m "feat: init <skill-name> v<version>"
   git branch -M main
   gh repo create hugohung/workbuddy-skill-<name> --public --source=. --push --description "<emoji> <中文标题> | <中文功能说明>"
   ```
8. 创建 Release：
   ```bash
   gh release create v<version> --repo hugohung/workbuddy-skill-<name> --title "v<version>" --notes "<description>"
   ```
9. 输出仓库地址和 zip 下载链接：
   - 仓库：`https://github.com/hugohung/workbuddy-skill-<name>`
   - 下载：`https://github.com/hugohung/workbuddy-skill-<name>/archive/refs/tags/v<version>.zip`

### 2. 更新已发布 Skill（update）

更新已发布 Skill 的代码和版本。

**步骤：**

1. 读取 `SKILL.md` 获取当前 version
2. 询问用户新版本号（遵循 semver：patch/minor/major），或根据变更自动建议
3. 更新 `SKILL.md` 中的 version 字段
4. **执行上传前自查清单**（见上方），确认 git credential 配置正确
5. 提交并推送：
   ```bash
   cd ~/.workbuddy/skills/<skill-name>
   git add .
   git commit -m "<commit message>"
   git pull --rebase origin main   # 先拉取，避免冲突
   git push origin main
   ```
6. 创建新 Release：
   ```bash
   gh release create v<new-version> --repo hugohung/workbuddy-skill-<name> --title "v<new-version>" --notes "<更新说明>"
   ```
7. 输出新的 zip 下载链接

### 3. 查询已发布仓库（list）

查看所有已发布到 GitHub 的 Skill 仓库。

**步骤：**

1. 运行：
   ```bash
   gh repo list hugohung --limit 50 --json name,description,url --jq '.[] | select(.name | startswith("workbuddy-skill-"))'
   ```
2. 对比本地 `~/.workbuddy/skills/` 目录，标注哪些已发布、哪些未发布
3. 以表格形式展示：

| 本地 Skill | GitHub 仓库 | 版本 | 状态 |
|---|---|---|---|
| drama-topic-research | workbuddy-skill-drama-topic-research | v2.0.0 | ✅ 已发布 |
| png2apng | - | - | ⏳ 未发布 |

### 4. 删除仓库（delete）

删除 GitHub 上的 Skill 仓库。

**⚠️ 危险操作，必须二次确认！**

**步骤：**

1. 列出用户要删除的仓库信息
2. **明确警告**：删除后无法恢复，所有人将无法访问
3. 要求用户明确回复"确认删除 <仓库名>"
4. 确认后执行：
   ```bash
   gh repo delete hugohung/<repo-name> --yes
   ```
5. 告知用户本地 Skill 文件不受影响

### 5. 批量发布（batch-publish）

一次性发布多个未发布的 Skill。

**步骤：**
1. 扫描本地 `~/.workbuddy/skills/` 目录
2. 对比 GitHub 已发布仓库
3. 列出所有未发布的 Skill，让用户选择要发布哪些
4. 按顺序逐个执行发布流程

## README.md 模板

发布新 Skill 时，如果目录下没有 `README.md`，自动生成：

```markdown
# <Skill 显示名>

> WorkBuddy Skill — <一句话描述>

## 功能特性

- <从 SKILL.md 中提取核心功能点>

## 安装方式

### WorkBuddy 用户

1. 下载 [Release zip](../../releases/latest)
2. 在 WorkBuddy 技能管理 → 上传技能，选择 zip 文件

### 从源码安装

\`\`\`bash
git clone https://github.com/hugohung/workbuddy-skill-<name>.git ~/.workbuddy/skills/<name>
\`\`\`

## 使用方式

在 WorkBuddy 对话中直接说：
> "<触发示例>"

## License

MIT License
```

## MIT LICENSE 内容

```
MIT License

Copyright (c) <当前年份> honghaoxiang

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 错误处理

| 场景 | 处理方式 |
|---|---|
| `gh` 未安装 / 未找到 | 优先检查 `~/.workbuddy/bin/gh/bin/gh*`，再用系统 `gh` |
| 未登录 GitHub | 提示用户执行 `gh auth login`（浏览器授权） |
| **git push 报 403 / 要求输入密码** | **credential helper 未配置，按「上传前自查清单第二步」修复** |
| 仓库已存在 | 切换到更新流程而非创建 |
| git push 失败（non-fast-forward） | 先 `git pull --rebase`，再 `git push` |
| Release 标签已存在 | 提示用户使用新版本号 |
| Mac 上 `gh` 路径不对 | 检查 `~/.workbuddy/bin/gh/bin/gh`，注意 macOS 可执行文件无 `.exe` 后缀 |

## 重要规则

- 删除操作必须二次确认，不可跳过
- 版本号遵循 semver 规范（major.minor.patch）
- 每次发布/更新都应创建对应的 Release
- 不要修改用户 SKILL.md 的核心内容，只更新 version 字段
- 如果 Skill 目录已有 `.git` 和 remote，直接用现有配置 push
- **`git push` 前必须先执行「上传前自查清单」，不可跳过**
- **跨平台执行时，注意 `gh` 路径差异（Windows `.exe` / macOS 无后缀）**
