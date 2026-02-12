# EOP Dashboard 需求追溯

## 核心要求（必须每次都做）

### 1. 版本号管理
- ✅ 每次修改后更新版本号
- ✅ 页面显示的版本号必须与实际版本一致
- ✅ 版本号位置：navbar badge 和 const VERSION

### 2. Git提交规范
- ✅ 每次上传git时必须包含用户需求文字
- ✅ 做好追溯记录
- ✅ 提交信息必须包含具体修改内容

### 3. 自动同步
- ✅ 每次修改后自动同步到git
- ✅ 使用自动化脚本推送
- ✅ 目标地址：https://wangtaooooo1109-coder.github.io/eop-dashboard/eop_dashboard.html

## 功能需求历史

### 2026-02-12 - v2.3.0 Tab重组
- 创建6个Tab页面重新组织所有图表
- 仅修改UI布局，计算逻辑和数据处理逻辑完全不修改
- 添加Top N筛选框到零部件品类和供应商排名
- 将T1&理想端对比改为饼图

### 2026-02-12 - 修复问题
- 语法错误修复（line 2512）
- 缺失的函数调用（专用性分析图表）
- HTML ID不匹配修复
- 列名映射增强

### 当前问题
- 帕累托分析显示为空 - 待修复

## 技术约束
- 完全本地运行，数据不上传服务器
- 支持Excel文件格式：.xlsx, .xls
- 使用Bootstrap 5 + ECharts + SheetJS

## 部署信息
- GitHub仓库：https://github.com/wangtaooooo1109-coder/eop-dashboard
- 在线地址：https://wangtaooooo1109-coder.github.io/eop-dashboard/
- 自动部署：GitHub Actions（每次push到main分支）

### 2026-02-12 v2.3.1 - 修复帕累托分析
**用户需求：** 帕累托分析是空的

**问题分析：**
- colMap中缺少呆滞总数量字段的映射
- 导致帕累托分析无法读取数据

**修复内容：**
1. 添加呆滞总数量到colMap映射
2. 更新版本号从v2.3.0到v2.3.1
3. 确保页面显示版本号与代码一致

**测试验证：**
- [ ] 上传Excel文件后帕累托分析-零件显示正常
- [ ] 帕累托分析-供应商显示正常
- [ ] 版本号显示为v2.3.1

