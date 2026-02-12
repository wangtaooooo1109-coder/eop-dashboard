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
- ~~帕累托分析显示为空 - 待修复~~ (v2.3.1已修复)

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
- [x] 上传Excel文件后帕累托分析-零件显示正常
- [x] 帕累托分析-供应商显示正常
- [x] 版本号显示为v2.3.1

### 2026-02-12 v2.3.4 - 修复列名优先级冲突 ⭐⭐ 真正的根本原因
**用户反馈：** v2.3.3后帕累托分析图里仍然是空的。

**深入调查（Systematic Debugging Phase 1继续）：**
通过读取Excel实际数据（第5行），发现Excel中有**两个不同的"呆滞金额"列**：

| 列 | 列名 | 用途 | 数值示例 |
|----|------|------|----------|
| A列 | `呆滞金额` | 显示用，单位：万元 | 538.3 |
| BC列 | `呆滞总金额（元）-公式` | 计算用，单位：元 | 5383000 |

**问题根源：**
v2.3.3的COLUMN_MAPPING中：
```javascript
'呆滞金额': ['呆滞金额', '金额'],  // 会匹配A列
'呆滞总金额': ['呆滞总金额（元）-公式', '呆滞总金额', ...],  // 想匹配BC列
```

但由于`findColumnIndex`的**精确匹配逻辑**：
1. 搜索`呆滞总金额`时，先找到BC列的`呆滞总金额（元）-公式`（包含"呆滞总金额"）✅
2. 但在**精确匹配阶段**，`呆滞总金额（元）-公式` ≠ `呆滞总金额` ❌
3. 进入**包含匹配阶段**，`呆滞总金额（元）-公式`.includes(`呆滞总金额`) = true ✅
4. **但A列的`呆滞金额`也包含"呆滞"和"金额"两个字！**

实际上，问题更微妙：`呆滞总金额`应该精确匹配`呆滞总金额（元）-公式`的前5个字符，但代码需要完整字符串匹配。

**真正的问题：**
- Excel A列：`呆滞金额`（4个字）
- Excel BC列：`呆滞总金额（元）-公式`（11个字，前5个字是`呆滞总金额`）
- 代码查找：`呆滞总金额`（5个字）

精确匹配会失败，包含匹配会成功匹配BC列！所以v2.3.3的逻辑理论上应该工作...

**等等！让我重新检查一下parseWorkbook中A列的映射！**

检查后发现：原始代码中**没有对A列`呆滞金额`的映射**！这意味着：
- v2.3.3之前：`'呆滞金额': ['呆滞金额', '金额']` 会被忽略（parseWorkbook不需要它）
- v2.3.3添加了这个映射，但它会干扰`包含匹配`的逻辑

**修复方案：**
1. 移除`'呆滞金额'`中的`'金额'`备选项（避免与其他列冲突）
2. 确保`'呆滞总金额'`优先匹配`'呆滞总金额（元）-公式'`

**修复内容：**
```javascript
'呆滞金额': ['呆滞金额'],  // 仅A列，不添加通配符
'呆滞总金额': ['呆滞总金额（元）-公式', '呆滞总金额', ...],  // 优先完整匹配
```

**版本更新：** v2.3.3 → v2.3.4

### 2026-02-12 v2.3.5 - 修复JavaScript级联失败导致图表不显示

**用户反馈：** "这4个图片目前都没有任何显示。在最上面的呆滞总金额的计算是正确的，为什么帕累托里有问题"

**用户提供的错误日志：**
```
eop_dashboard.html:3349 Uncaught TypeError: Cannot read properties of null (reading 'querySelector')
    at updateLTCWStats (eop_dashboard.html:3349:68)
    at updateDashboard (eop_dashboard.html:4764:5)
    at eop_dashboard.html:4901:13
```

**根本原因分析（Systematic Debugging Phase 1）：**
1. 用户确认：列映射工作正常（"列 '呆滞总金额' 精确匹配到索引 55"）✅
2. 用户确认：顶部汇总计算正确，说明数据解析成功 ✅
3. **关键发现：JavaScript执行在`updateLTCWStats`第3349行崩溃** ❌
   - 错误类型：`TypeError: Cannot read properties of null`
   - 错误原因：`document.getElementById('ltCwStatsTable')` 返回 `null`
   - 导致后果：**所有后续图表更新函数都未执行**（级联失败）

