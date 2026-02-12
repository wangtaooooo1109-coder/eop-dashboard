---
name: excel-interactive-dashboard
description: Use when creating interactive HTML dashboards from Excel data with dynamic column mapping, multiple chart types, and data filtering. Especially for business intelligence scenarios requiring template flexibility and self-contained deployment.
---

# Excel Interactive Dashboard

## Overview

Generate self-contained, interactive HTML dashboards from Excel files using SheetJS for parsing and ECharts for visualization. Core strength: dynamic column mapping supports multiple Excel templates without code changes.

## When to Use

Use when you need:
- Interactive data visualization dashboard from Excel files
- Support for multiple Excel templates with different column layouts
- Self-contained HTML (no server, deploy to GitHub Pages/cloud docs)
- Business intelligence charts (KPIs, rankings, trends, distributions)
- Client-side processing (data privacy, no backend needed)

Don't use when:
- Real-time data updates from databases (use API-driven dashboard)
- Large datasets (>10MB Excel files, use backend processing)
- Complex user authentication required
- Need server-side computation

## Architecture Pattern

```
Single HTML File
├── Libraries (CDN)
│   ├── SheetJS (xlsx.js) - Excel parsing
│   ├── ECharts - Charts and visualization
│   └── html2canvas (optional) - Export functionality
├── Data Layer
│   ├── Dynamic column mapping (COLUMN_MAPPING constant)
│   ├── Excel parsing (parseWorkbook function)
│   └── Data transformation
├── UI Layer
│   ├── File upload area (drag & drop)
│   ├── Filters and controls
│   └── Chart containers
└── Chart Layer
    ├── KPI cards
    ├── Distribution charts (pie, bar)
    ├── Ranking charts
    └── Scatter/correlation charts
```

## Core Pattern: Dynamic Column Mapping

**Problem**: Different Excel templates have columns in different positions or with slightly different names.

**Solution**: Map logical field names to possible column header variants.

```javascript
// Define all possible column name variations
const COLUMN_MAPPING = {
    '供应商名称': ['T1供应商名称', '供应商名称', '供应商'],
    '供应商编码': ['T1供应商编码', '供应商编码'],
    '物料编码': ['T1物料编码', '物料编码'],
    '零件名称': ['T1零部件名称', '零部件名称', '零件名称'],
    '呆滞金额': ['呆滞金额', '金额'],
    // ... more mappings
};

// Find column index with exact match first, then partial match
function findColumnIndex(headers, fieldName) {
    const possibleNames = COLUMN_MAPPING[fieldName] || [fieldName];

    // Priority 1: Exact match
    for (let i = 0; i < headers.length; i++) {
        const header = (headers[i] || '').toString().trim();
        for (const name of possibleNames) {
            if (header === name) {
                console.log(`列 "${fieldName}" 精确匹配到索引 ${i}: ${header}`);
                return i;
            }
        }
    }

    // Priority 2: Partial match (longest match wins)
    let bestMatch = { index: -1, matchLength: 0 };
    for (let i = 0; i < headers.length; i++) {
        const header = (headers[i] || '').toString().trim();
        for (const name of possibleNames) {
            if (header.includes(name) && name.length > bestMatch.matchLength) {
                bestMatch = { index: i, matchLength: name.length };
            }
        }
    }

    if (bestMatch.index !== -1) {
        console.log(`列 "${fieldName}" 部分匹配到索引 ${bestMatch.index}`);
        return bestMatch.index;
    }

    console.warn(`未找到列: ${fieldName}, 尝试的列名: ${possibleNames.join(', ')}`);
    return -1;
}

// Safe column value getter
function getColumnValue(row, index, defaultValue = '') {
    if (index === -1 || index >= row.length) {
        return defaultValue;
    }
    return row[index] !== undefined && row[index] !== null ? row[index] : defaultValue;
}
```

**Why this works**:
- Exact match prevents false positives (e.g., "供应商" matching "供应商编码")
- Longest match prioritizes more specific headers
- Console logging helps debug column mapping issues
- Graceful fallback to default values

## Excel Parsing Pattern

```javascript
function parseWorkbook(workbook) {
    const allData = {};

    ['Sheet1', 'Sheet2'].forEach(sheetName => {
        if (!workbook.SheetNames.includes(sheetName)) return;

        const worksheet = workbook.Sheets[sheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1});

        // Assuming headers at row 4 (index 3), data starts at row 5
        const headers = jsonData[3];
        const dataRows = jsonData.slice(4);

        console.log(`${sheetName} 表头:`, headers);

        // Build column index map
        const colMap = {
            供应商名称: findColumnIndex(headers, '供应商名称'),
            物料编码: findColumnIndex(headers, '物料编码'),
            金额: findColumnIndex(headers, '呆滞金额'),
            // ... all fields
        };

        console.log(`${sheetName} 列映射:`, colMap);

        // Parse rows using column map
        allData[sheetName] = dataRows.map(row => {
            if (!row[0]) return null; // Skip empty rows

            return {
                供应商名称: getColumnValue(row, colMap.供应商名称),
                物料编码: getColumnValue(row, colMap.物料编码),
                金额: parseFloat(getColumnValue(row, colMap.金额, 0)) || 0,
                // ... all fields
            };
        }).filter(item => item !== null);

        console.log(`${sheetName} 解析完成，共 ${allData[sheetName].length} 条数据`);
    });

    return allData;
}

// File upload handler
function handleFile(file) {
    const reader = new FileReader();
    reader.onload = function(e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, {type: 'array'});
        const parsedData = parseWorkbook(workbook);
        updateAllCharts(parsedData);
    };
    reader.readAsArrayBuffer(file);
}
```

