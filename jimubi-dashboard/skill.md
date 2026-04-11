---
name: jimubi-dashboard
description: Use when user asks to create/design a dashboard (仪表盘/看板), data kanban, or says "创建仪表盘", "生成仪表盘", "做一个仪表盘", "数据看板", "做一个看板", "创建看板", "数据面板", "统计看板", "运营看板", "create dashboard", "generate dashboard", "design dashboard", "data kanban", "KPI dashboard". Also triggers when user describes dashboard/kanban requirements like "做一个运营数据看板" or mentions grid-layout data display like "统计系统数据". Make sure to use this skill for dashboards (仪表盘/看板) — NOT big screens (大屏), which use completely different positioning, styling, and component configurations.
---

# JeecgBoot 仪表盘 AI 自动生成器

将自然语言的仪表盘需求转换为 drag page 配置，并通过 API 自动创建。

> **本 skill 专门处理仪表盘（default）模式**：网格布局（24列栅格），亮色主题，带卡片头，适用于日常数据看板。
> 大屏请使用 `jimubi-bigscreen` skill。

## 仪表盘特征

- **布局**：24 列栅格，坐标和尺寸单位为**栅格单位**（如 x=0, y=0, w=6, h=17）
- **主题**：默认 `default`，白色背景，深色文字
- **无背景图**：仪表盘通常不设背景图
- **卡片头**：仪表盘支持卡片头，但图表组件的 `card.title` 应留空（标题由 ECharts `option.title` 显示），避免标题重复
- **颜色体系**：白底 `#FFFFFF`、深灰标题 `#464646`、浅灰轴标签 `#909198`、浅灰网格 `#F3F3F3`

## 仪表盘栅格布局规则

| 组件类型 | 推荐 w | 推荐 h | 说明 |
|---------|--------|--------|------|
| JNumber | 6 | 17 | 数字卡片，4 个一行正好 24 列 |
| JLine/JBar/JSmoothLine | 12-14 | 28-35 | 图表，通常半宽或更宽 |
| JPie/JRing/JRose | 10-12 | 28-35 | 饼图/环形图 |
| JHorizontalBar | 12 | 28-35 | 横向柱状图 |
| JTable/JCommonTable | 12 | 30-40 | 数据表格 |
| JScrollTable | 12 | 30-40 | 滚动表格 |
| JScrollRankingBoard | 12 | 30-35 | 排行榜 |
| JGauge | 6-8 | 25-30 | 仪表盘表盘 |
| JLiquid | 6 | 25-30 | 水球图 |

**布局原则：**
- 总宽度 24 列，组件 w 之和不要超过 24
- 第一行通常放 4 个 JNumber（w=6×4=24）
- 第二行放图表组合（如 JLine w=14 + JPie w=10 = 24）
- 第三行放表格/排行等

## 前置条件

用户必须提供：
1. **API 地址**：JeecgBoot 后端地址（如 `https://api3.boot.jeecg.com`）
2. **X-Access-Token**：JWT 登录令牌（从浏览器 F12 获取）

## 交互流程

### Step 0: 解析用户需求

| 信息 | 默认值 | 示例 |
|------|--------|------|
| 页面名称 | 用户指定 | "运营数据看板" |
| 主题 | default | default |
| 组件列表 | 从描述中解析 | 用户总数(数字)、增长趋势(折线)、来源分布(饼图) |

### Step 0.5: 模板匹配（优先使用模板布局）

**生成整个仪表盘时，必须先匹配模板，复用已有布局。** 这是最优先的步骤，能确保生成的仪表盘布局专业、美观。

**模板目录**：`references/templates/default/`（29 个仪表盘模板 JSON）

**匹配流程：**

1. **根据用户需求关键词搜索模板**：将用户描述的行业/场景与模板名称进行语义匹配

| 用户需求关键词 | 推荐模板 |
|---------------|---------|
| 销售/订单/电商/运营 | 产品销售数据、某电商公司销售运营看板、某连锁饮品销售看板 |
| 招聘/HR/人事 | 公司年度招聘看板 |
| 金融/银行/封控 | 金融封控数据展示、示例_乡村振兴普惠金融服务 |
| 仓储/库存/物料 | 库存管理可视化大屏 |
| 医院/医疗/医美 | 示例_医院综合数据统计、医美行业网络关注度 |
| 旅游/景区/客流 | 示例_旅游数据监控 |
| 社区/物业/消防 | 示例_智慧社区、物业消防巡检状态 |
| 生产/制造/车间 | 车间生产管理 |
| 门户/首页/工作台 | 企业门户、流程门户、示例_首页 |
| 消费者/权益/投诉 | 消费者权益保护 |
| 数据分析/统计/报表 | 示例_数据分析、示例_数据表格、示例_统计近十日的登陆次数 |
| 查询/联动/筛选 | 示例_查询_联动、示例_日期范围查询、示例_钻取 |
| 通用/综合/看板 | 示例_智能大数据、示例_全组件、示例_首页 |

2. **找到匹配模板** → 使用「模板复制方式」创建仪表盘（参见下方"备选方式：从模板复制创建仪表盘"章节），保留模板的布局和装饰，仅替换业务数据和标题文字

3. **找不到匹配模板** → 随机选择一个通用模板作为布局基础（推荐选择：`示例_智能大数据`、`示例_首页`、`示例_全组件`），同样保留布局和装饰，替换业务数据

> **重要**：只有在用户明确要求"不使用模板"或"从零创建"时，才跳过模板匹配，直接使用 bi_utils 默认组件函数逐个添加。

### Step 1: 识别组件并选择类型

阅读 `references/bi-component-types.md` 获取完整组件类型清单。

**常用仪表盘组件速查：**

| 用户描述关键词 | 组件 component | 说明 |
|---------------|---------------|------|
| 数字/KPI/指标/总数 | `JNumber` | 数字指标卡（带卡片头） |
| 柱状图 | `JBar` | 基础柱状图 |
| 横向柱状图 | `JHorizontalBar` | 水平柱状图 |
| 折线图/趋势 | `JLine` | 折线图 |
| 曲线图 | `JSmoothLine` | 平滑曲线 |
| 柱线混合 | `JMixLineBar` | 柱状+折线混合 |
| 饼图 | `JPie` | 饼图 |
| 环形图 | `JRing` | 环形图 |
| 玫瑰图 | `JRose` | 南丁格尔玫瑰图 |
| 表盘 | `JGauge` | 仪表盘表盘 |
| 水球图 | `JLiquid` | 水球图 |
| 进度条 | `JProgress` | 进度条 |
| 雷达图 | `JRadar` | 雷达图 |
| 漏斗图 | `JFunnel` | 漏斗图 |
| 地图 | `JAreaMap` | 区域地图 |
| 数据表格 | `JTable` / `JCommonTable` | 数据表格 |
| 滚动表格 | `JScrollTable` | 自动滚动表格 |
| 排行榜 | `JScrollRankingBoard` | 滚动排行榜 |
| 日历 | `JCalendar` | 日历组件 |

### Step 2: 展示设计摘要并确认

**必须展示，等待用户确认后再执行：**

```
## 仪表盘设计摘要

- 页面名称：运营数据看板
- 主题：default

### 组件列表

| 序号 | 组件名称 | 组件类型 | 位置(x,y) | 尺寸(w×h) | 数据源 |
|------|---------|---------|-----------|----------|--------|
| 1 | 总用户数 | JNumber | (0,0) | 6×17 | 静态数据 |
| 2 | 今日活跃 | JNumber | (6,0) | 6×17 | 静态数据 |
| 3 | 用户增长趋势 | JLine | (0,17) | 14×35 | 静态数据 |
| 4 | 用户来源 | JPie | (14,17) | 10×35 | 静态数据 |

确认以上信息正确？(y/n)
```

### 快捷操作：全部预置脚本一览

脚本目录：`C:\Users\25067\.claude\skills\jimubi-dashboard\references\scripts\`

| 脚本 | 功能 | 常用命令 |
|------|------|---------|
| `comp_ops.py` | 组件增删改查 | `list`, `delete`, `edit`, `add`, `move` |
| `page_ops.py` | 页面配置（背景/水印/主题） | `info`, `set-bg`, `set-bgimg`, `set-theme`, `watermark`, `rename` |
| `dataset_ops.py` | 数据集管理 | `list`, `create-sql`, `create-api`, `test`, `delete`, `bind` |
| `template_ops.py` | 模板操作 | `list`, `preview`, `search`, `copy` |
| `linkage_ops.py` | 联动/钻取配置 | `show`, `add-linkage`, `remove-linkage`, `add-drill`, `remove-drill` |
| `map_ops.py` | 地图数据管理 | `list`, `check`, `upload`, `add-map` |
| `style_ops.py` | 批量样式修改 | `show-colors`, `set-title-color`, `set-palette`, `batch-edit` |
| `backup_ops.py` | 备份恢复 | `export`, `import`, `clone`, `diff` |
| `datasource_ops.py` | 数据源管理（含签名） | `list`, `detail`, `create`, `test`, `delete`, `parse-sql` |
| `group_ops.py` | 组合管理 | `list`, `create`, `ungroup`, `rename` |
| `dict_ops.py` | 字典管理 | `list`, `items`, `create`, `add-item`, `delete`, `bind` |

**使用前准备（所有脚本通用）：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-dashboard/references/scripts/脚本名.py" .
cp "C:/Users/25067/.claude/skills/jimubi-dashboard/references/bi_utils.py" .
# 执行完后清理: rm 脚本名.py bi_utils.py
```