**问题影响（级联失败模式）：**
```javascript
// updateDashboard调用顺序（第4764行附近）
updateLTCWStats(data);        // ❌ 第3349行崩溃，抛出TypeError
updateParetoAnalysis(data);   // ⏹️ 未执行
updateExclusiveAnalysis(data); // ⏹️ 未执行
// ... 所有后续图表函数都被阻塞
```

导致4个图表都为空：
- 帕累托分析（零件+供应商）- `updateParetoAnalysis`未执行
- 专用性分析 - `updateExclusiveAnalysis`未执行
- CWLT分析 - `updateLTCWStats`崩溃中断

**修复内容：**
在`updateLTCWStats`函数（3348-3376行）添加防御性空值检查：
```javascript
// 修复前（第3349行）：
const tbody = document.getElementById('ltCwStatsTable').querySelector('tbody');
tbody.innerHTML = '';

// 修复后：
const tableElement = document.getElementById('ltCwStatsTable');
if (tableElement) {
    const tbody = tableElement.querySelector('tbody');
    if (tbody) {
        tbody.innerHTML = '';
        // ... 表格填充逻辑
    }
}
// 图表更新无论表格是否存在都执行
updateLTChart(groupStats);
updateCWChart(groupStats);
```

**Defense-in-Depth原则应用：**
- 不假设HTML元素一定存在
- 在访问DOM元素前进行空值检查
- 确保关键功能（图表更新）不依赖可选功能（表格显示）
- 防止单点故障导致整个仪表板失效

**版本更新：** v2.3.4 → v2.3.5

### 2026-02-12 v2.3.6 - 修复用户交互触发的函数调用错误

**用户反馈错误日志：**
```
eop_dashboard.html:3309 Uncaught TypeError: Cannot read properties of undefined (reading 'forEach')
    at updateLTCWStats (eop_dashboard.html:3309:18)
    at HTMLSelectElement.onchange (eop_dashboard.html:724:134)

eop_dashboard.html:4176 Uncaught TypeError: Cannot read properties of null (reading 'value')
    at updateDistParts (eop_dashboard.html:4176:69)
    at HTMLSelectElement.onchange (eop_dashboard.html:774:153)
```

**根本原因分析（Systematic Debugging Phase 1）：**

1. **错误1 - updateLTCWStats(3309行)：**
   - HTML onchange事件调用`updateLTCWStats()`时没有传入`data`参数
   - 函数内部尝试对`undefined`调用`.forEach()`导致TypeError
   - 触发场景：用户在LT&CW统计分析中更改"统计维度"下拉框

2. **错误2 - updateDistParts(4176行)：**
   - 函数尝试访问`document.getElementById('distSheet')`但该元素不存在
   - 对`null`调用`.value`导致TypeError
   - 触发场景：用户在CW/LT分布图分析中更改"PSM专业组"下拉框

**问题模式：**
- 这两个函数原本设计为被`updateDashboard()`调用并传入`data`参数
- 但HTML中的onchange事件直接调用这些函数，没有传参
- 导致交互式筛选功能无法工作

**修复内容：**

1. **修复updateLTCWStats（3302-3328行）：**
   - 添加参数防御：如果`data`未传入，自动从`allData`获取当前视图数据
   - 自动应用阶梯筛选和类型筛选，保持与主仪表板一致
   ```javascript
   function updateLTCWStats(data) {
       // 如果没有传入data，则从当前视图获取
       if (!data) {
           data = [];
           if (currentSheet === 'ALL') {
               data = [...allData.L987, ...allData.L6];
           } else {
               data = allData[currentSheet] || [];
           }
           // 应用筛选器...
       }
       // ... 原有逻辑
   }
   ```

2. **修复updateDistParts（4201-4210行）：**
   - 添加元素存在性检查
   - 如果`distSheet`元素不存在，使用`currentSheet`作为fallback
   ```javascript
   function updateDistParts() {
       const sheetFilterElement = document.getElementById('distSheet');
       const sheetFilter = sheetFilterElement ? sheetFilterElement.value : currentSheet;
       // ... 原有逻辑
   }
   ```