## Chart Types Quick Reference

| Chart Type | ECharts Type | Use Case | Key Config |
|------------|--------------|----------|------------|
| KPI Cards | Custom HTML | Key metrics overview | Direct DOM manipulation |
| Pie Chart | `type: 'pie'` | Proportions, status distribution | `radius: ['40%', '70%']` for donut |
| Bar Chart | `type: 'bar'` | Rankings, comparisons | Horizontal or vertical |
| Scatter | `type: 'scatter'` | Correlations, distributions | `symbolSize` for value emphasis |
| Line Chart | `type: 'line'` | Trends over time | `smooth: true` for curves |

## Common Patterns

### 1. KPI Card with Trend

```javascript
function updateKPICard(data, config) {
    const total = data.reduce((sum, item) => sum + item.amount, 0);
    const count = data.length;
    const avg = total / count;

    document.getElementById('kpiTotal').textContent =
        '¥' + (total / 10000).toFixed(2) + '万';
    document.getElementById('kpiCount').textContent = count + '条';
    document.getElementById('kpiAvg').textContent =
        '¥' + (avg / 10000).toFixed(2) + '万';
}
```

### 2. Top N Ranking Chart

```javascript
function updateRankingChart(data, topN = 10) {
    const grouped = {};
    data.forEach(item => {
        const key = item.supplier;
        if (!grouped[key]) {
            grouped[key] = { count: 0, amount: 0 };
        }
        grouped[key].count += 1;
        grouped[key].amount += item.amount;
    });

    const sorted = Object.entries(grouped)
        .sort((a, b) => b[1].amount - a[1].amount)
        .slice(0, topN);

    const option = {
        xAxis: { type: 'value' },
        yAxis: {
            type: 'category',
            data: sorted.map(item => item[0])
        },
        series: [{
            type: 'bar',
            data: sorted.map(item => item[1].amount)
        }]
    };

    chart.setOption(option);
}
```

### 3. Multi-Dimensional Filtering

```javascript
function applyFilters(data, filters) {
    return data.filter(item => {
        // Multi-select filter (OR logic)
        if (filters.categories.length > 0 &&
            !filters.categories.includes(item.category)) {
            return false;
        }

        // Range filter
        if (item.amount < filters.minAmount ||
            item.amount > filters.maxAmount) {
            return false;
        }

        // Text search
        if (filters.searchText &&
            !item.name.toLowerCase().includes(filters.searchText.toLowerCase())) {
            return false;
        }

        return true;
    });
}
```

### 4. Pie Chart with Count vs Amount

```javascript
function updatePieChart(data, useCount = true) {
    const grouped = {};
    data.forEach(item => {
        const status = item.status || '未填写';
        if (!grouped[status]) {
            grouped[status] = { count: 0, amount: 0 };
        }
        grouped[status].count += 1;
        grouped[status].amount += item.amount;
    });

    const option = {
        series: [{
            type: 'pie',
            radius: ['40%', '70%'],
            data: Object.entries(grouped).map(([name, value]) => ({
                name: name,
                value: useCount ? value.count : value.amount  // Key choice
            }))
        }]
    };
}
```

**Important**: For status distribution, use `count` not `amount` to show correct proportions.

### 5. Distribution Chart by Interval (Histogram)

```javascript
// Create distribution chart with fixed bin size (e.g., 2 weeks = 0.5 months)
function createDistributionChart(values, binSize, chartId) {
    if (values.length === 0) {
        charts[chartId].clear();
        return;
    }

    // Calculate bins
    const maxValue = Math.max(...values);
    const binCount = Math.ceil(maxValue / binSize);
    const bins = new Array(binCount).fill(0);
    const binLabels = [];

    // Count items in each bin
    values.forEach(value => {
        const binIndex = Math.min(Math.floor(value / binSize), binCount - 1);
        bins[binIndex]++;
    });

    // Generate labels
    for (let i = 0; i < binCount; i++) {
        const start = (i * binSize).toFixed(1);
        const end = ((i + 1) * binSize).toFixed(1);
        binLabels.push(`${start}-${end}`);
    }

    const option = {
        xAxis: {
            type: 'category',
            data: binLabels,
            axisLabel: {
                rotate: 45,
                interval: Math.floor(binCount / 10) // Prevent label crowding
            }
        },
        yAxis: { type: 'value', name: '数量(条)' },
        series: [{
            type: 'bar',
            data: bins,
            label: {
                show: true,
                position: 'top',
                formatter: (params) => params.value > 0 ? params.value : ''
            }
        }]
    };

    charts[chartId].setOption(option);
}

// Usage: LT distribution by 2-week intervals
createDistributionChart(ltValues, 0.5, 'ltDistChart');  // 0.5 month = ~2 weeks
```

