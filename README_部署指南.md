# EOP呆滞管控数据看板 - 飞书云文档嵌入指南

## 📋 方案概述

本指南提供三种方案将数据看板嵌入飞书云文档，推荐使用 **方案1（最简单）** 或 **方案2（最稳定）**。

---

## 🎯 方案1: 使用GitHub Pages（推荐）

### 优点
- ✅ 完全免费
- ✅ 操作简单，5分钟部署
- ✅ 自动HTTPS支持
- ✅ 稳定可靠
- ✅ 支持团队协作

### 部署步骤

#### 1. 创建GitHub仓库

1. 登录 [GitHub](https://github.com)
2. 点击右上角 `+` → `New repository`
3. 填写仓库信息：
   - Repository name: `eop-dashboard`
   - 选择 `Public`（公开仓库）
   - 勾选 `Add a README file`
4. 点击 `Create repository`

#### 2. 上传看板文件

1. 进入刚创建的仓库
2. 点击 `Add file` → `Upload files`
3. 将 `eop_dashboard.html` 拖拽到页面
4. 点击 `Commit changes`

#### 3. 启用GitHub Pages

1. 进入仓库的 `Settings` 页面
2. 在左侧菜单找到 `Pages`
3. 在 `Source` 下拉框选择 `main` 分支
4. 点击 `Save`
5. 等待1-2分钟，页面会显示访问链接：
   ```
   https://你的用户名.github.io/eop-dashboard/eop_dashboard.html
   ```

#### 4. 在飞书中嵌入

**方法A: 嵌入网页卡片**
1. 在飞书云文档中，输入 `/` 调出菜单
2. 选择 `网页` 或 `链接`
3. 粘贴GitHub Pages链接
4. 其他人点击即可访问看板

**方法B: 使用iframe嵌入（如果飞书支持）**
1. 在飞书云文档中输入以下代码：
```html
<iframe src="https://你的用户名.github.io/eop-dashboard/eop_dashboard.html"
        width="100%"
        height="800px"
        frameborder="0">
</iframe>
```

---

## 🚀 方案2: 使用Vercel/Netlify（最简单）

### Vercel部署（推荐）

#### 1. 注册Vercel账号
访问 [vercel.com](https://vercel.com) 并使用GitHub账号登录

#### 2. 部署项目
1. 点击 `New Project`
2. 导入GitHub仓库（或直接拖拽HTML文件）
3. 点击 `Deploy`
4. 等待30秒，获得访问链接：
   ```
   https://你的项目名.vercel.app/eop_dashboard.html
   ```

#### 3. 在飞书中使用
与方案1相同，使用网页卡片或iframe嵌入

### 优点
- ⚡ 部署速度快（30秒）
- 🌍 全球CDN加速
- 🔄 自动更新
- 📊 访问统计

---

## 💻 方案3: 本地/内网服务器部署

### 适用场景
- 数据敏感，不能公开
- 公司内网环境
- 需要更高安全性

### 部署步骤

#### 使用Python快速启动（最简单）

1. 打开终端，进入文件所在目录：
```bash
cd /Users/wangtao1
```

2. 启动HTTP服务器：
```bash
# Python 3
python3 -m http.server 8080

# 或者 Python 2
python -m SimpleHTTPServer 8080
```

3. 访问地址：
```
http://localhost:8080/eop_dashboard.html
```

4. 如果在公司内网，使用：
```
http://你的IP地址:8080/eop_dashboard.html
```

#### 使用Node.js部署

```bash
# 安装http-server
npm install -g http-server

# 启动服务
http-server -p 8080

# 访问
http://localhost:8080/eop_dashboard.html
```

---

## 📱 飞书云文档嵌入方法详解

### 方法1: 网页卡片（推荐）

1. 在飞书云文档中输入 `/`
2. 选择 `网页` 或 `网页卡片`
3. 粘贴看板URL
4. 设置卡片标题：`EOP呆滞管控数据看板`
5. 点击保存

**效果**:
- 显示为可点击的卡片
- 点击后在新标签页打开
- 支持预览缩略图

### 方法2: iframe嵌入（如支持）

在飞书云文档的代码块中输入：

```html
<div style="width: 100%; height: 800px;">
    <iframe
        src="你的看板URL"
        width="100%"
        height="100%"
        frameborder="0"
        style="border: 1px solid #e0e0e0; border-radius: 10px;">
    </iframe>
</div>
```

### 方法3: 超链接

直接在文档中添加：
```
[📊 打开EOP呆滞管控数据看板](你的看板URL)
```

---

## 🔐 数据安全说明

### 公开部署（GitHub Pages/Vercel）
- ✅ 适合: 内部分析数据、非敏感信息
- ⚠️ 注意: 任何有链接的人都可以访问
- 💡 建议: 不要在URL中包含敏感参数

### 私有部署（内网服务器）
- ✅ 数据完全在内网
- ✅ 通过VPN访问
- ✅ 公司防火墙保护

### 增强安全性
1. **添加访问密码**（需要修改HTML）
2. **使用Vercel的密码保护功能**
3. **配置GitHub私有仓库** + Vercel部署

---

## 📊 使用Excel数据的建议

### 方案A: 手动上传（当前方式）
- 每次访问看板时上传Excel文件
- 数据不会保存在服务器
- 完全在浏览器中处理

### 方案B: 将Excel转为JSON（高级）
1. 使用Python脚本将Excel转为JSON
2. 将JSON文件上传到GitHub
3. 修改看板代码自动加载JSON
4. 适合数据更新不频繁的场景

---

## 🎨 自定义域名（可选）

### GitHub Pages自定义域名
1. 购买域名（如: `dashboard.yourcompany.com`）
2. 在GitHub仓库Settings → Pages中设置
3. 配置DNS记录
4. 使用自定义域名访问

### Vercel自定义域名
1. 在Vercel项目设置中添加域名
2. 配置DNS
3. 自动配置HTTPS

---

## ⚙️ 维护和更新

### 更新看板内容
1. 修改本地的 `eop_dashboard.html`
2. 上传到GitHub（或重新部署到Vercel）
3. 刷新飞书中的链接即可看到更新

### 版本管理
- 使用Git管理代码版本
- 每次更新添加版本说明
- 便于回滚到历史版本

---

## 🆘 常见问题

### Q1: 飞书不支持iframe怎么办？
**答**: 使用网页卡片或超链接方式，点击后在新标签页打开。

### Q2: 看板加载慢怎么办？
**答**:
- 使用Vercel代替GitHub Pages（有CDN）
- 压缩HTML文件
- 优化图表库加载

### Q3: 多人同时访问会冲突吗？
**答**: 不会。每个人访问的是独立的网页实例，数据在各自浏览器中处理。

### Q4: 如何设置访问权限？
**答**:
- **公开部署**: 只分享链接给需要的人
- **私有部署**: 使用Vercel密码保护或内网部署
- **企业方案**: 集成飞书身份认证（需要开发）

### Q5: Excel数据安全吗？
**答**:
- 数据只在用户浏览器中处理
- 不会上传到任何服务器
- HTML文件本身不包含数据

---

## 📞 技术支持

### 推荐配置
- **个人/小团队**: GitHub Pages（免费）
- **中型团队**: Vercel（免费，性能更好）
- **企业/敏感数据**: 内网服务器部署

### 进阶需求
如需以下功能，需要额外开发：
- ✨ 多人协作编辑
- 🔐 飞书账号登录
- 💾 数据自动保存
- 📧 定时邮件报告
- 📱 移动端适配

---

## 🎯 快速开始（5分钟）

1. **注册GitHub账号** → [github.com](https://github.com)
2. **创建新仓库** → 命名为 `eop-dashboard`
3. **上传HTML文件** → 拖拽 `eop_dashboard.html`
4. **启用Pages** → Settings → Pages → main分支
5. **复制链接** → 粘贴到飞书云文档

完成！🎉

---

## 📝 总结

| 方案 | 难度 | 时间 | 成本 | 推荐度 |
|------|------|------|------|--------|
| GitHub Pages | ⭐⭐ | 5分钟 | 免费 | ⭐⭐⭐⭐⭐ |
| Vercel | ⭐ | 2分钟 | 免费 | ⭐⭐⭐⭐⭐ |
| 内网服务器 | ⭐⭐⭐ | 10分钟 | 免费 | ⭐⭐⭐ |

**推荐**: 先用GitHub Pages测试，满意后可以迁移到Vercel获得更好性能。