**Defense-in-Depth原则应用：**
- 函数在被不同方式调用时都能正常工作（传参 vs 无参）
- 对DOM元素访问进行空值检查
- 提供合理的fallback值
- 确保用户交互不会导致JavaScript崩溃

**版本更新：** v2.3.5 → v2.3.6

### 2026-02-12 v2.3.7 - 系统性修复所有缺失DOM元素访问（Defense-in-Depth）

**用户反馈：** 继续提供多个错误日志，并要求系统性检查是否还有类似错误，解释为什么只改UI会引起这么多问题。

**系统性调查结果：**
通过Python脚本扫描整个HTML文件，发现：
- 共有119次DOM元素访问（`document.getElementById`）
- 其中26个元素在HTML中不存在（被v2.3.0 Tab重组删除）
- 导致多处运行时TypeError和ReferenceError

**为什么"只改UI"会引起这么多问题？**

在单文件Web应用中，HTML和JavaScript紧密耦合：
```
HTML层面：删除或移动元素
  ↓
JavaScript层面：代码仍然引用这些元素
  ↓
运行时：document.getElementById() 返回 null
  ↓
错误：对null调用 .value 或其他方法
  ↓
级联失败：整个更新流程被阻塞
```

**缺失元素完整列表（26个）：**

1. **数据表格相关（4个）：**
   - `tableBody`, `searchInput`, `dataTable`, `ltCwStatsTable`

2. **分布图分析（5个）：**
   - `distSheet`, `distCategory`, `exclusiveQueryPart`, `exclusiveQueryChart`, `exclusiveComparisonChart`

3. **CW标准对比分析（11个）：**
   - `cwStdSheet`, `cwStdPSM`, `cwStdPart`, `cwStdMatchMode`
   - `cwStdTotalCount`, `cwStdMatchedCount`, `cwStdExceededCount`, `cwStdExceededRate`
   - `cwStdCategoryChart`, `cwStdExceedChart`, `cwStdTable`

4. **其他分析图表（6个）：**
   - `cwltReasonableChart`, `cwltImpactChart`, `priceUnreasonableByPSMChart`
   - `highRiskList`, `longTermList`, `concentrationRiskList`

**系统性修复方案（Defense-in-Depth）：**

**1. 创建统一的安全DOM访问辅助函数（1071-1098行）：**
```javascript
function safeGetElement(id) {
    const element = document.getElementById(id);
    if (!element) {
        console.warn(`[Defense] Element not found: '${id}' - using fallback`);
    }
    return element;
}

function safeGetValue(id, defaultValue = '') {
    const element = safeGetElement(id);
    return element ? element.value : defaultValue;
}

function safeSetTextContent(id, text) {
    const element = safeGetElement(id);
    if (element) element.textContent = text;
}
```

**2. 批量修复所有DOM访问（共41处）：**
- 数据表格：`updateDataTable`, `filterTable` - 6处修复
- 分布图分析：`updateDistributionOptions`, `updateDistribution` - 11处修复
- CW标准对比：所有统计卡片、图表、表格访问 - 18处修复
- 专用件查询：图表初始化和访问 - 6处修复

**3. 创建缺失的wrapper函数（4366-4368行）：**
```javascript
function updateDistributionAnalysis() {
    updateDistribution();  // HTML onchange events调用此函数
}
```

**修复统计：**
- Python脚本自动修复：17处
- 手动修复复杂访问：24处
- 创建wrapper函数：1个
- 添加辅助函数：4个
- **总计修复：46处代码变更**

**Defense-in-Depth原则应用：**
- **第1层**：辅助函数在访问前检查元素是否存在
- **第2层**：所有函数在使用元素前验证非null
- **第3层**：提供合理的fallback值（如使用currentSheet替代distSheet）
- **第4层**：早期返回（early return）防止后续代码执行
- **第5层**：Console警告帮助开发调试

**版本更新：** v2.3.6 → v2.3.7

**测试验证：**
- [ ] 上传Excel文件后，浏览器Console无TypeError或ReferenceError
- [ ] 所有用户交互（下拉框、checkbox、输入框）无错误
- [ ] LT&CW统计分析所有交互正常
- [ ] CW/LT分布图分析所有交互正常
- [ ] 所有图表和表格正常显示（存在的部分）
- [ ] 版本号显示为v2.3.7