> 详细用法参考大屏 skill 的脚本文档，命令格式相同。仪表盘与大屏的区别仅在于坐标单位（栅格 vs 像素）和默认样式。

### Step 3: 调用 API 创建仪表盘

**优先使用共通工具库 `bi_utils.py`**：
- Skills 目录（权威副本）：`references/bi_utils.py`

> 如果工作目录没有 `bi_utils.py`，先从 skills 目录复制过去再使用。

**执行步骤：**
```
1. 确认工作目录有 bi_utils.py（没有则从 skills 复制）
2. Write 工具 → 写入业务脚本 create_xxx_dashboard.py
3. Bash 工具 → python create_xxx_dashboard.py
4. Bash 工具 → rm create_xxx_dashboard.py（清理临时脚本）
```

**仪表盘创建示例：**
```python
import sys
sys.path.insert(0, '.')
from bi_utils import *

init_api('https://api3.boot.jeecg.com', 'your-token')

# 创建仪表盘（style='default'，栅格坐标）
page_id = create_page('运营数据看板', style='default', theme='default')

# 第一行：4 个数字卡片（w=6×4=24，h=17）
add_number(page_id, '总用户数', x=0, y=0, w=6, h=17, value=15890, suffix='人')
add_number(page_id, '今日活跃', x=6, y=0, w=6, h=17, value=3256, suffix='人')
add_number(page_id, '今日收入', x=12, y=0, w=6, h=17, value=89600, prefix='¥')
add_number(page_id, '转化率', x=18, y=0, w=6, h=17, value=23.5, suffix='%')

# 第二行：折线图 + 饼图
add_chart(page_id, 'JLine', '用户增长趋势', x=0, y=17, w=14, h=35,
          categories=['周一','周二','周三','周四','周五','周六','周日'],
          series=[{'name':'新增用户', 'data':[120,200,150,80,70,110,130]}])

add_chart(page_id, 'JPie', '用户来源', x=14, y=17, w=10, h=35,
          pie_data=[
              {'name':'微信','value':40},
              {'name':'APP','value':30},
              {'name':'网页','value':20},
              {'name':'其他','value':10},
          ])

save_page(page_id)
print(f'仪表盘创建成功！ID: {page_id}')
```

**仪表盘样式特点（bi_utils.py 自动应用）：**
- 背景：白色 `#FFFFFF`
- 边框：浅灰 `#E8E8E8`
- 标题颜色：深灰 `#464646`
- 轴标签：`#909198`
- 网格线：`#F3F3F3`
- 卡片头：白色背景 + 深灰粗体标题（`headColor: '#FFFFFF'`）
- 图例：深灰色文字

## 仪表盘标题规则（重要）

### 图表组件：card.title 留空，用 option.title 显示

根据真实模板验证，**图表组件**（JBar/JLine/JPie/JRing 等）在仪表盘模式下 `card.title` 应为空字符串，标题通过 ECharts `option.title.text` 显示。如果两者都设置，标题会重复出现（卡片头一次 + 图表内部一次）。

`bi_utils.py` 的 `add_chart()` 已自动处理：调用 `_make_card(mode, '')` 传入空标题。

**JNumber 等非图表组件**可以使用 `card.title` 显示标题。

### 大屏 vs 仪表盘标题对比

| 特征 | 大屏（bigScreen） | 仪表盘（default） |
|------|-------------------|-------------------|
| 图表标题 | `option.title.text`（ECharts 内部） | `option.title.text`（ECharts 内部） |
| card.title（图表） | 必须为空 `''` | 必须为空 `''`（避免重复） |
| card.title（JNumber等） | 为空 `''` | 可填标题 |
| 页面主标题 | JText 组件（fontSize 40+） | 不需要 |

### JText 正确的 config 格式

如果仪表盘中需要使用 JText（少见），config 结构为：
```python
config = {
    'dataType': 1,
    'chartData': {'value': '显示文本'},  # dict 格式，不是字符串
    'option': {
        'body': {
            'color': '#464646',
            'fontSize': 16,
            'fontWeight': 'normal',
            'letterSpacing': 0,
            'text': '',
            'marginTop': 0,
            'marginLeft': 0,
        },
        'textAlign': 'left',
        'card': {'title': '', ...},
    },
}
```

**手动构建组件（用于高级定制，需直接操作 config）：**

当 `add_chart` 等快捷函数无法满足需求时（如需要多系列 chartData、自定义 customColor），可直接构建组件 config：

```python
import json, time, random
import bi_utils

def _key():
    return f'{int(time.time()*1000)}_{random.randint(100000,999999)}'

# 仪表盘亮色主题通用样式
CARD = {
    'size': 'default',
    'headColor': '#FFFFFF',
    'textStyle': {'color': '#464646', 'fontSize': 16, 'fontWeight': 'bold'},
    'extra': '', 'rightHref': ''
}

# 直接构建折线图组件
line_data = [
    {'name': '1月', 'value': 120, 'type': '新增'},
    {'name': '1月', 'value': 80, 'type': '流失'},
    # ...
]
comp = {
    'component': 'JLine',
    'x': 0, 'y': 17, 'w': 14, 'h': 35,
    'i': _key(),
    'config': json.dumps({
        'dataType': 1,
        'chartData': json.dumps(line_data, ensure_ascii=False),
        'background': '#FFFFFF',
        'borderColor': '#E8E8E8',
        'size': {'width': 700, 'height': 375},
        'option': {
            'customColor': [
                {'color': '#1890FF', 'color1': '#1890FF'},
                {'color': '#52C41A', 'color1': '#52C41A'},
            ],
            'title': {'show': True, 'text': '用户变化趋势',
                      'textStyle': {'color': '#464646'}},
            'tooltip': {'show': True},
            'legend': {'show': True, 'textStyle': {'fontSize': 12}},
            'xAxis': {
                'type': 'category',
                'axisLabel': {'color': '#909198'},
                'axisLine': {'lineStyle': {'color': '#F3F3F3'}},
            },
            'yAxis': {
                'axisLabel': {'color': '#909198'},
                'splitLine': {'lineStyle': {'color': '#F3F3F3'}},
            },
            'grid': {'top': 70, 'left': 60, 'right': 30, 'bottom': 40},
            'card': {**CARD, 'title': '用户变化趋势'},
        }
    }, ensure_ascii=False)
}
bi_utils._page_components[page_id].append(comp)
```

### Step 4: 输出结果

```
## 仪表盘创建成功

- 页面ID：{id}
- 页面名称：{name}
- 模式：仪表盘（default）
- 预览地址：{API_BASE}/drag/page/view/{id}
- 组件数量：{count} 个

请在仪表盘设计器中查看：打开 JeecgBoot 后台 → 仪表盘 → 找到该页面
```

---

## 数据集管理（动态数据源）

仪表盘组件支持三种数据类型（`config.dataType`）：
- `1` — 静态数据（直接写在 `chartData` 中）
- `2` — 动态数据（从数据集获取，支持 SQL / API / JSON / WebSocket）
- `4` — 表单数据（Online表单 / 设计器表单，**无需创建数据集**，见下方「Online表单/设计器表单图表」章节）

### 数据集 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/drag/onlDragDatasetHead/add` | POST | 创建数据集 |
| `/drag/onlDragDatasetHead/edit` | POST | 编辑数据集（需要 `sign` 字段） |
| `/drag/onlDragDatasetHead/delete?id=xxx` | DELETE | 删除数据集 |
| `/drag/onlDragDatasetHead/list` | GET | 分页查询数据集列表 |
| `/drag/onlDragDatasetHead/getAllChartData` | POST | 执行数据集查询（获取图表数据） |
| `/drag/onlDragDatasetHead/queryFieldBySql` | POST | 解析 SQL 返回字段列表（**需签名**） |
| `/drag/onlDragDatasetHead/queryFieldByApi` | POST | 解析 API 返回字段列表 |

### 数据集实体结构（OnlDragDatasetHead）

```python
{
    'name': '数据集名称',
    'code': '数据集编码',          # 可选，唯一标识
    'dataType': 'sql',             # sql / api / json / singleFile / FILES
    'dbSource': '707437208002265088',  # 数据库源 ID（SQL 类型必填！）
    'querySql': 'SELECT ...',      # SQL 语句（SQL 类型）或 API 地址（API 类型）
    'apiMethod': 'get',            # HTTP 方法（API 类型用）
    'izAgent': '0',                # 是否代理：'0'=直连, '1'=服务端代理
    'content': '',                 # 描述
    'parentId': '',                # 父级分类 ID
    'datasetItemList': [           # 字段列表（注意：不是 onlDragDatasetItemList）
        {'fieldName': 'name', 'fieldTxt': '名称', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 0},
        {'fieldName': 'value', 'fieldTxt': '数值', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 1}
    ],
    'datasetParamList': [          # 参数列表（注意：不是 onlDragDatasetParamList）
        {'paramName': 'sex', 'paramTxt': '性别', 'paramValue': '1', 'dictCode': 'sex'}
    ]
}
```

### 创建 SQL 数据集