**Use case**: Analyze Lead Time (LT) or Component Weight (CW) distribution by fixed time intervals. Helps identify common ranges and outliers.

### 6. Cascading Filters (Parent-Child Dropdowns)

```javascript
// Update child dropdown based on parent selection
function updateCascadingFilter(data, parentValue, childSelectId) {
    const childSelect = document.getElementById(childSelectId);
    const childSet = new Set();

    // Filter data by parent, collect unique child values
    data.filter(item => !parentValue || item.parentField === parentValue)
        .forEach(item => {
            if (item.childField) childSet.add(item.childField);
        });

    // Preserve current selection if still valid
    const currentSelection = childSelect.value;
    childSelect.innerHTML = '<option value="">--全部--</option>';

    Array.from(childSet).sort().forEach(value => {
        const option = document.createElement('option');
        option.value = value;
        option.textContent = value;
        if (value === currentSelection) option.selected = true;
        childSelect.appendChild(option);
    });
}

// Example: PSM group changes → update parts dropdown
document.getElementById('psmSelect').onchange = function() {
    updateCascadingFilter(data, this.value, 'partSelect');
    updateCharts(); // Refresh visualizations
};
```

**Pattern**: When user selects PSM group, parts dropdown shows only relevant parts. Reduces clutter and improves UX.

## Deployment Pattern

**Single HTML file advantages**:
- No build process
- No dependencies to install
- Works offline
- Easy to share (email, cloud storage)
- Deploy to GitHub Pages in one commit

**GitHub Pages deployment**:
```bash
git add dashboard.html
git commit -m "Update dashboard v1.0.0"
git push origin main

# Access at: https://username.github.io/repo/dashboard.html
```

**Embed in Feishu/Lark docs**:
1. Upload to GitHub Pages
2. Copy the GitHub Pages URL
3. Paste URL in Feishu iframe widget

## Version Management

```javascript
// Add version constant
const VERSION = 'v1.0.0';

// Display in UI
<span class="badge" id="versionBadge">{VERSION}</span>

// Use in git commit
git commit -m "v1.0.0 - Initial release

Features:
- Dynamic column mapping
- 10 chart types
- Export functionality

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Pie chart shows wrong proportions | Use `value.count` not `value.amount` for status distribution |
| Column mapping matches wrong column | Use exact match first, then longest partial match |
| Empty values break charts | Use `getColumnValue()` with default values |
| Supplier names show as codes | Check COLUMN_MAPPING includes correct variants |
| Charts not responsive | Use `window.addEventListener('resize', () => chart.resize())` |
| Large Excel files freeze browser | Add loading indicator, use Web Workers for parsing |

## Technology Stack

**Required libraries** (load from CDN):
```html
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
<script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
```

**Browser compatibility**: Modern browsers (Chrome, Firefox, Safari, Edge). No IE11 support needed.

## Real-World Example

See the complete implementation at `/Users/wangtao1/eop_dashboard.html`:
- 5350+ lines single HTML file
- Supports L987 and L6 sheet templates
- 10 KPI cards, multiple chart types, Pareto analysis
- Dynamic column mapping with 23 field mappings
- **Version v2.0.3 features**:
  - Bug fixes for supplier display and review chart proportions (v2.0.1)
  - CW/LT distribution charts by 2-week intervals (v2.0.2)
  - Multi-dimensional cascading filters (PSM group → parts, category, range)
  - Real-time chart updates on filter changes
  - Sortable detail tables (by LT/CW/amount)
  - **CW标准对比分析** (v2.0.3):
    - Embedded 51 CW reference standards (6 major categories)
    - Smart matching algorithm (exact/fuzzy/keyword-based)
    - Exceeds standard analysis with percentage breakdown
    - Category-wise comparison charts
    - Filter by PSM/part/data source

## Testing Checklist

When creating a new dashboard:
- [ ] Test with multiple Excel templates (verify column mapping)
- [ ] Check console for column mapping warnings
- [ ] Verify pie charts use count or amount appropriately
- [ ] Test all filters work correctly
- [ ] Check responsiveness on different screen sizes
- [ ] Test empty/null value handling
- [ ] Verify chart tooltips show correct information
- [ ] Test export functionality (if implemented)
- [ ] Check version number displayed correctly
- [ ] Verify deployment URL works (GitHub Pages/cloud docs)