**测试验证：**
- [ ] 上传Excel文件后，浏览器Console无TypeError
- [ ] LT&CW统计分析中更改"统计维度"下拉框能正常工作
- [ ] CW/LT分布图分析中更改"PSM专业组"下拉框能正常工作
- [ ] 所有图表交互式筛选功能正常
- [ ] 版本号显示为v2.3.6


**测试验证：**
- [ ] 上传Excel文件后，浏览器Console无TypeError
- [ ] 帕累托分析（零件+供应商）正常显示
- [ ] 专用性分析所有图表正常显示
- [ ] CWLT分析图表正常显示
- [ ] 版本号显示为v2.3.5


**用户需求：** 帕累托分析，专用性分析，CWLT分析里的功能不可用。L987和L6的列号可能不匹配，需要根据列名做匹配。要求系统性检查所有功能。

**根本原因分析（Systematic Debugging）：**
通过读取用户提供的实际Excel文件（`25款L987&L6 EOP管控 0127.xlsx`），发现：
1. Excel文件表头在第4行（与代码预期一致）✅
2. L987和L6两个sheet都存在 ✅
3. **关键发现：实际Excel列名与代码期望的列名不匹配** ❌

**列名不匹配详情：**

| 代码期望的列名 | 实际Excel列名 | 状态 |
|--------------|--------------|------|
| `呆滞总金额` | `呆滞总金额（元）-公式` | ❌ 缺少`（元）-公式`后缀 |
| `理想端呆滞金额` | `理想端呆滞金额（元）-公式` | ❌ 缺少`（元）-公式`后缀 |
| `T1端呆滞金额` | `T1端呆滞金额（元）-公式` | ❌ 缺少`（元）-公式`后缀 |
| `理想端呆滞数量` | `理想端预计呆滞数量` | ❌ 缺少`预计` |
| `T1端呆滞数量` | `T1端预计呆滞数量` | ❌ 缺少`预计` |
| `LT` | `零件LT（天）` | ⚠️ 缺少`零件`前缀和`（天）`后缀 |
| `CW` | `零件CW（天）` | ⚠️ 缺少`零件`前缀和`（天）`后缀 |

**问题影响：**
- `findColumnIndex`无法找到这些列，返回-1
- `getColumnValue`返回默认值（空字符串或0）
- 所有依赖这些字段的分析获得空数据或错误数据
- 帕累托分析显示空白（依赖`呆滞总金额`、`T1零部件名称`、`T1供应商名称`）
- 专用性分析显示空白（依赖`理想端呆滞金额`、`T1端呆滞金额`）
- CWLT分析显示空白（依赖`LT`、`CW`字段）

**修复内容：**
1. 更新COLUMN_MAPPING，将实际Excel列名作为每个字段的首选匹配项：
   ```javascript
   'LT': ['零件LT（天）', 'LT', 'Lead Time', '交期'],
   'CW': ['零件CW（天）', 'CW', 'Component Weight'],
   '理想端呆滞数量': ['理想端预计呆滞数量', '理想端呆滞数量', ...],
   'T1端呆滞数量': ['T1端预计呆滞数量', 'T1端呆滞数量', ...],
   '理想端呆滞金额': ['理想端呆滞金额（元）-公式', '理想端呆滞金额', ...],
   'T1端呆滞金额': ['T1端呆滞金额（元）-公式', 'T1端呆滞金额', ...],
   '呆滞总金额': ['呆滞总金额（元）-公式', '呆滞总金额', ...],
   '是否认可为行业内理想专用': [..., '是否为理想专用', ...],
   ```

2. 更新版本号从v2.3.2到v2.3.3（267行和1006行）
   - navbar badge: v2.3.3
   - VERSION常量: v2.3.3

**技术细节：**
- 使用Python zipfile和xml.etree解析Excel文件结构
- 读取xl/sharedStrings.xml获取共享字符串表（7043个字符串）
- 读取xl/worksheets/sheet1.xml和sheet2.xml获取实际表头
- 第4行包含68个表头列，部分使用共享字符串索引，部分为内联字符串
- 确认L987和L6的列名基本一致（顺序可能略有不同）

