# EOP Dashboard 项目指令

⚠️ **重要！每次修改 EOP Dashboard 时，必须严格遵守以下要求！**

---

## 强制要求（必须100%执行）

### 1. 文件路径识别
- **项目文件**: `/Users/wangtao1/eop_dashboard.html`
- **部署目标**: https://wangtaooooo1109-coder.github.io/eop-dashboard/eop_dashboard.html
- **需求文档**: `/Users/wangtao1/REQUIREMENTS.md`

### 2. 版本号管理（强制）

每次修改必须同时更新两个位置：
1. JavaScript常量：`const VERSION = 'v{版本号}';`（约第1153行）
2. Navbar Badge：`<span id="versionBadge">v{版本号}</span>`（约第267行）

**⚠️ 两个版本号必须一致，不允许不同步！**

### 3. 每次修改后的必做步骤

#### 步骤1: 更新版本号
修改 `eop_dashboard.html` 中的版本号：
- 第1153行：`const VERSION = '{新版本号}';`
- 从当前版本号递增（如 v2.3.22 → v2.3.23）

#### 步骤2: Git提交
在 `/Users/wangtao1` 目录执行：
```bash
cd ~
git add eop_dashboard.html
git commit -m "v{版本号}: {简短描述}（用户需求原文）"
```

**必须包含**：
- 版本号
- 用户需求的原文或主要文字
- 具体修改内容

#### 步骤3: 推送到GitHub
```bash
cd ~
git push
```

---

## 仓库信息

```
- GitHub仓库: https://github.com/wangtaooooo1109-coder/eop-dashboard
- 在线访问: https://wangtaooooo1109-coder.github.io/eop-dashboard/eop_dashboard.html
- 部署: GitHub Actions 自动部署（每次 push 到 main 分支）
```

---

## Git提交模板

```
v{版本号}: {功能描述}

用户需求：{用户原始需求描述}

修改内容：
- {具体修改项1}
- {具体修改项2}
- ...

⚠️ 请确保以下内容已检查：
✓ navbar badge 版本号已更新（约第267行）
✓ const VERSION 常量已更新（约第1153行）
✓ 两个版本号一致

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

---

## 版本号示例

```
v2.3.22 → v2.3.23 → v2.3.24 → ...
```

格式：v{主版本}.{次版本}.{修订版本}

---

## 自动同步确认

每次提交后，检查：
1. GitHub 仓库是否有新 commit
2. GitHub Actions 是否成功运行
3. 在线地址是否更新（可能需要1-2分钟）

---

## 重要提醒

⚠️ **即使清空了上下文，当用户提及 EOP Dashboard 时：**

1. 首先读取此文件：`~/.claude/PROJECT_INSTRUCTIONS.md`
2. 读取需求文档：`~/REQUIREMENTS.md`
3. 确认要修改的文件路径
4. 修改后必须更新版本号
5. 必须同步到 GitHub

---

## 快速检查清单

修改完成后，检查以下项目：

- [ ] navbar badge 版本号已更新（约第267行）
- [ ] const VERSION 常量已更新（约第1153行）
- [ ] 两个版本号一致 ⚠️ 必须相同
- [ ] git add eop_dashboard.html
- [ ] git commit 包含版本号和用户需求
- [ ] git push 成功
- [ ] 验证 GitHub Actions 自动部署成功