```python
import sys, json
sys.path.insert(0, r'D:\webstorm_project_2023\vue3-jeecg-drag-design-antd4')
import bi_utils
bi_utils.init_api('http://api3.boot.jeecg.com', 'your-token')

# 创建 SQL 数据集（dbSource 必填！）
result = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '用户男女比例统计',
    'code': 'user_sex_ratio',
    'dataType': 'sql',
    'dbSource': '707437208002265088',   # 本地 MySQL 数据源 ID
    'querySql': "SELECT sex as name, COUNT(*) AS value FROM demo WHERE sex IS NOT NULL AND sex != '' GROUP BY sex",
    'apiMethod': 'GET',
    'datasetItemList': [
        {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 0},
        {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 1}
    ],
    'datasetParamList': []
})
dataset_id = result['result']['id']

# 测试数据集
test = bi_utils._request('POST', '/drag/onlDragDatasetHead/getAllChartData', data={'id': dataset_id})
print(json.dumps(test, ensure_ascii=False))
```

### 创建 API 数据集

```python
result = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '产品销量排行榜',
    'code': 'product_sales',
    'dataType': 'api',
    'dbSource': None,
    'querySql': 'https://api.jeecg.com/mock/31/graphreport/aiproducttest',
    'apiMethod': 'get',
    'izAgent': '0',
    'datasetItemList': [
        {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 0},
        {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 1}
    ],
    'datasetParamList': []
})
dataset_id = result['result']['id']
```

### 组件绑定数据集（dataType=2）

组件的 `config` 中需要设置以下字段来绑定数据集：

```python
config = {
    'dataType': 2,                      # 2=动态数据
    'dataSetId': dataset_id,            # 数据集 ID
    'dataSetName': '数据集名称',
    'dataSetType': 'sql',               # sql / api / json / websocket
    'dataSetApi': 'SELECT ...',         # SQL 语句或 API 地址
    'dataSetMethod': 'get',             # HTTP 方法
    'dataSetIzAgent': '1',              # SQL 类型用 '1'（走后端代理），API 直连用 '0'
    'dataMapping': [                    # 字段映射（关键！）
        {'filed': '维度', 'mapping': 'name'},   # 注意：filed 不是 field（系统拼写）
        {'filed': '数值', 'mapping': 'value'},
        # {'filed': '分组', 'mapping': 'type'},  # 多系列图表需要
    ],
    'fieldOption': [                    # 字段定义列表
        {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String'},
        {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String'},
    ],
    'paramOption': [],                  # 参数定义列表（有动态条件时填写）
    'chartData': '[]',                  # 动态数据时可为空数组
    'option': { ... }                   # ECharts 配置
}
```

### 标准字段映射规则

| 映射标签（filed） | 标准字段（key） | 说明 |
|-------------------|----------------|------|
| `维度` / `名称` | `name` | 图表类目/维度 |
| `数值` | `value` | 图表数值 |
| `分组` | `type` | 多系列区分字段 |
| `文本` | `label` | 文本标签 |

### 组件绑定数据集完整示例（SQL 饼图 - 仪表盘）

```python
pie_comp = {
    'component': 'JPie',
    'componentName': '男女比例',
    'visible': True,
    'i': bi_utils._gen_uuid(),
    'x': 0, 'y': 52, 'w': 12, 'h': 35,  # 栅格坐标
    'orderNum': 100,
    'config': {
        'dataType': 2,
        'w': 900, 'h': 385,              # 像素尺寸（栅格转像素：w*75, h*11）
        'size': {'width': 900, 'height': 385},
        'dataSetId': dataset_id,
        'dataSetName': '用户男女比例统计',
        'dataSetType': 'sql',
        'dataSetApi': "SELECT sex as name, COUNT(*) AS value FROM demo ...",
        'dataSetMethod': 'GET',
        'dataSetIzAgent': '1',
        'dataMapping': [
            {'filed': '维度', 'mapping': 'name'},
            {'filed': '数值', 'mapping': 'value'}
        ],
        'fieldOption': [
            {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String'},
            {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String'},
        ],
        'paramOption': [],
        'chartData': '[]',
        'option': { ... }
    }
}
```

### 组件绑定数据集完整示例（API 柱形图 - 仪表盘）

```python
bar_comp = {
    'component': 'JBar',
    'componentName': '销量排行',
    'visible': True,
    'i': bi_utils._gen_uuid(),
    'x': 12, 'y': 52, 'w': 12, 'h': 35,  # 栅格坐标
    'orderNum': 100,
    'config': {
        'dataType': 2,
        'w': 900, 'h': 385,
        'size': {'width': 900, 'height': 385},
        'dataSetType': 'api',
        'dataSetApi': 'https://api.jeecg.com/mock/31/graphreport/aiproducttest',
        'dataSetMethod': 'get',
        'dataSetIzAgent': '0',          # API 直连不走代理
        'dataMapping': [
            {'filed': '维度', 'mapping': 'name'},
            {'filed': '数值', 'mapping': 'value'}
        ],
        'chartData': '[]',
        'option': { ... }
    }
}
```

---

## API 接口签名机制

部分接口（如 `queryFieldBySql`）带有 `@SignatureValidation` 注解，需要在请求头中携带签名信息。

### 签名算法

```
Secret = 'dd05f1c54d63749eda95f9fa6d49v442a'（来自 window._CONFIG.signatureSecret）

X-TIMESTAMP = 当前毫秒时间戳
X-Sign = MD5(JSON.stringify(sortedUrlParams) + secret + timestamp).toUpperCase()
V-Sign = MD5(JSON.stringify(sortedStringBodyFields + sign) + secret + timestamp).toUpperCase()
```

### Python 签名实现

```python
import hashlib, json, time, urllib.parse

SECRET = 'dd05f1c54d63749eda95f9fa6d49v442a'

def get_sign(url, timestamp):
    """生成 X-Sign（URL 参数签名）"""
    parsed = urllib.parse.urlparse(url)
    params = urllib.parse.parse_qs(parsed.query, keep_blank_values=True)
    sorted_params = {k: params[k][0] for k in sorted(params.keys())}
    params_json = json.dumps(sorted_params, separators=(',', ':'), ensure_ascii=False)
    sign_str = params_json + SECRET + timestamp
    return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()

def get_vsign(data, sign, timestamp):
    """生成 V-Sign（Body 签名）"""
    string_fields = {}
    if isinstance(data, dict):
        for k in sorted(data.keys()):
            v = data[k]
            if isinstance(v, str):
                string_fields[k] = v
    string_fields['sign'] = sign
    body_json = json.dumps(string_fields, separators=(',', ':'), ensure_ascii=False)
    sign_str = body_json + SECRET + timestamp
    return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()

def signed_request(method, url, data=None, token=None, api_base=None):
    """发送带签名的请求"""
    import urllib.request
    full_url = (api_base or '') + url
    timestamp = str(int(time.time() * 1000))
    x_sign = get_sign(full_url, timestamp)
    headers = {
        'Content-Type': 'application/json',
        'X-Access-Token': token,
        'X-TIMESTAMP': timestamp,
        'X-Sign': x_sign,
    }
    if data:
        v_sign = get_vsign(data, x_sign, timestamp)
        headers['V-Sign'] = v_sign
    body = json.dumps(data, ensure_ascii=False).encode('utf-8') if data else None
    req = urllib.request.Request(full_url, data=body, headers=headers, method=method)
    resp = urllib.request.urlopen(req, timeout=30)
    return json.loads(resp.read().decode('utf-8'))
```

### 需要签名的接口

| 接口 | 说明 |
|------|------|
| `/drag/onlDragDatasetHead/queryFieldBySql` | SQL 字段解析 |
| `/drag/onlDragDatasetHead/queryFieldByApi` | API 字段解析 |

> 普通 CRUD 接口（add/edit/delete/list）不需要签名，只需 `X-Access-Token`。

---

## 数据源管理

### 数据源 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/drag/onlDragDataSource/add` | POST | 创建数据源 |
| `/drag/onlDragDataSource/edit` | PUT/POST | 编辑数据源 |
| `/drag/onlDragDataSource/delete?id=xxx` | DELETE | 删除数据源 |
| `/drag/onlDragDataSource/testConnection` | POST | 测试数据库连接 |
| `/drag/onlDragDataSource/getOptions` | GET | 获取数据源下拉列表 |

### 已适配的 18 种数据库类型