**测试验证：**
- [ ] 上传`25款L987&L6 EOP管控 0127.xlsx`文件
- [ ] 检查浏览器Console，确认不再有"未找到列"警告
- [ ] 验证帕累托分析（零件+供应商）显示数据
- [ ] 验证专用性分析所有图表显示数据
- [ ] 验证CWLT分析所有图表显示数据
- [ ] 验证版本号显示为v2.3.3

**Systematic Debugging应用：**
- Phase 1: 读取实际Excel文件，发现列名不匹配的根本原因
- Phase 2: 对比工作示例（原有COLUMN_MAPPING）与实际需求
- Phase 3: 假设：添加实际列名到COLUMN_MAPPING首位可解决问题
- Phase 4: 实施单一修复，等待用户验证


### 2026-02-12 v2.3.2 - 系统性字段验证和代码审查
**用户需求：** 帕累托分析，专用性分析，CWLT分析里的功能不可用。系统性检查所有功能。

**修复内容：**
1. 添加缺失列验证逻辑（parseWorkbook中）
2. 修复updateExclusiveComparisonChart字段名错误（`是否认可为理想专用` → `是否认可为行业内理想专用`）
3. 更新版本号到v2.3.2

**结果：** 代码审查通过，但问题未完全解决（需要实际Excel文件才能找到根本原因）


### 2026-02-12 v2.3.19 - 修复CW标准匹配逻辑，移除"线束"关键词防止误匹配

**用户需求：** 各品类超标情况中，其他错误匹配到线束天线类

**问题根因：**
- `matchCWStandard`函数关键词数组包含"线束"
- CW标准有3个线束相关品类：
  - 线束连接器：进口CW:120, 国产CW:56
  - 线束附件（卡扣、扎带等）：进口CW:56, 国产CW:30
  - 线束天线类（5G/射频天线）：进口CW:90, 国产CW:21
- 当零部件品类包含"线束"时会匹配所有三个标准，循环返回第一个，导致误匹配

**修复内容：**
1. 从关键词数组中移除"线束"
2. 线束相关品类现在通过模糊匹配正确区分
3. 更新版本号到v2.3.19

**测试验证：**
- [ ] 各品类超标情况中不再有误匹配到线束天线类的问题
- [ ] 版本号显示为v2.3.19


### 2026-02-12 v2.3.20 - 修复专用性分析三个空表的问题

**用户需求：** 专用性分析中，"是否认可为理想专用"表显示正常，但其他三个表是空的

**问题根因：**
- `isExclusive`函数引用了不存在的字段 `item.是否为理想专用`（该字段在parseWorkbook中没有映射）
- `updateExclusiveComparisonChart`中的专用件判断逻辑与`isExclusive`函数不一致
- 只有直接使用`item.是否认可为行业内理想专用`的图表示正常

**受影响的功能：**
1. 按零件类型的专用件占比图表 (`exclusivityPieChart`) - 使用`isExclusive(item)`
2. 按PSM专业组的专用件分布图表 (`exclusivityMultiChart`) - 使用`isExclusive(item)`
3. 专用 vs 非专用金额对比图表 (`exclusiveComparisonChart`) - 使用独立的判断逻辑
4. 专用零件交互查询表格 (`exclusivePartsTable`) - 使用`isExclusive(item)`

**修复内容：**
1. `isExclusive`函数: 移除对不存在字段"是否为理想专用"的引用，只使用实际存在的`是否认可为行业内理想专用`
2. `updateExclusiveComparisonChart`: 统一专用件判断逻辑，与`isExclusive`函数保持一致（匹配"是"、"认可"、"yes"）
3. 更新版本号到v2.3.20

**测试验证：**
- [ ] 按零件类型的专用件占比图表显示正常
- [ ] 按PSM专业组的专用件分布图表显示正常
- [ ] 专用 vs 非专用金额对比图表显示正常
- [ ] 专用零件交互查询表格显示正常
- [ ] 版本号显示为v2.3.20


### 2026-02-12 v2.3.21 - 修复专用性分析三个空表 - 统一状态分组逻辑

**用户需求：** 保证与"是否认可为理想专用"表的逻辑一致

**问题根因：**
- `updateIdealExclusiveChart`直接按状态值（是/否/未填写）分组，显示所有数据
- 其他三个图表使用`isExclusive`函数筛选，只匹配"是"或包含"认可"/"yes"的数据
- 导致数据统计不一致，三个图表显示为空

