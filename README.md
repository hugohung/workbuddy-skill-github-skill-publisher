# GitHub Skill Publisher

> WorkBuddy Skill — 管理 WorkBuddy Skill 在 GitHub 上的完整生命周期：发布、更新、查询、删除

## 功能特性

- 📤 **发布** — 一键将本地 Skill 发布到 GitHub 公开仓库，自动生成 README、LICENSE、Release
- 🔄 **更新** — 修改后推送新版本，自动创建新 Release
- 📋 **查询** — 查看所有已发布/未发布的 Skill 状态
- 🗑️ **删除** — 安全删除 GitHub 仓库（二次确认）
- 📦 **批量发布** — 一次发布多个未发布的 Skill

## 安装方式

### WorkBuddy 用户

1. 下载 [Release zip](../../releases/latest)
2. 在 WorkBuddy 技能管理 → 上传技能，选择 zip 文件

### 从源码安装

```bash
git clone https://github.com/hugohung/workbuddy-skill-github-skill-publisher.git ~/.workbuddy/skills/github-skill-publisher
```

## 使用方式

在 WorkBuddy 对话中直接说：

- "帮我把 xxx skill 发布到 GitHub"
- "更新 xxx skill 到 v1.1.0"
- "我有哪些 Skill 还没发布？"
- "删除 xxx 仓库"

## 前置条件

- 已安装 `gh` CLI（`brew install gh`）
- 已登录 GitHub（`gh auth login`），账号为 hugohung

## License

MIT License