| 序号 | 数据库类型 | value | 驱动 | 默认端口 | JDBC URL 模板 |
|------|-----------|-------|------|---------|--------------|
| 1 | MySQL5.5 | MYSQL5.5 | com.mysql.jdbc.Driver | 3306 | jdbc:mysql://host:3306/db?characterEncoding=UTF-8&useUnicode=true&useSSL=false |
| 2 | MySQL5.7+ | MYSQL5.7 | com.mysql.cj.jdbc.Driver | 3306 | jdbc:mysql://host:3306/db?characterEncoding=UTF-8&useUnicode=true&useSSL=false&tinyInt1isBit=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai |
| 3 | TIDB | TIDB | com.mysql.cj.jdbc.Driver | 4000 | jdbc:mysql://host:4000/db?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8&tinyInt1isBit=false |
| 4 | Oracle | ORACLE | oracle.jdbc.OracleDriver | 1521 | jdbc:oracle:thin:@host:1521:ORCL |
| 5 | SQLServer | SQLSERVER | com.microsoft.sqlserver.jdbc.SQLServerDriver | 1433 | jdbc:sqlserver://host:1433;SelectMethod=cursor;DatabaseName=db |
| 6 | MariaDB | MARIADB | org.mariadb.jdbc.Driver | 3306 | jdbc:mariadb://host:3306/db?characterEncoding=UTF-8&useSSL=false |
| 7 | PostgreSQL | POSTGRESQL | org.postgresql.Driver | 5432 | jdbc:postgresql://host:5432/db |
| 8 | 达梦 | dm | dm.jdbc.driver.DmDriver | 5236 | jdbc:dm://host:5236/?db&zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=utf-8 |
| 9 | 人大金仓 | kingbase8 | com.kingbase8.Driver | 54321 | jdbc:kingbase8://host:54321/db |
| 10 | 神通 | oscar | com.oscar.Driver | 2003 | jdbc:oscar://host:2003/db |
| 11 | DB2 | DB2 | com.ibm.db2.jcc.DB2Driver | 50000 | jdbc:db2://host:50000/db |
| 12 | Hsqldb | Hsqldb | org.hsqldb.jdbc.JDBCDriver | - | jdbc:hsqldb:hsql://host/db |
| 13 | Derby | Derby | org.apache.derby.jdbc.ClientDriver | 1527 | jdbc:derby://host:1527/db |
| 14 | MongoDB | mongodb | (无驱动) | 27017 | host:27017/db |
| 15 | Redis | redis | (无驱动) | 6379 | host:6379 |
| 16 | Elasticsearch | es | / | 9200 | host:9200 |
| 17 | Doris | Doris | com.mysql.cj.jdbc.Driver | 9030 | jdbc:mysql://host:9030/db?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8&tinyInt1isBit=false |
| 18 | SQLite | SQLite | org.sqlite.JDBC | - | jdbc:sqlite://path/db.db |

> 来源: `packages/utils/constant.ts` 的 `dbTypeOption`。创建新数据源时，应列出以上选项让用户选择。

### 创建数据源 + 测试连接

```python
# 1. 创建数据源
ds_result = bi_utils._request('POST', '/drag/onlDragDataSource/add', data={
    'name': '业务数据库',
    'dbType': 'MYSQL5.7',
    'dbDriver': 'com.mysql.cj.jdbc.Driver',
    'dbUrl': 'jdbc:mysql://192.168.1.66:3306/jeecgbootsy?characterEncoding=UTF-8&useUnicode=true&useSSL=false&tinyInt1isBit=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai',
    'dbUsername': 'root',
    'dbPassword': 'root',
    'type': 1
})
db_source_id = ds_result['result']

# 2. 测试连接
test_result = bi_utils._request('POST', '/drag/onlDragDataSource/testConnection', data={
    'dbDriver': 'com.mysql.cj.jdbc.Driver',
    'dbUrl': 'jdbc:mysql://192.168.1.66:3306/jeecgbootsy?...',
    'dbUsername': 'root',
    'dbPassword': 'root',
})
print('连接结果:', test_result)
```

### 数据库源 ID 参考

| 环境 | dbSource / dbCode | 说明 |
|------|-------------------|------|
| api3.boot.jeecg.com 主库 | `707437208002265088` | 默认 MySQL 数据库 |

> **注意**：不同环境的 dbSource ID 不同，部署到新环境时需要通过 `/drag/onlDragDataSource/getOptions` 查询可用的数据源列表。

---

## SQL 数据集动态查询条件

SQL 数据集支持 FreeMarker 语法的动态查询条件，实现运行时按参数过滤。

### FreeMarker 语法

```sql
SELECT username, realname, sex
FROM sys_user
WHERE del_flag = 0
<#if isNotEmpty(sex)>
  AND sex = '${sex}'
</#if>
<#if isNotEmpty(realname)>
  AND realname LIKE '%${realname}%'
</#if>
```

### 参数配置

数据集的 `datasetParamList` 定义动态参数：

```python
'datasetParamList': [
    {'paramName': 'sex', 'paramTxt': '性别', 'paramValue': '1', 'dictCode': 'sex'},
    {'paramName': 'realname', 'paramTxt': '姓名', 'paramValue': '', 'dictCode': ''},
]
```

### 组件 config 中的参数配置

绑定带动态条件的数据集时，组件 config 中需要额外设置 `paramOption`：

```python
config = {
    'dataType': 2,
    'dataSetId': dataset_id,
    'dataSetType': 'sql',
    'dataSetApi': 'SELECT ... <#if isNotEmpty(sex)> AND sex = \'${sex}\' </#if>',
    'paramOption': [
        {'paramName': 'sex', 'paramTxt': '性别', 'paramValue': '1', 'dictCode': 'sex'},
    ],
    'fieldOption': [...],
    'dataMapping': [...],
    ...
}
```

### FreeMarker 常用语法

| 语法 | 说明 |
|------|------|
| `<#if isNotEmpty(param)>...</#if>` | 参数非空时拼接条件 |
| `${param}` | 参数值替换 |
| `<#if param == '1'>...</#if>` | 等值判断 |
| `<#list items as item>...</#list>` | 循环 |

### 系统内置变量

| 变量 | 说明 |
|------|------|
| `#{sys.login_user}` | 当前登录用户 |
| `#{sys.login_user_realname}` | 当前用户真名 |
| `#{sys.sys_org_code}` | 当前部门编码 |
| `#{sys.sys_date}` | 当前日期 |

---

## SQL 数据集绑定图表完整端到端流程

### 完整步骤

```
1. 创建数据源（/drag/onlDragDataSource/add）→ 获取 dbSourceId
2. 测试连接（/drag/onlDragDataSource/testConnection）→ 确认可用
3. 解析 SQL 字段（/drag/onlDragDatasetHead/queryFieldBySql）→ 获取字段列表（需签名）
4. 创建数据集（/drag/onlDragDatasetHead/add）→ 获取 datasetId
5. 测试数据集（/drag/onlDragDatasetHead/getAllChartData）→ 验证数据
6. 构建组件 config（dataType=2 + dataMapping + fieldOption + paramOption）
7. 添加组件到页面 → save_page()
```

### queryFieldBySql 调用示例

```python
# 需要签名！
fields_result = signed_request('POST',
    '/drag/onlDragDatasetHead/queryFieldBySql',
    data={
        'dbSource': db_source_id,
        'querySql': 'SELECT username, realname FROM sys_user WHERE del_flag = 0'
    },
    token='your-token',
    api_base='http://192.168.1.66:8080/jeecg-boot'
)
# 返回: {"success":true,"result":[{"fieldName":"username","fieldType":"String"},{"fieldName":"realname","fieldType":"String"}]}
```

### 端到端完整脚本示例（仪表盘折线图 + SQL 数据集 + 动态条件）

```python
import sys, json
sys.path.insert(0, r'D:\webstorm_project_2023\vue3-jeecg-drag-design-antd4')
from bi_utils import *
import bi_utils

API_BASE = 'http://192.168.1.66:8080/jeecg-boot'
TOKEN = 'your-token'
init_api(API_BASE, TOKEN)

# 1. 创建数据源
ds = bi_utils._request('POST', '/drag/onlDragDataSource/add', data={
    'name': '业务库', 'dbType': 'MYSQL5.7',
    'dbDriver': 'com.mysql.cj.jdbc.Driver',
    'dbUrl': 'jdbc:mysql://192.168.1.66:3306/jeecgbootsy?characterEncoding=UTF-8&useUnicode=true&useSSL=false&tinyInt1isBit=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai',
    'dbUsername': 'root', 'dbPassword': 'root', 'type': 1
})
db_id = ds['result']

# 2. 解析字段（需签名）
sql = """SELECT username as name, score as value FROM sys_user
WHERE del_flag = 0
<#if isNotEmpty(sex)> AND sex = '${sex}' </#if>
ORDER BY score DESC LIMIT 10"""

fields = signed_request('POST', '/drag/onlDragDatasetHead/queryFieldBySql',
    data={'dbSource': db_id, 'querySql': sql}, token=TOKEN, api_base=API_BASE)
field_list = [{'fieldName': f['fieldName'], 'fieldTxt': f['fieldName'],
               'fieldType': f.get('fieldType','String'), 'izShow': 'Y', 'orderNum': i}
              for i, f in enumerate(fields.get('result', []))]

# 3. 创建数据集
ds_head = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '用户评分Top10', 'code': 'user_score_top10', 'dataType': 'sql',
    'dbSource': db_id, 'querySql': sql, 'apiMethod': 'GET',
    'datasetItemList': field_list,
    'datasetParamList': [{'paramName': 'sex', 'paramTxt': '性别', 'paramValue': '', 'dictCode': 'sex'}]
})
dataset_id = ds_head['result']['id']

# 4. 创建仪表盘并添加折线图（栅格坐标）
page_id = create_page('用户评分看板', style='default', theme='default')

add_chart(page_id, 'JLine', '用户评分Top10', x=0, y=0, w=24, h=40,
          categories=['加载中...'], series=[{'name': '评分', 'data': [0]}])

# 5. 修改最后一个组件为动态数据
comp = bi_utils._page_components[page_id][-1]
config = comp['config'] if isinstance(comp['config'], dict) else json.loads(comp['config'])
config['dataType'] = 2
config['dataSetId'] = dataset_id
config['dataSetName'] = '用户评分Top10'
config['dataSetType'] = 'sql'
config['dataSetApi'] = sql
config['dataSetMethod'] = 'GET'
config['dataSetIzAgent'] = '1'
config['dataMapping'] = [
    {'filed': '维度', 'mapping': 'name'},
    {'filed': '数值', 'mapping': 'value'}
]
config['fieldOption'] = field_list
config['paramOption'] = [{'paramName': 'sex', 'paramTxt': '性别', 'paramValue': '', 'dictCode': 'sex'}]
config['chartData'] = '[]'
comp['config'] = config

save_page(page_id)
print(f'仪表盘创建成功！ID: {page_id}')
```