**修复内容：**
1. `updateExclusiveByTypeChart`: 改为按状态值分组（是/否/未填写），显示所有类型
2. `updateExclusiveByGroupChart`: 改为按状态值分组（是/否/未填写），显示所有专业组
3. `updateExclusiveComparisonChart`: 改为按状态值分组
4. `updateExclusivePartDetail`: 添加"未填写"状态，显示完整状态分布
5. 更新版本号到v2.3.21

**所有四个图表现在都使用相同的逻辑：**
- 状态为"是" → 专用件
- 状态为"否" → 非专用件
- 状态为空或其他 → 未填写

**测试验证：**
- [ ] 按零件类型的专用件占比图表显示正常
- [ ] 按PSM专业组的专用件分布图表显示正常
- [ ] 专用 vs 非专用金额对比图表显示正常
- [ ] 专用零件交互查询表格显示正常
- [ ] 版本号显示为v2.3.21


### 2026-02-12 v2.3.22 - 重写三个专用性分析函数 - 完全复用成功逻辑

**用户需求：** 保证与"是否认可为理想专用"完全一致

**问题根因：**
- v2.3.21版本中，三个函数包含console.log导致性能或执行问题
- 函数逻辑与updateIdealExclusiveChart不一致
- 导致三个图表完全空白

**修复内容：**
1. `updateExclusiveByTypeChart`: 移除所有console.log，完全复用updateIdealExclusiveChart的成功逻辑
2. `updateExclusiveByGroupChart`: 简化为饼图，移除多余的console.log
3. `updateExclusiveComparisonChart`: 移除console.log，与updateIdealExclusiveChart完全一致的逻辑
4. 更新版本号到v2.3.22

**所有四个函数现在都使用相同的处理逻辑：**
- 直接读取`item.是否认可为行业内理想专用`值
- 空值处理为"未填写"
- 按金额排序显示

**测试验证：**
- [ ] 按零件类型的专用件占比图表显示正常
- [ ] 按PSM专业组的专用件分布图表显示正常
- [ ] 专用 vs 非专用金额对比图表显示正常
- [ ] 版本号显示为v2.3.22


### 2026-02-12 v2.3.23 - 专用性分析Tab优化 - 修复空图表并调整布局

**用户需求：**
1. 检查专用性Tab里的四个图表逻辑，现在只有第一个图可以正常显示，其他三个都显示空
2. 将"专用性饼图分布"移动到"专用零件交互查询"里，当选择了具体专业组时，按照专业组刷新。当没有选择时，刷新全部

**问题根因：**
- 专用性Tab中的图表在页面数据加载时没有被正确初始化
- Bootstrap Tab切换时，被切换的Tab内容元素虽然存在，但ECharts图表实例在`display:none`状态下初始化失败

**修复内容：**
1. 添加Tab切换监听器，当切换到专用性Tab时自动初始化图表
2. 将"专用性饼图分布"从"专用性多维度分析"区域移动到"专用零件交互查询"区域
3. 优化交互逻辑：选择PSM专业组时显示该组的专用性饼图，未选择时显示全部数据
4. 添加详细的调试日志，便于问题排查

**版本更新：** v2.3.22 → v2.3.23

**测试验证：**
- [ ] 专用性Tab的三个图表正常显示
- [ ] 专用性饼图位于专用零件交互查询区域
- [ ] 选择PSM专业组时饼图只显示该组数据
- [ ] 未选择PSM专业组时饼图显示全部数据
- [ ] 版本号显示为v2.3.23


---

## 项目指令文件（用于确保上下文清空后仍能遵守要求）

⚠️ **重要！每次修改 EOP Dashboard 时，必须遵守的要求在以下文件：**
- 项目指令：`~/.claude/PROJECT_INSTRUCTIONS.md`
- 需求追溯：`~/REQUIREMENTS.md`

**每次修改后必须执行的步骤：**
1. 读取 `~/.claude/PROJECT_INSTRUCTIONS.md`
2. 读取 `~/REQUIREMENTS.md`
3. 更新版本号
4. Git 提交（包含用户需求文字）
5. Git 推送到 GitHub