### API 数据集绑定图表完整端到端流程

#### 流程概览

```
1. 创建 API 数据集（/drag/onlDragDatasetHead/add, dataType='api'）→ 获取 datasetId
2. 测试数据集（/drag/onlDragDatasetHead/getAllChartData）→ 验证数据格式
3. 构建组件 config（dataType=2 + dataSetType='api' + dataMapping + fieldOption）
4. 添加组件到页面 → save_page()
```

> **API 数据集不需要签名**：创建 API 数据集只需 `X-Access-Token`，不需要 `X-Sign`/`V-Sign`。与 SQL 数据集不同，API 数据集不依赖 `dbSource`，也不需要调用 `queryFieldBySql`。

#### API 数据集与 SQL 数据集的关键差异

| 项目 | API 数据集 | SQL 数据集 |
|------|-----------|-----------|
| `dataType`（数据集） | `'api'` | `'sql'` |
| `dbSource` | `None`（不需要） | 必填（数据库源 ID） |
| `querySql` | 存放 **API URL**（不是 SQL） | 存放 SQL 语句 |
| `izAgent` | `'0'`=前端直连, `'1'`=后端代理 | 不使用 |
| `apiMethod` | `'get'` / `'post'` | `'GET'`（无实际意义） |
| 字段解析 | `queryFieldByApi`（需签名）或手动指定 | `queryFieldBySql`（需签名） |
| 组件 `dataSetIzAgent` | `'0'`（直连）或 `'1'`（代理） | `''`（留空） |
| 创建数据集接口 | 不需要签名 | 不需要签名 |

#### API 数据格式要求

API 返回的数据必须是 JSON 数组，每项包含图表所需字段：

```json
// 标准 name/value 格式（适用于饼图、漏斗图、柱形图等大多数图表）
[
  {"name": "新洲店", "value": 8500},
  {"name": "水围店", "value": 7200},
  {"name": "怡安店", "value": 7000}
]
```

#### 端到端完整脚本示例（API 漏斗图 - 仪表盘）

```python
import sys, json
sys.path.insert(0, r'D:\webstorm_project_2023\vue3-jeecg-drag-design-antd4')
from bi_utils import *
import bi_utils

API_BASE = 'http://192.168.1.66:8080/jeecg-boot'
TOKEN = 'your-token'
DATA_API_URL = 'https://api.jeecg.com/mock/51/beverageSales?type=salesRanking'

init_api(API_BASE, TOKEN)

# 1. 创建 API 数据集
ds_result = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '门店销量排行-API',
    'code': 'store_sales_ranking_api',
    'dataType': 'api',
    'dbSource': None,
    'querySql': DATA_API_URL,       # API 地址存在 querySql 字段
    'apiMethod': 'get',
    'izAgent': '0',
    'datasetItemList': [
        {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 0},
        {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String', 'izShow': 'Y', 'orderNum': 1}
    ],
    'datasetParamList': []
})
dataset_id = ds_result['result']['id']

# 2. 创建仪表盘并添加漏斗图（栅格坐标）
page_id = create_page('门店销量看板', style='default', theme='default')

add_chart(page_id, 'JFunnel', '门店销量漏斗', x=0, y=0, w=12, h=40,
          pie_data=[
              {'name': '新洲店', 'value': 8500},
              {'name': '水围店', 'value': 7200},
              {'name': '怡安店', 'value': 7000},
              {'name': '天健店', 'value': 6400},
              {'name': '欧景店', 'value': 6100},
          ])

# 3. 修改组件为动态数据
comp = bi_utils._page_components[page_id][-1]
config = comp['config'] if isinstance(comp['config'], dict) else json.loads(comp['config'])
config['dataType'] = 2
config['dataSetId'] = dataset_id
config['dataSetName'] = '门店销量排行-API'
config['dataSetType'] = 'api'
config['dataSetApi'] = DATA_API_URL
config['dataSetMethod'] = 'get'
config['dataSetIzAgent'] = '0'
config['dataMapping'] = [
    {'filed': '维度', 'mapping': 'name'},
    {'filed': '数值', 'mapping': 'value'}
]
config['fieldOption'] = [
    {'label': 'name', 'text': 'name', 'type': 'String', 'value': 'name', 'show': 'Y'},
    {'label': 'value', 'text': 'value', 'type': 'String', 'value': 'value', 'show': 'Y'},
]
config['paramOption'] = []
config['chartData'] = '[]'
config['dataFilterNum'] = 5    # 漏斗图只取前5条
comp['config'] = config

save_page(page_id)
```

#### API 数据集常用 mock 地址

| API 地址 | 返回格式 | 适用图表 |
|----------|---------|---------|
| `https://api.jeecg.com/mock/31/graphreport/aiproducttest` | `[{name,value}]` | 柱形/饼图/漏斗 |
| `https://api.jeecg.com/mock/51/beverageSales?type=salesRanking` | `[{name,value}]` 13条门店数据 | 柱形/排行榜/漏斗 |

---

### JFunnel 漏斗图组件配置参考

**数据格式**：`[{name, value}]`（与饼图相同）

**关键配置项**：

| 配置路径 | 说明 | 示例值 |
|----------|------|--------|
| `config.dataFilterNum` | 只取前 N 条数据（建议 3-7） | `5` |
| `option.series[0].sort` | `'descending'`（上大下小）/ `'ascending'`（上小下大） | `'descending'` |
| `option.series[0].gap` | 漏斗层间距（像素） | `2` |
| `option.series[0].left` | 漏斗左偏移 | `'10%'` |
| `option.series[0].width` | 漏斗宽度 | `'80%'` |
| `option.series[0].label.position` | 标签位置：`'inside'` / `'left'` / `'right'` | `'inside'` |
| `option.series[0].label.formatter` | 标签格式 | `'{b}: {c}'` |
| `option.customColor` | 自定义配色数组 `[{color:'#xxx'}]` | — |
| `config.option.reversal` | 是否反转（`true`=上小下大） | `false` |

---

### 文件数据集（单文件 singleFile / 多文件 FILES）

文件数据集通过上传 Excel/CSV/JSON 文件作为数据源，无需外部数据库连接。

#### 文件数据集 vs SQL/API 数据集的关键差异

| 项目 | 单文件 (singleFile) | 多文件 (FILES) | SQL 数据集 | API 数据集 |
|------|-------------------|---------------|-----------|-----------|
| `dataType`（数据集） | `'singleFile'` | `'FILES'` | `'sql'` | `'api'` |
| `dbSource` | **reportId**（页面 ID） | **reportId**（页面 ID） | 数据库源 ID | `None` |
| `querySql` | `select * from {tableName}` | 可跨表 SQL 查询 | SQL 语句 | API URL |
| 文件上传 | 1 个文件（`isSingle=true`） | 多个文件 | 不需要 | 不需要 |
| `content` | `JSON.stringify(fileList)` | 不需要 | 不需要 | 不需要 |
| 字段解析 API | 自动从文件解析 | `queryFileFieldBySql`（非 `queryFieldBySql`） | `queryFieldBySql` | `queryFieldByApi` |
| 支持格式 | `.csv .xls .xlsx .json` | `.csv .xls .xlsx .json` | — | — |

#### 文件上传 API

| 端点 | 方法 | 说明 |
|------|------|------|
| `/jmreport/source/datasource/files/add` | POST (multipart) | 上传文件 |
| `/jmreport/source/datasource/files/get` | GET | 获取文件列表 `?reportId=xxx` |
| `/jmreport/source/datasource/files/preview` | GET | 预览文件数据 |
| `/jmreport/source/datasource/files/del` | DELETE | 删除数据源 |
| `/jmreport/source/datasource/files/del/file` | DELETE | 删除单个文件 |

**上传参数（multipart/form-data）：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `file` | File | 上传的文件 |
| `reportId` | String | 页面 ID（大屏/仪表盘 ID） |
| `isSingle` | Boolean | 单文件数据集设为 `true`，多文件不传 |
| `X-Access-Token` | Header | JWT 令牌 |

**上传返回结构：**
```json
{
  "success": true,
  "message": "filesDataSet/PAGE_ID/default.xls",
  "result": {
    "id": "xxx",
    "dbUrl": "[{\"fileName\":\"default.xls\",\"name\":\"jmf.Sheet1_default_excel\"}]"
  }
}
```

> **表名命名规则**：`jmf.{SheetName}_{fileName}_{ext}`（XLS 取 Sheet 名）或 `jmf.{fileName}_{ext}`（CSV/JSON）。

#### Python 文件上传函数

```python
def upload_file(file_path, report_id, is_single=False):
    url = f'{API_BASE}/jmreport/source/datasource/files/add'
    boundary = f'----WebKitFormBoundary{int(time.time()*1000)}'
    file_name = os.path.basename(file_path)
    with open(file_path, 'rb') as f:
        file_data = f.read()
    body_parts = []
    body_parts.append(f'--{boundary}\r\nContent-Disposition: form-data; name="reportId"\r\n\r\n{report_id}\r\n'.encode())
    if is_single:
        body_parts.append(f'--{boundary}\r\nContent-Disposition: form-data; name="isSingle"\r\n\r\ntrue\r\n'.encode())
    body_parts.append(f'--{boundary}\r\nContent-Disposition: form-data; name="file"; filename="{file_name}"\r\nContent-Type: application/octet-stream\r\n\r\n'.encode())
    body_parts.append(file_data)
    body_parts.append(f'\r\n--{boundary}--\r\n'.encode())
    body = b''.join(body_parts)
    headers = {'Content-Type': f'multipart/form-data; boundary={boundary}', 'X-Access-Token': TOKEN}
    req = urllib.request.Request(url, data=body, headers=headers, method='POST')
    with urllib.request.urlopen(req, timeout=60) as resp:
        return json.loads(resp.read().decode('utf-8'))
```

#### 创建单文件数据集（singleFile）

```python
# 1. 上传文件（isSingle=True）
result = upload_file(FILE_PATH, PAGE_ID, is_single=True)
file_list = json.loads(result['result']['dbUrl'])
table_name = file_list[0]['name']  # 如 jmf.Sheet1_default_excel

# 2. 创建数据集
ds = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '销售数据(单文件)', 'code': 'sales_single',
    'dataType': 'singleFile',        # 关键
    'dbSource': PAGE_ID,             # 关键：页面ID
    'querySql': f'select * from {table_name}',
    'content': json.dumps(file_list, ensure_ascii=False),  # 文件列表
    'datasetItemList': [...], 'datasetParamList': []
})

# 3. 组件 config 绑定
config = {'dataType': 2, 'dataSetType': 'singleFile', 'dataSetApi': f'select ...', ...}
```

#### 创建多文件数据集（FILES）

```python
# 1. 上传多个文件
upload_file(r'E:\data\temp.csv', PAGE_ID)
upload_file(r'E:\data\pop.csv', PAGE_ID)

# 2. 获取文件列表
files = bi_utils._request('GET', '/jmreport/source/datasource/files/get', params={'reportId': PAGE_ID})
file_list = json.loads(files['result']['dbUrl'])

# 3. 创建数据集（可跨文件表 SQL）
ds = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '多文件数据集', 'code': 'multi_files',
    'dataType': 'FILES',             # 关键：大写 FILES
    'dbSource': PAGE_ID,             # 关键：页面ID
    'querySql': f'select name, value from {table_name} order by value desc',
    'datasetItemList': [...], 'datasetParamList': []
})

# 4. 组件 config 绑定
config = {'dataType': 2, 'dataSetType': 'FILES', 'dataSetApi': 'select ...', ...}
```

---

### 数据集踩坑记录

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| **"数据源不存在"** | SQL 数据集未设置 `dbSource` | 必须指定 `dbSource`（如 `707437208002265088`） |
| **字段列表不生效** | 用了 `onlDragDatasetItemList` | 正确字段名是 `datasetItemList` |
| **编辑数据集 510 权限错误** | 缺少 `sign` 字段 | 编辑时需传 `sign: 'E19D6243CB1945AB4F7202A1B00F77D5'` |
| **dataMapping 的 filed 拼写** | 系统中 `filed` 不是 `field` | 必须用 `filed`（少一个 d），这是系统设计 |
| **API 类型跨域** | 前端直连外部 API 遇到 CORS | 设置 `izAgent: '1'` 走后端代理 |
| **SQL 参数替换** | 需要动态参数 | SQL 中用 `#{paramName}`（系统变量）或 `${paramName}`（FreeMarker） |
| **SQL 最大返回 1000 条** | 后端限制 | `getChartData` 方法限制最大 1000 条记录 |
| **queryFieldBySql 签名验证失败** | 该接口带 `@SignatureValidation` | 必须用 `signed_request()` 携带 X-Sign/V-Sign/X-TIMESTAMP |
| **SQL 注入检测拦截** | 查询 information_schema 被拦截 | 后端 `SqlInjectionUtil` 会拦截敏感关键词，直接用已知表名 |
| **API 地址存在 querySql 字段** | API 数据集没有独立的 url 字段 | `querySql` 对 SQL 类型存 SQL，对 API 类型存 API URL |
| **API 数据集不需要 dbSource** | API 类型直接访问外部接口 | `dbSource` 设为 `None`，否则可能报错 |
| **漏斗图数据过多显示拥挤** | 漏斗图层级太多影响视觉效果 | 使用 `dataFilterNum` 限制前 N 条（建议 3-7 条） |
| **API 数据集 izAgent 选择** | mock API 无跨域问题，外部 API 可能有 | 同域/mock 用 `'0'`（直连），跨域用 `'1'`（后端代理） |
| **文件数据集 dbSource 不是数据库ID** | singleFile/FILES 的 `dbSource` 是页面 ID | `dbSource = reportId`（页面 ID），不是数据库连接 ID |
| **单文件数据集需要 content 字段** | 单文件的 `content` 存文件列表 JSON | `content = JSON.stringify([{fileName, name}])`，多文件不需要 |
| **多文件字段解析用 queryFileFieldBySql** | 多文件的字段解析 API 不同于 SQL 数据集 | 用 `queryFileFieldBySql`（非 `queryFieldBySql`），参数 `dbCode = reportId` |
| **XLS 文件表名含 Sheet 名** | 系统从 Excel 的 Sheet 名生成表名 | 表名格式 `jmf.{SheetName}_{fileName}_{ext}`，如 `jmf.Sheet1_default_excel` |
| **CSV 编码问题** | UTF-8 BOM 头导致字段名乱码 | 上传前确保文件为纯 UTF-8（无 BOM），或系统会自动处理 |
| **文件上传 isSingle 参数** | 单文件和多文件的区别标志 | 单文件上传传 `isSingle=true`，多文件不传此参数 |

---

## Online表单 / 设计器表单 生成图表（dataType=4）

> 仪表盘与大屏共用此机制，完整技术细节见 bigscreen skill 的
> `references/online-design-form-chart-guide.md`

### 概述

`config.dataType = 4` 表示组件数据来源于 Online表单（cgform）或设计器表单（desform），
而非普通数据集。与 dataType=2 的区别：
- **不需要创建数据集**，直接绑定表单
- 通过字段映射（nameFields/valueFields）配置图表维度和指标
- 后端 API：`POST /drag/onlDragDatasetHead/getTotalDataByCompId`

### 何时选 Online 表单 vs 设计器表单

| 场景 | 类型 | formType |
|------|------|---------|
| 通过 cgform 创建的业务表 | Online表单 | `'online'` |
| 通过拖拽设计器（desform）建的表单 | 设计器表单 | `'design'` |

### 查询表单信息

```python
# Online 表单
res = bi_utils._request('GET', '/online/cgform/head/list',
    params={'keyword': '关键词', 'pageNo': 1, 'pageSize': 10, 'copyType': 0})
form = res['result']['records'][0]
form_id = form['id']          # formId
table_name = form['tableName'] # 实际表名
form_name = form['tableTxt']

# 设计器表单
res = bi_utils._request('GET', '/desform/api/list/options',
    params={'keywords': '关键词', 'pageNo': 1, 'pageSize': 10})
form = res['result'][0]
form_id = table_name = form['value']  # code（同一个值）
form_name = form['label']
```

### 查询字段

```python
# Online 表单字段（过滤非主键且已持久化）
res = bi_utils._request('GET', '/online/cgform/field/listByHeadId',
    params={'headId': form_id})
fields = [f for f in res['result'] if f.get('dbIsKey') == 0 and f.get('dbIsPersist') != 0]
# 关键属性：dbFieldName, dbFieldTxt, dbType('String'/'int'/'double'/'Date'), fieldShowType, dictField

# 设计器表单字段
res = bi_utils._request('GET', f'/desform/api/fields/{form_id}',
    params={'subTable': 'true'})
fields = res['result']['fields']
# 关键属性：model(code), name(txt), type(widget), options
```

### 字段角色分配

| 角色 | 字段 key | 放什么 |
|------|---------|--------|
| 维度（X轴/分类） | `nameFields` | 分类字段（部门/性别/状态），String 类型 |
| 指标（Y轴/数值） | `valueFields` | 数值字段，或 `record_count`（记录数量） |
| 分组 | `typeFields` | 仅分组图表使用，放分类字段 |

**`record_count` 字段**（统计记录数，无需表内存在此字段）：
```python
record_count_field = {
    'fieldName': 'record_count', 'fieldTxt': '记录数量',
    'fieldType': 'count', 'widgetType': 'text',
}
```

### 构建 config（通用模板）

```python
def make_form_chart_config(form_type, form_id, form_name, table_name,
                            name_fields, value_fields, chart_type='JBar',
                            type_fields=None, time_range='all'):
    def field_obj(f, ftype):
        if ftype == 'online':
            return {'fieldName': f['dbFieldName'], 'fieldTxt': f['dbFieldTxt'],
                    'fieldType': f['dbType'], 'widgetType': f.get('fieldShowType','text'),
                    'dictCode': f.get('dictField','')}
        else:  # design
            t = 'number' if f['type'] in ['money','integer','number','rate','slider','formula','summary'] else 'string'
            if f['type'] == 'date': t = 'date'
            return {'fieldName': f['model'], 'fieldTxt': f['name'],
                    'fieldType': t, 'widgetType': f['type'], 'options': f.get('options',{})}

    return {
        'dataType': 4,
        'formType': form_type,                          # 'online' | 'design'
        'formId': form_id,
        'formName': form_name,
        'tableName': table_name,
        'type': 'design' if form_type == 'design' else None,
        'appId': '', 'appType': 'current',
        'nameFields': [field_obj(f, form_type) for f in name_fields],
        'valueFields': [field_obj(f, form_type) if not f.get('fieldType') == 'count'
                        else f for f in value_fields],
        'typeFields': [field_obj(f, form_type) for f in (type_fields or [])],
        'assistYFields': [], 'assistTypeFields': [], 'calcFields': [],
        'sorts': {'name': ''},
        'filter': {'queryField': 'create_time', 'queryRange': time_range},
        'chart': {'category': 'Bar', 'subclass': chart_type, 'isGroup': False},
        'compStyleConfig': {}, 'analysis': {}, 'filterField': [],
        'option': {}, 'size': {'height': 385},  # 仪表盘用像素尺寸
        'turnConfig': {'url': ''}, 'jsConfig': '', 'drillData': [],
        'authFieldShowResult': [], 'timeOut': 0,
    }
```

### 仪表盘组件 newItem 示例（dataType=4）

```python
import uuid
new_item = {
    'component': 'JPie',          # 最终图表类型（与 online/design compType 无关）
    'componentName': '部门人员分布',
    'config': make_form_chart_config(
        form_type='online',
        form_id=form_id,
        form_name=form_name,
        table_name=table_name,
        name_fields=[dept_field],         # X轴：部门字段
        value_fields=[record_count_field], # Y轴：记录数量
        chart_type='JPie',
    ),
    'i': uuid.uuid4().hex,
    'x': 12, 'y': 17,       # 栅格坐标
    'w': 12, 'h': 30,        # 栅格尺寸
    'orderNum': 50,
}
```

**注意**：仪表盘 `config.size.height` 用像素（如 385），不用栅格单位。

### 图表类型推荐

| 字段组合 | 推荐图表 | component |
|---------|---------|-----------|
| 1个分类 nameField + record_count | 饼图 | `JPie` |
| 1个分类 nameField + 数值 valueField | 柱形图 | `JBar` |
| 仅1个 valueField（总量） | 数字指标 | `JNumber` |
| 时间 nameField + 数值 valueField | 折线图 | `JLine` |
| 多 valueField | 多系列柱形图 | `JMultipleBar` |

### queryRange 时间范围值

`all`（全部）/ `today` / `yesterday` / `last7days` / `last30days` /
`week`（本周）/ `preWeek`（上周）/ `month`（本月）/ `preMonth`（上月）/ `year`（本年）

---

## 编辑已有仪表盘

```python
from bi_utils import *
init_api('https://api3.boot.jeecg.com', 'your-token')

page = query_page(page_id)
print(page['name'], page['updateCount'])

add_chart(page_id, 'JBar', '新增图表', x=0, y=52, w=12, h=35,
          categories=['A','B','C'], series=[{'name':'值','data':[10,20,30]}])
save_page(page_id)
```

---

## 删除仪表盘

```python
from bi_utils import *
init_api('https://api3.boot.jeecg.com', 'your-token')

delete_page(page_id)                # 软删除
delete_page(page_id, physical=True) # 硬删除
recover_page(page_id)               # 恢复
```

---

## 修改组件样式

阅读 `references/bi-comp-option-config.md` 获取每种组件的完整配置项路径。

**仪表盘样式修改关键规则：**
- 颜色使用色值（`#000000`），不用英文单词
- customColor 格式：`[{color1:'#xxx',color:'#xxx'}]`
- 卡片头样式：`option.card.textStyle.color`、`option.card.headColor`
- 背景色：`config.background`（仪表盘默认 `#FFFFFF`）
- 边框色：`config.borderColor`（仪表盘默认 `#E8E8E8`）

```python
import sys, json
sys.path.insert(0, '.')
from bi_utils import *
import bi_utils

init_api('https://api3.boot.jeecg.com', 'your-token')

page_id = 'xxx'
page = query_page(page_id)
tmpl = page.get('template', [])
if isinstance(tmpl, str):
    tmpl = json.loads(tmpl)

for comp in tmpl:
    config_str = comp.get('config', '{}')
    config = json.loads(config_str) if isinstance(config_str, str) else config_str
    if comp.get('component') == 'JBar':
        option = config.get('option', {})
        option['series'][0]['itemStyle'] = {'color': '#1890FF'}
        config['option'] = option
        comp['config'] = json.dumps(config, ensure_ascii=False)

bi_utils._page_components[page_id] = tmpl
save_page(page_id)
```

---

## 可用的快捷函数

**API 初始化：**
- `init_api(api_base, token)` — 初始化 API 地址和 Token

**页面管理：**
- `create_page(name, style='default', theme='default')` — 创建仪表盘
- `query_page(page_id)` — 查询页面详情
- `list_pages(style='default')` — 列表查询
- `save_page(page_id)` — 保存设计
- `delete_page(page_id, physical)` — 删除
- `recover_page(page_id)` — 恢复
- `copy_page(page_id)` — 复制

**添加组件（栅格坐标）：**
- `add_number(page_id, title, x, y, w, h, value, prefix, suffix)` — 数字指标
- `add_chart(page_id, chart_type, title, x, y, w, h, categories, series, pie_data)` — 图表
- `add_table(page_id, title, x, y, w, h, columns, data)` — 数据表格
- `add_scroll_table(page_id, title, x, y, w, h, columns, data)` — 滚动表格
- `add_ranking(page_id, title, x, y, w, h, data)` — 排行榜
- `add_text(page_id, title, x, y, w, h, content, font_size, color)` — 文本
- `add_image(page_id, title, x, y, w, h, src)` — 图片
- `add_gauge(page_id, title, x, y, w, h, value, max_val, unit, color)` — 仪表盘表盘
- `add_liquid(page_id, title, x, y, w, h, value, color)` — 水球图
- `add_component(page_id, component, title, x, y, w, h, config)` — 通用组件

---

## API 踩坑记录

| 问题 | 说明 |
|------|------|
| `POST /drag/page/add` 返回值 | 返回完整实体含 ID |
| `POST /drag/page/edit` 乐观锁 | 必须传 `updateCount` |
| Windows curl 中文问题 | 必须用 Python urllib/requests |
| 坐标单位 | 仪表盘用**栅格**坐标（24列） |
| 组件 config 分离 | config 存在 onl_drag_page_comp 表 |
| **size 字段必须是像素** | `config.size.width/height` 是像素值，不是栅格单位。仪表盘栅格转像素：`width = w * 75`, `height = h * 11`。如果直接传栅格值，图表会缩成一小团 |
| **chartData 必须是 JSON 字符串** | `config.chartData` 的值必须是 `json.dumps(...)` 后的字符串，不能是原生 list/dict，否则图表不渲染或显示异常 |
| **图表标题去重** | 图表组件（JBar/JLine/JPie 等）的 `option.card.title` 应为空字符串，标题仅通过 `option.title.text` 显示；否则卡片头和图表内部会重复显示标题 |
| **多系列 chartData 格式** | 多系列图表的 chartData 需要 `type` 字段区分系列：`[{"name":"1月","value":10,"type":"系列A"}, {"name":"1月","value":20,"type":"系列B"}]` |
| **HTTPS 连接问题** | api3.boot.jeecg.com 使用 HTTP 协议（非 HTTPS），`init_api` 时用 `http://` 前缀 |
| **queryFieldBySql 签名失败** | 该接口带 `@SignatureValidation`，需要 X-Sign/V-Sign/X-TIMESTAMP 请求头 |
| **dataMapping 的 filed 拼写** | 系统中字段名是 `filed`（不是 `field`），少一个 d，这是系统设计 |
| **SQL 数据集无 dbSource** | SQL 类型数据集必须指定 `dbSource`（数据库源 ID），否则报"数据源不存在" |
| **datasetItemList 字段名** | 创建数据集时字段列表用 `datasetItemList`，不是 `onlDragDatasetItemList` |
| **SQL 注入检测拦截** | 后端 `SqlInjectionUtil` 会拦截 information_schema 等敏感关键词 |
| **SQL 最大返回 1000 条** | `getChartData` 方法限制最大 1000 条记录 |
| **字典翻译用 jimu_dict 不是 sys_dict** | 大屏/仪表盘的字典存储在 `jimu_dict`/`jimu_dict_item` 表中，API 为 `/jmreport/dict/*` 和 `/jmreport/dictItem/*`。**不要**用系统字典 `/sys/dict/*`，那是 JeecgBoot 业务系统的字典，大屏/仪表盘数据集无法识别 |

## 字典管理（jimu_dict / jimu_dict_item）

大屏/仪表盘的字典翻译使用 `jimu_dict` 和 `jimu_dict_item` 表，**不是**系统字典表 `sys_dict` / `sys_dict_item`。这是两套独立的字典体系，不要混用。

### 关键区别

| 项目 | 大屏/仪表盘字典（正确） | 系统字典（错误） |
|------|----------------------|-----------------|
| 数据库表 | `jimu_dict` / `jimu_dict_item` | `sys_dict` / `sys_dict_item` |
| API 前缀 | `/jmreport/dict/*`、`/jmreport/dictItem/*` | `/sys/dict/*`、`/sys/dictItem/*` |
| 使用场景 | 大屏/仪表盘组件的数据集字典翻译 | JeecgBoot 系统业务字典 |
| 数据集绑定 | `datasetItemList[].dictCode` 引用 `jimu_dict.dictCode` | 不适用 |

### API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/jmreport/dict/list` | GET | 分页查询字典列表（支持 `dictCode` 过滤） |
| `/jmreport/dict/add` | POST | 新增字典 |
| `/jmreport/dict/edit` | POST | 编辑字典 |
| `/jmreport/dictItem/list` | GET | 查询字典项列表（`dictId` 过滤） |
| `/jmreport/dictItem/add` | POST | 新增字典项 |
| `/jmreport/dictItem/edit` | POST | 编辑字典项 |

### jimu_dict 实体结构

```python
{
    'dictName': '性别新',        # 字典名称
    'dictCode': 'sexnew',       # 字典编码（唯一标识，数据集中引用此值）
    'description': '性别字典',   # 描述
}
```

### jimu_dict_item 实体结构

```python
{
    'dictId': '字典主表ID',      # 关联 jimu_dict.id
    'itemText': '男性',          # 显示文本
    'itemValue': '1',           # 实际值（与数据库字段值对应）
    'sortOrder': 1,             # 排序号
    'status': 1,                # 状态 1=启用
}
```

### 创建字典完整示例

```python
# 1. 检查字典是否已存在
check = req('GET', '/jmreport/dict/list', params={'dictCode': 'sexnew', 'pageNo': 1, 'pageSize': 10})
records = check.get('result', {}).get('records', [])

# 2. 创建字典主表
if not records:
    add_result = req('POST', '/jmreport/dict/add', data={
        'dictName': '性别新',
        'dictCode': 'sexnew',
        'description': '性别字典（男性、女性、其他）',
    })
    # 注意：返回 result 为 null，需要重新查询获取 ID
    re_query = req('GET', '/jmreport/dict/list', params={'dictCode': 'sexnew', 'pageNo': 1, 'pageSize': 10})
    dict_id = re_query['result']['records'][0]['id']

# 3. 创建字典项
for item in [
    {'itemText': '男性', 'itemValue': '1', 'sortOrder': 1},
    {'itemText': '女性', 'itemValue': '2', 'sortOrder': 2},
    {'itemText': '其他', 'itemValue': '0', 'sortOrder': 3},
]:
    req('POST', '/jmreport/dictItem/add', data={
        'dictId': dict_id,
        'itemText': item['itemText'],
        'itemValue': item['itemValue'],
        'sortOrder': item['sortOrder'],
        'status': 1,
    })
```

### 数据集中使用字典翻译

在创建数据集时，`datasetItemList` 中的字段可以通过 `dictCode` 绑定字典：

```python
'datasetItemList': [
    {'fieldName': 'name', 'fieldTxt': 'name', 'fieldType': 'String',
     'izShow': 'Y', 'orderNum': 0, 'dictCode': 'sexnew'},  # 绑定字典翻译
    {'fieldName': 'value', 'fieldTxt': 'value', 'fieldType': 'String',
     'izShow': 'Y', 'orderNum': 1}
]
```

绑定后，数据集查询返回的 `dictOptions` 会包含翻译映射：
```json
{
  "dictOptions": {
    "name": [
      {"value": "1", "text": "男性"},
      {"value": "2", "text": "女性"},
      {"value": "0", "text": "其他"}
    ]
  },
  "data": [{"name": "1", "value": 6}, {"name": "2", "value": 1}]
}
```

组件渲染时会自动将 `name=1` 显示为"男性"，`name=2` 显示为"女性"。

### 踩坑记录

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| **字典翻译不生效** | 字典创建到了 `sys_dict` 表 | 必须使用 `/jmreport/dict/add` 创建到 `jimu_dict` 表 |
| **创建字典后返回 result=null** | `/jmreport/dict/add` 不返回 ID | 创建后重新查询 `/jmreport/dict/list?dictCode=xxx` 获取 ID |
| **字典项 itemValue 类型** | 必须是字符串 | 即使数据库中 sex 是数字，`itemValue` 也要传字符串 `'1'` 而不是数字 `1` |
| **dictCode 不是 dictId** | 数据集绑定用的是编码不是 ID | `datasetItemList[].dictCode` 填 `'sexnew'`（编码），不是填 jimu_dict 的 ID |

---

## 从模板复制创建仪表盘

`references/templates/default/` 下有 41 个仪表盘模板 JSON 可供选择。当用户需求与某个模板相似时，可复制模板并只替换业务数据，效果优于从零拼装。

### 模板复制原则（重要）

**只替换数据，不改装饰：**
- **不要替换的组件样式**：JDragBorder（边框装饰）、JDragDecoration（装饰条）、JImg（图片）、JSelectRadio（选项卡）等装饰/图片/边框类组件的**样式、类型、位置、尺寸**不要修改，这些决定了模板的视觉风格
- **边框/装饰中的标题文字可以替换**：JDragBorder 等装饰组件如果 config 中包含标题文本（如 `option.card.title` 或 `componentName`），可以将标题改为业务相关的文字，但边框样式本身（type、color、borderWidth 等）保持不变
- **可以替换的组件**：JBar/JLine/JPie/JRing 等图表组件的 `chartData`、JScrollBoard/JScrollTable 的数据、JColorBlock 的指标数据、JText 的文本内容
- **替换范围**：只修改 `config.chartData`（业务数据）和相关标题文字，不修改 `config.option` 中的样式配置（颜色、字号、边距等）
- **数据量控制**：替换后的数据条数应与原模板保持一致或接近，防止图表内容溢出或显示不适配

### 模板复制完整流程

```python
import sys, json
sys.path.insert(0, r'当前工作目录')
from bi_utils import *
import bi_utils

init_api('http://192.168.1.66:8080/jeecg-boot', 'your-token')

# 1. 读取模板 JSON
tpl_path = r'C:/Users/25067/.claude/skills/jimubi-dashboard/references/templates/default/模板名.json'
with open(tpl_path, 'r', encoding='utf-8') as f:
    tpl_data = json.load(f)
template_components = tpl_data.get('template', [])

# 2. 建立旧 ID → 新 ID 映射
id_mapping = {}
for comp in template_components:
    old_i = comp['i']
    id_mapping[old_i] = bi_utils._gen_uuid()

# 3. 更新组件 ID 和清理
for comp in template_components:
    comp['i'] = id_mapping[comp['i']]
    comp.pop('pageCompId', None)
    config = comp.get('config', {})
    if isinstance(config, str):
        try: config = json.loads(config)
        except: config = {}
    comp['config'] = config

# 4. 只替换图表组件的 chartData（装饰组件不动）
for comp in template_components:
    component = comp.get('component', '')
    cfg = comp.get('config', {})
    if component in ('JBar', 'JLine', 'JPie', 'JRing', ...):
        cfg['chartData'] = json.dumps(新数据, ensure_ascii=False)
    # JDragBorder/JImg 等装饰组件：不修改

# 5. 创建页面并保存
page_id = create_page('仪表盘名称', style='default', theme='light')
bi_utils._page_components[page_id] = template_components
save_page(page_id)
```

### 组件边界检查

仪表盘使用栅格布局（24列），组件不应超出栅格范围。复制模板后应检查：
- **宽度**：每个组件的 `x + w <= 24`（栅格列数上限）
- **高度**：虽然仪表盘支持纵向滚动，但组件过于靠下会影响首屏展示效果
- **size 字段同步**：修改 w/h 后，`config.size.width`（= w × 75）和 `config.size.height`（= h × 11）也必须同步更新，否则图表渲染尺寸与容器不匹配

### 模板复制踩坑记录

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| **页签切换不工作** | JTabToggle 的 `compVals` 引用了旧组件 ID | 必须建立 ID 映射，更新 `config.option.items[].compVals` |
| **JGroup 内部组件异常** | JGroup 的 `props.elements` 内也有 ID 交叉引用 | 序列化后批量替换旧 ID |
| **替换数据后图表显示拥挤或溢出** | 替换的数据条数、文字长度与原模板差异过大 | 替换数据时保持数据量与原模板一致，避免数据过多导致图表内容溢出 |
| **装饰组件被意外修改导致风格丢失** | 替换时误改了 JDragBorder/JImg 等装饰组件的样式 | 只替换图表组件的 chartData 和标题文字，装饰/图片/边框类组件的样式完全保留 |

---

## 错误处理

| 错误 | 解决方案 |
|------|---------|
| Token 过期（401） | 重新获取 X-Access-Token |
| `updateCount` 不匹配 | 重新查询页面获取最新值 |
| 组件不显示 | 检查 dataType、chartData（必须是 JSON 字符串）、option 是否完整 |
| 图表缩成小点 | 检查 `config.size` 是否用了像素值（不是栅格单位），仪表盘需 `w*75` / `h*11` |
| 标题重复显示 | 图表组件的 `option.card.title` 设为空，仅用 `option.title.text` |
| 布局错乱 | 确认使用栅格坐标（不是像素），w 总和 ≤ 24 |
| 中文乱码 | 使用 Python（不要用 curl） |

## 参考文档

- `references/bi-component-types.md` — 完整组件类型清单
- `references/bi-comp-option-config.md` — 组件样式配置路径
- `references/bi_utils.py` — 工具库源码
- `references/templates/default/` — 41 个仪表盘模板 JSON 参考
