---
name: jimubi-bigscreen
description: Use when user asks to create/design a big screen (大屏), full-screen data visualization, or says "创建大屏", "生成大屏", "新建大屏", "设计大屏", "做一个大屏", "BI大屏", "数据大屏", "可视化大屏", "监控大屏", "create big screen", "design big screen", "BI visualization big screen". Also triggers when user describes big screen requirements like "做一个销售数据大屏" or mentions full-screen display like "展厅展示", "监控室大屏". Make sure to use this skill for big screens (大屏) — NOT dashboards (仪表盘/看板), which use a completely different layout and styling system.
---

# JeecgBoot 大屏 AI 自动生成器

将自然语言的大屏需求转换为 drag page 配置，并通过 API 自动创建。

> **本 skill 专门处理大屏（bigScreen）模式**：全屏展示，绝对定位（像素坐标），深色主题，适用于监控室/展厅/展示墙。
> 仪表盘（看板）请使用 `jimubi-dashboard` skill。

## 按需加载指南

本 skill 采用分层加载：核心规则始终在上下文中，专题文档按需读取。

| 场景 | 读取文件 |
|------|---------|
| 需要示例/演示数据（用户未提供数据源）| `references/api-dataset-examples.md`（92条公开 mock API，按行业分类：饮料零售/快消品/电商运营/库存管理/物业消防/AI产品/企业OA，直接用 `dataset_ops.py create-api` 创建，无需鉴权） |
| 创建/绑定/修改数据集（SQL/API/文件） | `references/dataset-guide.md`（**仅自定义脚本时需要**；使用 `dataset_ops.py` / `comp_ops.py --dataset-name` / `comp_ops.py --create-sql` / `comp_ops.py --sql-params` 预置脚本时**无需读取**，脚本已封装全部逻辑，含 FreeMarker 查询参数支持） |
| **多文件数据集（FILES）+ 图表** | 直接用 `files_ops.py create-bind`（**无需 Write 脚本**，无需读文档）。封装：创建空数据集→上传多文件→自动探查列名→推断 JOIN SQL→绑定图表。详见下方「快捷操作：files_ops.py」章节。**⚠️ 执行前必须先确认 Excel 列名**：JOIN 模式需要 `--group-by`、`--agg`、`--join-on` 三个参数，列名未知时禁止盲目执行，必须先询问用户 |
| 创建 WebSocket 数据集 | `references/dataset-guide.md`「创建 WebSocket 数据集」章节（**预置脚本不支持 WebSocket 类型**，必须自定义脚本：`dataType='websocket'`，`querySql` 存 ws:// 地址，无需 dbSource。绑定组件后通过 `/drag/websocket/sendData?chartId=xxx` 推送数据） |
| SQL 数据集使用存储过程（CALL） | 直接用 `proc_ops.py bindcomp`，**无需读任何文档**（一键完成：pymysql 创建存储过程 + 创建数据集 + 绑定组件；自动从数据源 API 获取数据库连接信息） |
| **生成全组件大屏**（含全部 99 个业务组件） | 直接用 `gen_all_comps.py`（预置脚本，**无需 Write 脚本**，无需读任何文档） |
| 从模板复制创建大屏 | 直接用 `template_ops.py copy`，**无需读任何文档** |
| 模板复制遇到问题时 | `references/template-copy-guide.md` |
| 地图组件（JAreaMap 等）| `references/map-guide.md` + **静态数据必须用 `references/map-static-data.md`**（省份GDP/城市散点/飞线/时间轴飞线/分组柱形，禁止自行设计简陋数据） |
| 创建数据源 + SQL数据集 + 图表（完整流程或分步） | 直接用 `datasource_ops.py create` + `comp_ops.py add --create-sql --db-source`，**无需读文档**。遇到参数错误或流程问题时读 `references/datasource-dataset-chart-guide.md` |
| 自写 Java API 接口 + API 数据集 + 批量图表 | 直接参考 `references/pitfalls.md`「完整工作流：自写API接口」章节（Step 1-6），**无需读其他文档**。关键点：`@JimuNoLoginRequired`、`String` 返回裸数组、`dataType='api'`、不同图表 slot_labels 不同、脚本末尾输出接口地址+重启提示 |
| **YApi Mock 系统 + API 数据集**（用户选择 mock 方式） | 直接用 `yapi_ops.py create-mock`（**无需读任何文档**，已封装登录+创建+URL拼接）。固定项目：**proj_id=57（claude AI），catid=1157，basepath=/claude**，禁止探测。Mock URL 格式：`YAPI_BASE/mock/57/claude/{path}`。内置模板：`--template single/multi/pie/gauge/table/bar_multi` |
| 签名接口 / 数据源管理 / NoSQL 数据源 | `references/signing-datasource-guide.md`（含 MongoDB/ES/Redis 创建流程、dbUrl 格式、SQL 前缀语法、已知问题） |
| 组件联动 / 钻取 | 直接用 `linkage_ops.py`，**无需读任何文档**（参数说明见下方「快捷操作：linkage_ops.py」章节）。仅遇到复杂问题时读 `references/linkage-drill-guide.md` |
| 组件外部链接跳转 | 直接用 `link_ops.py`，**无需读任何文档** |
| 组件自定义JS脚本 | 直接用 `link_ops.py set-js` 或 `comp_ops.py edit --set "jsConfig=..."`，**无需读任何文档**（参数说明见下方「快捷操作：自定义JS脚本」章节） |
| 组件组合（JGroup） | `references/group-guide.md` |
| 修改页面配置（背景/水印/宽高） | `references/page-config-guide.md` |
| 字典翻译（jimu_dict） | `references/dict-guide.md` |
| 组件右键操作（图层排序/复制/删除/锁定） | `references/rightclick-actions-guide.md` |
| 遇到奇怪问题时查阅 | `references/pitfalls.md` |
| 组件样式配置路径 | `references/bi-comp-option-config.md`（**仅当 SKILL.md 中未列出目标组件的配置路径时才读取**；JStatsSummary/JCapsuleChart/JGauge/JProgress/JColorBlock/JScrollBoard 等常用组件的配置路径已内联在 SKILL.md「常用组件配置路径速查」章节，无需读 600 行文档） |
| 完整组件类型清单 | `references/bi-component-types.md` |
| 新增组件默认尺寸/数据/option | `references/core-configs/component-defaults.md`（82+ 组件的 w/h/chartData/option 默认值速查） |
| 组件创建流程（addPageComp） | `references/core-configs/addPageComp-logic.md`（newItem 结构、位置计算、保存逻辑） |
| 组件菜单分类树 | `references/core-configs/menu-hierarchy.md`（完整 menuData 层级 + 统计）。**全组件批量生成时无需读取**，SKILL.md 组件速查表已包含全部 compType→中文名映射，脚本中直接硬编码 categories 列表即可 |
| Online表单/设计器表单生成图表（dataType:4） | `references/online-design-form-chart-guide.md`（完整流程、API接口、config结构、字段映射、脚本化方案） |

## SQL数据集创建标准流程（强制）

> **触发条件**：用户说"使用SQL数据集"、"增加SQL数据集"、"统计 xxx 表"、"生成图表"等涉及 SQL 数据集的任何场景，必须严格按以下四步执行，**不得跳过第1步**。

### 第1步：确认数据源（必须询问，禁止擅自选择，无任何例外）

> ⚠️ **即使使用 `comp_ops.py add --create-sql` 也必须先询问数据源，禁止直接执行。** 已观测违规案例（2026-04-09）：用户说"统计 demo 表"，跳过询问直接执行，被用户明确指出错误。

**执行步骤（强制）：**
1. 先运行 `py datasource_ops.py list API_BASE TOKEN` 列出所有可用数据源
2. 向用户展示列表，询问"请问使用哪个数据源？"
3. 等待用户选择后，用选定的数据源 ID 继续执行

```bash
py datasource_ops.py list "http://192.168.1.66:8080/jeecg-boot" "TOKEN"
```

### 第2步：根据业务场景自行编写SQL
- 用户指定数据源后，根据用户描述的业务场景，自行设计并编写合适的 SQL 语句
- **必须使用预置脚本创建 SQL 数据集，禁止直接调用 `/drag/onlDragDatasetHead/add` API**
- 直接调 API 会报"数据源连接失败"错误（即使数据源测试连接成功），原因未明。使用 `comp_ops.py add --create-sql --db-source <ID>` 或 `dataset_ops.py create-sql` 创建
- 原因分析（2026-04-03）：预置脚本内部封装了正确的请求体格式和字段处理逻辑，与直接调 API 的请求体有差异。

### 第3步：创建SQL数据集
- 分组必须使用 **"示例数据集"**（`dataset_ops.py create-sql` 已内置 `--group "示例数据集"` 默认值，自动查询/创建，无需手写代码）
- **直接使用 `dataset_ops.py create-sql`**（已修复 `--db-source` 参数 bug，可正常使用）：
```bash
py dataset_ops.py create-sql $API_BASE $TOKEN \
  --name "数据集名称" --db-source "数据源ID" \
  --sql "SELECT name, value FROM table GROUP BY name" \
  --fields "name:String,value:Integer"
# --group 默认 "示例数据集"，无需额外传参
```
- 数据集创建完成后，**必须执行查询解析验证**确认数据正常返回

### 第4步：后续绑定操作（按需）
- 询问用户是否需要将数据集绑定到图表组件
- 如需要，使用 `comp_ops.py --dataset-name` 或 `comp_ops.py --create-sql` 执行绑定

---

## 文件数据集创建标准流程（多文件 FILES）

> **触发条件**：用户说"多文件数据集"、"FILES"、"上传多个Excel文件"、"JOIN多个文件"，或明确指定需要多个文件关联查询时使用。
> **注意**：用户只说"文件数据集"或提供一个文件时，应使用**单文件数据集（singleFile）**流程（见下一节）。

### 标准流程（5步）

> ⚠️ **实测可行顺序**：先创建空数据集 → 上传文件 → 获取表名 → 更新 SQL + 字段列表；也可先上传再创建（带 content 字段）。两种顺序均可，但先创建再上传更常见（数据集 ID 上传后立即可用）。

| 步骤 | 操作 | 接口地址 |
|------|------|----------|
| 第1步 | 创建空 FILES 数据集（querySql 可为空，后续更新） | `POST /drag/onlDragDatasetHead/add`（`dataType='FILES'`, `dbSource=页面ID`, `parentId=示例数据集分组ID`） |
| 第2步 | 上传所有文件（不传 isSingle，带 reportId） | `POST /jmreport/source/datasource/files/add`（参数：`{reportId: 大屏ID, file: 二进制}`，**不传** `isSingle`） |
| 第3步 | 获取文件列表，提取系统生成的真实表名 | `GET /jmreport/source/datasource/files/get?reportId=大屏ID`，**⚠️ `result` 是 dict**，真实表名在 `json.loads(result['dbUrl'])` 数组的 `name` 字段（格式：`jmf.Sheet1_orders_excel`） |
| 第4步 | 用真实表名编写 SQL，更新数据集（queryById + edit） | 先 `queryById` 取实体，再填入 `querySql` 和 `datasetItemList`，调 `edit` 保存 |
| 第5步 | 添加图表，绑定多文件数据集 | 组件绑定时 `dataSetType: 'FILES'`，`dataSetIzAgent: '1'`，dataMapping 用 SQL 字段别名 |

### 关键要点

- **上传文件自动加 HHMM 时间戳后缀** - `files_ops.py` 在 `upload_file` 中自动将 `products.xlsx` 重命名为 `products_1733.xlsx`（当前时间），确保每次上传文件名唯一，避免 H2 表名冲突
- **value 是 SQL 关键字** - 必须避开 value 关键字，用 `sales`、`amount`、`total` 等替代
- **直接新增** - 用户没有指定使用哪个多文件数据集时，直接创建新的 FILES 数据集，无需先查询是否已存在
- **⚠️ 上传文件必须带 reportId 参数** - 上传接口需要带 `reportId=页面ID` 参数，否则文件会传到错误的数据源
- **⚠️ dbSource 必须与 reportId 一致** - FILES 数据集的 `dbSource` 必须与上传文件时用的 `reportId` 匹配，否则查询时报"表不存在"
- **⚠️ files/get 的 result 是 dict 不是 list** - `result` 返回对象 `{"id":"...","dbUrl":"[{...}]",...}`，不能当 list 迭代，必须先 `json.loads(result['dbUrl'])` 再提取 `name` 字段
- **⚠️ queryFileFieldBySql 需要签名验证，bi_utils 不支持** - 调用会报"时间戳为空"，不能用来解析 SQL 字段。替代方案：字段已知时直接在 `edit` 时手动传 `datasetItemList`，用 `getAllChartData` 验证数据正常
- **⚠️ SQL 别名不能用单引号** - `SELECT col as 'type'` 解析报错，必须用 `SELECT col as type`（不带引号）
- **Excel 表名推断规律** - 文件 `xxx.xlsx` 上传后表名为 `Sheet1_{xxx}_excel`（如 `products.xlsx` → `Sheet1_products_excel`），可在 files/get 解析失败时用作 fallback

### files/get 正确解析方式

```python
# ✅ 正确：result 是 dict，dbUrl 是 JSON 字符串
files_resp = bi_utils._request('GET', '/jmreport/source/datasource/files/get', params={'reportId': PAGE_ID})
result = files_resp.get('result') or {}
file_list = json.loads(result.get('dbUrl', '[]'))  # [{"fileName":"products.xlsx","name":"jmf.Sheet1_products_excel"}, ...]

# 按文件名关键词匹配
products_table = next((f['name'] for f in file_list if 'product' in f['name'].lower()), 'jmf.Sheet1_products_excel')
orders_table   = next((f['name'] for f in file_list if 'order'   in f['name'].lower()), 'jmf.Sheet1_orders_excel')

# ❌ 错误：不能把 result 当 list 迭代
# file_list = (files_resp.get('result') or [])  # result 是 dict 不是 list！
```

### SQL 字段别名与 dataMapping 对应

| SQL 字段别名 | dataMapping mapping | 说明 |
|-------------|---------------------|------|
| `as sales` | `'sales'` | 不能用 `'value'` |
| `as amount` | `'amount'` | 不能用 `'value'` |
| `as type` | `'type'` | 分组字段，**不能加单引号** |
| `as name` | `'name'` | 维度字段，**不能加单引号** |

**详细操作步骤和代码示例见** `references/dataset-guide.md` 的「多文件数据集（FILES）标准操作流程」章节。

---

## 单文件数据集创建标准流程（singleFile）

> **触发条件**：用户说"单文件数据集"、"使用单个Excel文件"、"singleFile"，或明确指定只上传一个文件时使用。

### 关键区别

| 特性 | 单文件 (singleFile) | 多文件 (FILES) |
|------|---------------------|---------------|
| dataType | `'singleFile'` | `'FILES'` |
| 文件数量 | 仅1个文件 | 支持多个文件，可JOIN |
| SQL查询 | `select * from {table_name}` 简单查询 | 支持复杂SQL和JOIN |
| 文件上传参数 | `isSingle=true` | 不传isSingle参数 |
| dbSource | 页面ID（reportId） | 页面ID（reportId） |
| code字段 | 必须等于实际表名 | 可自定义 |

### 标准流程（5步）

| 步骤 | 操作 | 关键点 |
|------|------|--------|
| 第1步 | 上传文件（带isSingle=true） | 必须传`isSingle=true`参数；**`bi_utils._request()` 不支持 `files` 参数，必须用 `requests.post(..., files=...)` 直接上传** |
| 第2步 | 预览获取字段 | 用preview API获取文件实际字段 |
| 第3步 | 创建数据集 | dataType='singleFile'，code=表名，content=文件列表JSON |
| 第4步 | 验证数据 | getAllChartData确认返回数据 |
| 第5步 | 绑定图表 | dataSetType='singleFile'，dataSetIzAgent=''（空字符串） |

### 关键要点

- **querySql必须简单**：`select * from jmf.{table_name}`（**必须含 `jmf.` schema 前缀**），不要构建聚合SQL
- **⚠️ 表名必须从上传响应 `result.dbUrl` 解析（不是 `result.tableName`）**：上传接口响应的 `result` 无 `tableName` 字段，真实表名（含 `jmf.` 前缀）在 `result.dbUrl` JSON 字符串中。正确提取：`json.loads(result.get("dbUrl","[]"))[0].get("name")`，格式如 `jmf.Sheet1_default_excel`。不提取或猜测表名会导致 `getAllChartData` 返回0行
- **dataSetIzAgent为空**：singleFile 单文件类型组件绑定时设为空字符串`''`，不是`'0'`或`'1'`（FILES多文件类型才用`'1'`）
- **字段从数据集获取**：图表的dataMapping使用数据集的字段列表，不要自己构建SQL别名
- **⚠️ 写自定义脚本必须用 Write 工具，禁止 bash heredoc**：bash heredoc 遇 Python 单引号必报错；Python `b"\r\n"` 通过 bash 写入文件变成实际 CRLF 字节导致 SyntaxError。正确做法：先 `Read` 目标路径，再 `Write` 写入完整内容
- **⚠️ `code` 字段必须保留 `jmf.` 前缀，禁止 `replace('jmf.','')`**：系统用 `code` 字段拼接图表聚合SQL（`SELECT ... FROM {code} GROUP BY ...`），若 `code='Sheet1_default_excel'`（缺少前缀）则报 `uncategorized SQLException for SQL [... FROM Sheet1_default_excel ...]`。正确：`code = table_name`（直接用从 `dbUrl` 提取的完整值 `jmf.Sheet1_default_excel`，不做任何 replace）
- **⚠️ 创建数据集时 `parentId` 必须设为"示例数据集"分组的 id**：先调 `GET /drag/onlDragDatasetHead/getAllGroup` 查询分组列表，找到 `name=='示例数据集'` 的节点取其 `id`；若不存在则调 `POST /drag/onlDragDatasetHead/addGroup` 创建后取返回的 `id`；最后将该 `id` 作为 `/add` 请求的 `parentId`。禁止传空字符串 `''`，否则数据集落入根目录无法归类。本环境"示例数据集"固定 id=`1516743332632494082`，可直接硬编码跳过查询

```python
# 标准分组查询逻辑（singleFile 与 SQL/API 数据集统一规范）
group_r = bi_utils._request('GET', '/drag/onlDragDatasetHead/getAllGroup')
groups = (group_r.get('result') or [])
if isinstance(groups, dict):
    groups = groups.get('records', [])
parent_id = ''
for g in groups:
    if g.get('name') == '示例数据集' and g.get('dataType') is None:
        parent_id = g.get('id', '')
        break
if not parent_id:
    add_g = bi_utils._request('POST', '/drag/onlDragDatasetHead/addGroup',
                               data={'name': '示例数据集'})
    parent_id = (add_g.get('result') or {}).get('id', '')
# 然后在 /add 中传 parentId=parent_id
```

**详细操作步骤和代码示例见** `references/dataset-guide.md` 的「创建单文件数据集（singleFile）端到端」章节。

---

## API数据集创建标准流程（强制）

> **触发条件**：用户说"使用API数据集创建图表"、"用API数据集"，且未明确指定接口地址或SQL数据集时，必须严格按以下五步执行。

### 第1步：确认是否已有接口
- 用户**已指定**接口地址 → 直接跳到第4步
- 用户**未指定** → 执行第2步

### 第2步：询问数据来源方式（必须询问，禁止擅自决定）
向用户提问，选择：
1. 使用 **mock** 创建API接口（使用YApi Mock系统）
2. 使用**自定义接口**（在后端编写Java接口）

### 第3步：根据选择收集信息并创建接口

**选择1（mock接口）：**
- 询问：mock系统接口地址、账号密码、业务场景描述
- 参考按需加载表中「YApi Mock 系统 + API 数据集」章节（`references/pitfalls.md`）执行

**选择2（自定义接口）：**
- 询问：接口要编写到哪个Controller文件、业务场景描述
- **禁止自行搜索文件**，必须直接询问用户Controller路径
- 参考按需加载表中「自写 Java API 接口 + API 数据集 + 批量图表」章节（`references/pitfalls.md`）执行

### 第4步：创建API数据集（含字段列表）
- 分组必须使用 **"示例数据集"**（先 `getAllGroup` 查询，取 `id` 作 `parentId`，不存在则 `addGroup` 创建）
- **【强制】`/add` 时直接传 `datasetItemList`**，无需调用 `queryFieldByApi` 解析接口
  - 自写接口或 mock 接口的字段是我们自己定义的，字段名和类型已知，直接写进去
  - `queryFieldByApi` 对外部 URL 常返回空，且增加一次无意义的网络请求
  - **字段名必须用 `datasetItemList`（不是 `onlDragDatasetItemList`）**
- `/add` 返回的 `result` 是**完整对象 dict**，必须用 `(add_r.get('result') or {}).get('id')` 取 DS_ID

```python
ds_fields = [
    {'fieldName': 'name',  'fieldTxt': '名称', 'fieldType': 'String',  'izShow': 'Y', 'orderNum': 0},
    {'fieldName': 'value', 'fieldTxt': '数值', 'fieldType': 'Integer', 'izShow': 'Y', 'orderNum': 1},
    # 多系列图表额外加 type/group 字段
    {'fieldName': 'type',  'fieldTxt': '分组', 'fieldType': 'String',  'izShow': 'Y', 'orderNum': 2},
]
add_r = bi_utils._request('POST', '/drag/onlDragDatasetHead/add', data={
    'name': '数据集名', 'code': 'ds_xxx', 'dataType': 'api',
    'querySql': API_URL, 'apiMethod': 'get', 'parentId': parent_id,
    'datasetItemList': ds_fields,  # ⚠️ 直接传，无需解析接口
})
DS_ID = (add_r.get('result') or {}).get('id')  # ⚠️ result 是 dict，取 .id
```

### 第5步：绑定图表组件（dataMapping 必须按语义映射）
- **【强制】dataMapping 必须按字段语义显式指定，禁止按数组索引顺序映射**
  - 错误做法：`for i, slot in enumerate(slots): mapping.append({'filed': slot, 'mapping': field_names[i]})`（索引 0→0, 1→1 不等于语义对应）
  - 正确做法：按 `slot标签` → `字段名` 的实际含义一一对应

```python
# ✅ 语义映射模板（单系列图表：JLine/JSmoothLine/JStepLine/JArea/JPie/JBar 等）
SINGLE_MAP = [
    {'filed': '维度', 'mapping': 'name'},
    {'filed': '数值', 'mapping': 'value'},
]
# ✅ 语义映射模板（多系列图表：JMultipleLine/DoubleLineBar/JStackBar/JMultipleBar 等）
MULTI_MAP = [
    {'filed': '分组', 'mapping': 'type'},   # 分组字段对应 type/group
    {'filed': '维度', 'mapping': 'name'},   # 维度对应 name（x轴标签）
    {'filed': '数值', 'mapping': 'value'},  # 数值对应 value
]
```

- `fieldOption` 必须同步传入（前端字段面板显示用）：
```python
field_option = [
    {'label': 'name',  'text': '名称', 'type': 'String',  'value': 'name',  'show': 'Y'},
    {'label': 'value', 'text': '数值', 'type': 'Integer', 'value': 'value', 'show': 'Y'},
    {'label': 'type',  'text': '分组', 'type': 'String',  'value': 'type',  'show': 'Y'},
]
```

---

## 大屏特征

- **布局**：绝对定位，坐标和尺寸单位为**像素**（如 x=50, y=280, w=860, h=380）
- **主题**：默认 `dark`，深色背景，亮色/霓虹文字
- **背景图**：默认 `/img/bg/bg4.png`，支持自定义
- **装饰元素**：常用 JDragBorder（边框）、JDragDecoration（装饰条）增强视觉效果
- **典型分辨率**：1920×1080

### 图层背景色规则（强制）

> **严禁将图层背景色设为红色或任何非透明颜色（除非用户明确指定）。** 用户未指定背景色时，`config.background` 和 `config.borderColor` 必须设为 `#FFFFFF00`（透明）。

| 规则 | 说明 |
|------|------|
| **默认值** | `config.background = '#FFFFFF00'`，`config.borderColor = '#FFFFFF00'` |
| **严禁使用 `rgba(0,0,0,0)`** | Ant Design 颜色选择器将其解析为**红色**（色相 0°=红色） |
| **唯一正确的透明写法** | `#FFFFFF00`（带 alpha 通道的十六进制透明白色） |
| **何时设为非透明** | 仅当用户明确指定了具体背景颜色时 |

## 前置条件

用户必须提供 API 地址和 X-Access-Token。

## 执行效率规则（强制）

### 简单操作直接执行，禁止多余探索

**对所有大屏操作（包括创建新大屏、组件增删改查），必须跳过以下步骤直接执行：**
- 禁止触发 brainstorming 流程
- 禁止启动 Explore 子代理去探索源码
- 禁止启动子代理去读 data.ts 默认配置（skill 文档已包含完整信息）
- 禁止启动子代理去分析模板 JSON 结构（template_ops.py 已封装全部逻辑）
- 禁止用 Read 工具读取模板 JSON 文件（92KB+，严重浪费 token）
- 禁止读取 template-copy-guide.md（template_ops.py copy 已实现全部流程）
- 禁止展示设计摘要等待确认（除非用户明确要求确认）
- 禁止使用预置脚本时读取 dataset-guide.md（`dataset_ops.py` / `comp_ops.py --dataset-name` / `comp_ops.py --create-sql` / `comp_ops.py --sql-params` / `proc_ops.py bindcomp` 已封装全部数据集逻辑含存储过程+FreeMarker 参数支持，读 557 行文档浪费 ~5000 token）
- 禁止执行预置脚本前先 `--help` 查看用法（skill 文档已包含完整参数说明）
- 禁止存储过程场景手动写 pymysql 脚本再调 comp_ops.py（`proc_ops.py bindcomp` 一条命令搞定全部流程）

**正确做法：直接使用预置脚本（`template_ops.py`、`comp_ops.py`、`dataset_ops.py`）完成，1-2 轮工具调用。**

### 模板创建快速路径（强制，token 节省 90%+）

**创建整个大屏时，必须走以下快速路径，禁止自定义脚本：**

```
轮次1: cp template_ops.py + bi_utils.py（1 条 Bash 命令）
轮次2: py template_ops.py copy ... --replace '{...}' --board-data '{...}' && echo URL | clip.exe && rm（1 条 Bash 命令）
```

**替换字典构建规则：** 根据用户行业需求，直接构建 `--replace` JSON，无需分析模板内容。各模板中的占位文本是固定的：

**模板1：大数据可视化展示平台（通用/销售/综合类）**

| 占位文本 | 替换为行业术语 |
|---------|--------------|
| 大数据可视化展示平台 | 行业大屏标题 |
| 总金额 | 行业核心指标1名称 |
| 数量 | 行业核心指标2名称 |
| 数量结算率 / 金额结算率 等 | 行业百分比指标 |
| 2017年 / 2018年 | 近两年年份 |
| 结算率 | 行业趋势指标 |
| 年度数据 / 图例数据 | 行业分组标签 |
| 图例1/2/3/4 | 行业分类项 |
| 行1列1~行5列3 | 轮播表业务数据 |

**模板2：北京科技数字化云平台（科技/能源/电力/IoT/设备监控类）**

| 占位文本 | 替换为行业术语 |
|---------|--------------|
| 北京科技数字化云平台 | 行业大屏标题 |
| 网关管理/云组态/设备管理/动态数据/人员管理/监控管理/人员定位/能源管理 | 顶部8个导航菜单项 |
| 功耗总量 | 核心指标标题 |
| 电能耗 / 水能耗 | 双翻牌器标签 |
| 17563 / 11163 | 双翻牌器数值 |
| kw/h | 翻牌器单位 |
| 近七日电能耗 / 近七日水能耗 | 左侧双柱状图标题 |
| 一号~五号机房功率 | 5个仪表盘标题 |
| 设备功率信息 | 中间仪表盘区域标题 |
| 设备列表 / 信息 | 右侧面板标题 |
| 站点号：0001 / 设备状态：正常 / 环境温度：36摄氏度 / 在线设备：20 台 | 右侧4行信息文本 |
| 近七日设备在线数 | 右下折线图标题 |
| 基础折线图 | 折线图标题 |
| 1号~5号机房 / 0374~0378 / 正常 | 滚动表格数据 |
| 功率 / 编号 / 设备名 / 设备状态 | 数据字段名 |

**示例（风力发电行业，使用北京科技数字化云平台模板）：**
```bash
py template_ops.py copy $API_BASE $TOKEN \
  --template "北京科技数字化云平台_1014376428645961728.json" \
  --name "风力发电智慧监控平台" \
  --bg-image "/img/bg/bg4.png" \
  --replace '{"北京科技数字化云平台":"风力发电智慧监控平台","网关管理":"风机管理","云组态":"SCADA系统","设备管理":"机组管理","动态数据":"实时数据","人员管理":"运维管理","监控管理":"故障监控","人员定位":"风场巡检","能源管理":"发电管理","功耗总量":"发电总量","电能耗":"发电量","水能耗":"上网电量","17563":"58260","11163":"42850","kw/h":"万kWh","设备列表":"风机列表","信息":"风场信息","站点号：0001":"风场编号：WF-001","设备状态：正常":"风机状态：正常运行","环境温度：36摄氏度":"平均风速：8.5m/s","在线设备：20 台":"在线风机：126 台","近七日设备在线数":"近七日风机在线数","设备功率信息":"风机功率信息","近七日电能耗":"近七日发电量","近七日水能耗":"近七日弃风量","一号机房功率":"A区风机功率","二号机房功率":"B区风机功率","三号机房功率":"C区风机功率","四号机房功率":"D区风机功率","五号机房功率":"E区风机功率","基础折线图":"风机在线趋势","1号机房":"A区-01号风机","2号机房":"B区-03号风机","3号机房":"C区-07号风机","4号机房":"D区-12号风机","5号机房":"E区-05号风机","0374":"WF-A01","0375":"WF-B03","0376":"WF-C07","0377":"WF-D12","0378":"WF-E05","正常":"运行中","功率":"功率(MW)","编号":"风机编号","设备名":"风机名称","设备状态":"运行状态"}'
```

### 存储过程快速路径（强制，2 轮完成）

**存储过程任务必须走 `proc_ops.py bindcomp`，禁止手动 pymysql + comp_ops.py 分步执行：**

```
轮次1: Read 凭据 + Bash: cp proc_ops.py comp_ops.py bi_utils.py default_configs.json（并行）
轮次2: py proc_ops.py bindcomp ... && rm proc_ops.py comp_ops.py bi_utils.py default_configs.json && echo URL | clip.exe
```

**示例（查询 demo 表）：**
```bash
py proc_ops.py bindcomp "http://192.168.1.66:8080/jeecg-boot" "$TOKEN" \
  --page "PAGE_ID" --comp "JCommonTable" --title "Demo数据表格" \
  --x 50 --y 280 --w 900 --h 450 \
  --proc-name "sp_query_demo" \
  --proc-sql "SELECT id, name, sex, age, birthday, salary_money, email FROM demo ORDER BY create_time DESC" \
  --fields "id:String,name:String,sex:String,age:String,birthday:String,salary_money:String,email:String" \
  --dict "sex=sex"
```

**带参数的存储过程：**
```bash
py proc_ops.py bindcomp ... \
  --proc-name "sp_query_demo_by_sex" \
  --proc-params "IN p_sex varchar(10)" \
  --proc-sql "SELECT id, name, sex, age FROM demo WHERE sex = p_sex ORDER BY create_time DESC" \
  --fields "id:String,name:String,sex:String,age:String"
```

`proc_ops.py bindcomp` 自动完成：从数据源 API 获取 DB 连接信息 → pymysql 创建存储过程 → 验证 CALL → 调用 comp_ops.py 创建数据集+绑定组件。**无需读 dataset-guide.md，无需手写 pymysql 脚本。**

**使用前准备（需额外复制 comp_ops.py + default_configs.json，因为 bindcomp 内部调用 comp_ops.py）：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/proc_ops.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/comp_ops.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/default_configs.json" .
# 执行完后清理
rm proc_ops.py comp_ops.py bi_utils.py default_configs.json
```

### 多字段页面配置修改必须合并（强制）

修改 2 个及以上页面属性时，禁止逐个调用 page_ops.py，必须编写合并脚本（1 次 query + 修改所有字段 + 1 次 edit）。详见 `references/page-config-guide.md`。

### 所有大屏操作以耗时最少为第一优先级（强制）

1. 用户说什么就做什么，不反复权衡
2. 优先用预置脚本，能一条命令解决的不写自定义脚本
3. 多个独立操作必须并行执行
4. 复杂组件从 `default_configs.json` 加载配置
5. 最少工具调用轮次（理想 2 轮：准备 + 执行清理）
6. API 凭据已在上下文中时不重复读取

**耗时目标：**

| 操作类型 | 目标耗时 | 做法 |
|---------|---------|------|
| 单组件增/删/改/查 | ≤30s | comp_ops.py 一条命令（cp + 执行 + rm，共 2 轮） |
| 数据集 + 单组件 | ≤45s | comp_ops.py --create-sql 或 dataset_ops.py + comp_ops.py --dataset-name |
| 复合操作（数据集 + 多组件） | ≤60s | 并行 Bash 调用 |
| 模板复制创建大屏 | ≤60s | template_ops.py copy --replace |
| 存储过程 + 单组件 | ≤45s | proc_ops.py bindcomp 一条命令（cp + 执行 + rm，共 2 轮） |
| 页面配置修改（≥2项） | ≤30s | 合并脚本一次完成 |
| 全组件批量生成（99个） | ≤5s | **预置脚本 `gen_all_comps.py`**，无需 Write 脚本。严格 2 轮：轮次1 Read凭据（已知时跳过），轮次2 `cp 3个文件 && ls && py gen_all_comps.py API TOKEN && rm`（一条命令） |

### 全组件批量生成快速路径（强制）

**用户要求"生成全组件大屏"时，直接调用预置脚本 `gen_all_comps.py`，禁止每次重新 Write 脚本。**

**预置脚本方式（最优，严格 2 轮）：**

```
凭据已知时：仅1轮
轮次1: cp 3个文件 && ls验证 && py gen_all_comps.py API_BASE TOKEN [页面名] && rm（一条命令）
凭据未知时：2轮
轮次1: Read 凭据
轮次2: cp 3个文件 && ls验证 && py gen_all_comps.py API_BASE TOKEN [页面名] && rm（一条命令）
```

**完整命令：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/gen_all_comps.py" "C:/Users/25067/gen_all_comps.py" && \
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" "C:/Users/25067/bi_utils.py" && \
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/default_configs.json" "C:/Users/25067/default_configs.json" && \
ls C:/Users/25067/gen_all_comps.py C:/Users/25067/bi_utils.py C:/Users/25067/default_configs.json && \
cd C:/Users/25067 && py gen_all_comps.py "http://192.168.1.66:8080/jeecg-boot" "TOKEN值" "全组件大屏" && \
rm C:/Users/25067/gen_all_comps.py C:/Users/25067/bi_utils.py C:/Users/25067/default_configs.json
```

> **⚠️ cp 必须与 py 在同一命令链（轮次2），不能放轮次1。**  
> 原因：cp 放轮次1 后文件可能丢失（已观测到：ls 显示成功但实际 py 执行时不存在）

> 脚本已内置 subprocess clip，URL 自动复制到剪贴板，无需外部 `| clip.exe`

**脚本特性：**
- 99 个业务组件（排除 JDragBorder/JDragDecoration/JWeatherForecast）
- 4 列网格布局（440×300，间距 20px），自动计算页面高度（约 8150px）
- 全静态演示数据（从 default_configs.json 加载），无需数据集；传 `--ds-id` 时绑定指定数据集
- 接受命令行参数（2026-04-09 更新，**禁止再 Read 源文件**）：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `API_BASE` | API地址（必填） | — |
| `TOKEN` | Token（必填） | — |
| `[页面名]` | 新建页面时用的名称 | `全组件大屏` |
| `--page-id PAGE_ID` | 使用已有页面（跳过创建，组件追加在已有组件下方） | 新建页面 |
| `--ds-id DS_ID` | 绑定数据集ID（所有可绑定组件均绑此集） | 不绑定 |
| `--ds-type TYPE` | 数据集类型（api/FILES/singleFile） | `api` |
| `--ds-name NAME` | 数据集名称（显示用） | 从数据集自动取 |
| `--ds-fields FIELDS` | 字段列表，格式 `f1:T1,f2:T2`（如 `name:String,type:String,sales:Integer`） | 从数据集自动查 |

**FILES + 已有页面全组件标准命令（3轮完成）：**
```bash
# 轮次1: 分析Excel列名（openpyxl）+ Read凭据
# 轮次2: cp 4个文件 + ls验证 + 创建数据集（--no-chart）+ 生成全组件 + rm
cp "/c/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/files_ops.py" "/c/Users/25067/files_ops.py" && \
cp "/c/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/gen_all_comps.py" "/c/Users/25067/gen_all_comps.py" && \
cp "/c/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" "/c/Users/25067/bi_utils.py" && \
cp "/c/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/default_configs.json" "/c/Users/25067/default_configs.json" && \
ls /c/Users/25067/files_ops.py /c/Users/25067/gen_all_comps.py /c/Users/25067/bi_utils.py /c/Users/25067/default_configs.json && \
cd /c/Users/25067 && \
PYTHONIOENCODING=utf-8 py files_ops.py create-bind API_BASE TOKEN PAGE_ID \
    --files file1.xlsx file2.xlsx --join-on 关联列 --group-by 字段1,字段2 --agg 聚合列 \
    --col-name name --col-type type --col-sales sales --no-chart 2>&1 | tee /tmp/fo.txt && \
DS_ID=$(PYTHONIOENCODING=utf-8 py -c "import re,sys; m=re.search(r'数据集已创建: (\S+)', open('/tmp/fo.txt').read()); print(m.group(1) if m else sys.exit(1))") && \
echo "DS_ID=$DS_ID" && \
PYTHONIOENCODING=utf-8 py gen_all_comps.py API_BASE TOKEN "页面名" \
    --page-id PAGE_ID --ds-id "$DS_ID" --ds-type FILES \
    --ds-name "数据集名称" --ds-fields "name:String,type:String,sales:Integer" && \
rm /c/Users/25067/files_ops.py /c/Users/25067/gen_all_comps.py /c/Users/25067/bi_utils.py /c/Users/25067/default_configs.json
```

**字段映射规则（自动推断，无需手动指定）：**
- `str_fields[0]`（第1个字符串字段）→ 单系列维度 / 表格名称 / 起点名称
- `str_fields[-1]`（最后一个字符串字段）→ 多系列分组 / 终点名称（与 files_ops.py 的 col_type 一致）
- `num_fields[0]`（第1个数字字段）→ 数值

**核心原则（仅自定义修改脚本时参考）：**
1. 必须用 `bi_utils.add_component()` — 自动处理 config 结构（flat dict → JSON string）、size/chart/turnConfig 等字段
2. `_page_components[page_id] = []` 清空缓存 → 批量 add_component → queryById+edit 合并保存
3. `chartData` 在传入前序列化为 JSON 字符串
4. `option.title` 可能是 str 类型，需检查后再赋值
5. `desJson.height` 存页面高度（不是 pageConfig），创建后必须同步更新

**关键代码结构：**
```python
import bi_utils, json, copy
bi_utils.API_BASE = API_BASE
bi_utils.TOKEN = TOKEN

defaults = json.load(open('default_configs.json', 'r', encoding='utf-8'))

# ── 组件 dataMapping 槽位配置（必须从 skill 中 data.ts 获取，禁止自行创建）────────────
# 源码位置：references/core-configs/data.ts
# ⚠️ 严禁自行创建不存在的槽位标签（如"数值2"、"分组2"），必须用源码定义的标签
# 格式: compType: ['维度', '数值'] 或 ['分组', '维度', '数值']
SLOT_CONFIGS = {
    # 单系列图表（2槽位：维度+数值）
    'JBar': ['维度', '数值'], 'JDynamicBar': ['维度', '数值'], 'JCapsuleChart': ['维度', '数值'],
    'JHorizontalBar': ['维度', '数值'], 'JBackgroundBar': ['维度', '数值'],
    'JPie': ['维度', '数值'], 'JRose': ['维度', '数值'], 'JRotatePie': ['维度', '数值'],
    'JLine': ['维度', '数值'], 'JSmoothLine': ['维度', '数值'], 'JStepLine': ['维度', '数值'], 'JArea': ['维度', '数值'],
    'JCustomProgress': ['维度', '数值'], 'JProgress': ['维度', '数值'], 'JListProgress': ['维度', '数值'],
    'JPictorialBar': ['维度', '数值'], 'JPictorial': ['维度', '数值'], 'JGender': ['维度', '数值'],
    'JScatter': ['维度', '数值'], 'JQuadrant': ['维度', '数值'],
    'JFunnel': ['维度', '数值'], 'JPyramidFunnel': ['维度', '数值'], 'JPyramid3D': ['维度', '数值'],
    'JRadar': ['维度', '数值'], 'JCircleRadar': ['维度', '数值'],
    'JRing': ['维度', '数值'], 'JBreakRing': ['维度', '数值'], 'JRingProgress': ['维度', '数值'], 'JActiveRing': ['维度', '数值'], 'JRadialBar': ['维度', '数值'],
    'JRectangle': ['维度', '数值'], 'JBar3d': ['维度', '数值'],
    'JWordCloud': ['维度', '数值'], 'JImgWordCloud': ['维度', '数值'], 'JFlashCloud': ['维度', '数值'],
    # 仪表盘（总计+已用）
    'JGauge': ['总计', '已用'], 'JColorGauge': ['总计', '已用'], 'JAntvGauge': ['总计', '已用'], 'JSemiGauge': ['总计', '已用'],
    # 多系列图表（3槽位：分组+维度+数值）
    'JStackBar': ['分组', '维度', '数值'], 'JMultipleBar': ['分组', '维度', '数值'],
    'JNegativeBar': ['分组', '维度', '数值'], 'JPercentBar': ['分组', '维度', '数值'],
    'JMixLineBar': ['分组', '维度', '数值'], 'JMultipleLine': ['分组', '维度', '数值'],
    'DoubleLineBar': ['分组', '维度', '数值'],  # ⚠️ 双轴图是分组+维度+数值，不是数值2
    'JBubble': ['分组', '维度', '数值'], 'JBarGroup3d': ['分组', '维度', '数值'],
    # 表格列表（名称+数值）
    'JScrollBoard': ['名称', '数值'], 'JScrollTable': ['名称', '数值'], 'JDevHistory': ['名称', '数值'],
    'JCommonTable': ['名称', '数值'], 'JList': ['名称', '数值'],
    'JScrollRankingBoard': ['名称', '数值'], 'JFlashList': ['名称', '数值'], 'JBubbleRank': ['名称', '数值'],
    'JScrollList': ['名称', '数值'],
    # 统计概览（数值1+数值2）
    'JStatsSummary': ['数值1', '数值2'],
    # 翻牌器/颜色块（数值）
    'JCountTo': ['数值'], 'JColorBlock': ['数值'], 'JNumber': ['数值'],
    # 圆形进度图（总计+已用）
    'JRoundProgress': ['总计', '已用'],
    # 水波图（总量+当前）
    'JLiquid': ['总量', '当前'],
    # 轨道环形（名称+数值）
    'JOrbitRing': ['名称', '数值'],
    # 地图类
    'JBubbleMap': ['维度', '数值'], 'JBarMap': ['维度', '数值'], 'JHeatMap': ['维度', '数值'], 'JAreaMap': ['维度', '数值'], 'JGaoDeMap': ['维度', '数值'],
    'JFlyLineMap': ['起点名称', '终点名称'], 'JTotalFlyLineMap': ['起点名称', '终点名称', '分组'],
    'JTotalBarMap': ['分组', '维度', '数值'],
    # 统计进度图/透视表/排行榜
    'JTotalProgress': ['分组', '维度', '数值'], 'JPivotTable': ['维度', '数值'], 'JRankingList': ['名称', '数值'],
    # 卡片滚动
    'JCardScroll': ['内容'], 'JCardCarousel': ['内容'],
}
# UI-only 组件不绑数据集
NO_BIND = {'JImg', 'JCarousel', 'JCustomIcon', 'JText', 'JCurrentTime',
           'JIframe', 'JDragEditor', 'JRadioButton', 'JForm',
           'JPermanentCalendar', 'JSelectRadio', 'JTabToggle', 'JCommon',
           'JVideoPlay', 'JVideoJs'}

# ── 绑定已有数据集时：正确获取字段并构建 DS_CONFIG ──────────────────────────
# ⚠️ 必须用 POST /drag/onlDragDatasetHead/getAllChartData（不是 GET /drag/view/getAllChartData）
# ⚠️ dataMapping.filed 是语义槽位标签（"维度"/"数值"/"分组"），mapping 才是字段名
# ⚠️ 必须包含 fieldOption，否则前端字段选项面板为空
resp = bi_utils._request('GET', '/drag/onlDragDatasetHead/list', params={'pageNo': 1, 'pageSize': 200})
ds = next((r for r in resp.get('result', {}).get('records', []) if r.get('name') == '数据集名称'), None)
DS_ID = ds['id']
items = ds.get('onlDragDatasetItemList') or ds.get('datasetItemList') or []
if not items:
    test_resp = bi_utils._request('POST', '/drag/onlDragDatasetHead/getAllChartData', data={'id': DS_ID})
    test_data = test_resp.get('result', {}).get('data', [])
    if test_data and isinstance(test_data[0], dict):
        items = [{'fieldName': k, 'fieldTxt': k, 'fieldType': 'String', 'izShow': 'Y'} for k in test_data[0].keys()]
field_names = [it['fieldName'] for it in items]
# ─────────────────────────────────────────────────────────────────────────────

# 创建页面（或直接用已有页面 ID）+ 清空缓存
page_id = bi_utils.create_page('全组件大屏', style='bigScreen', theme='dark',
                                background_image='/img/bg/bg4.png')
bi_utils._page_components[page_id] = []

# 扁平网格布局（4列，无分类标题组件）
COLS, COMP_W, COMP_H, MARGIN = 4, 440, 300, 20
START_Y = 100  # 顶部标题区下方

# all_comps = [(key, compType, name), ...] 扁平列表，分类仅作注释
added = 0
for key, comp_type, name in all_comps:
    if key not in defaults:
        continue
    cfg = json.loads(json.dumps(defaults[key]))  # deep copy
    w, h = min(cfg.pop('w', COMP_W), COMP_W), min(cfg.pop('h', COMP_H), COMP_H)
    cfg['background'] = '#FFFFFF00'
    cfg['borderColor'] = '#FFFFFF00'
    opt = cfg.get('option', {})
    if isinstance(opt, str):
        try: opt = json.loads(opt)
        except: opt = {}
        cfg['option'] = opt
    opt_title = opt.get('title')
    if isinstance(opt_title, str):
        opt['title'] = {'text': name, 'show': True}
    elif isinstance(opt_title, dict):
        opt_title['text'] = name
    # 绑定数据集（UI-only 组件跳过，每个组件使用自己的 SLOT_CONFIGS）
    if comp_type not in NO_BIND:
        slot_labels = SLOT_CONFIGS.get(comp_type, ['维度', '数值'])
        if slot_labels:  # 有 dataMapping 的组件才绑定
            data_mapping = []
            for i, slot in enumerate(slot_labels):
                if i < len(field_names):
                    data_mapping.append({'filed': slot, 'mapping': field_names[i]})
                else:
                    data_mapping.append({'filed': slot, 'mapping': ''})
            field_option = [{'label': fn, 'text': fn, 'type': 'String', 'value': fn, 'show': 'Y'} for fn in field_names]
            cfg.update({
                'dataType': 2, 'dataSetId': DS_ID,
                'dataSetName': ds.get('name', ''), 'dataSetType': ds.get('dataType', 'api'),
                'dataSetApi': ds.get('querySql', ''), 'dataSetMethod': ds.get('apiMethod', 'get'),
                'dataSetIzAgent': '1', 'chartData': '[]', 'viewLoading': True, 'paramOption': [],
                'dataMapping': data_mapping, 'fieldOption': field_option,
            })
    if 'chartData' in cfg and not isinstance(cfg['chartData'], str):
        cfg['chartData'] = json.dumps(cfg['chartData'], ensure_ascii=False)
    col, row = added % COLS, added // COLS
    x = MARGIN + col * (COMP_W + MARGIN)
    y = START_Y + row * (COMP_H + MARGIN)
    bi_utils.add_component(page_id, comp_type, name, x, y, w, h, cfg)
    added += 1

# ✅ 优化：queryById + edit 合并，同时保存组件 + 更新页面高度（共3次API，比 save_page+height 两步少2次）
# 禁止：先 bi_utils.save_page() 再单独 queryById+edit 更新高度（5次API），必须用下方合并写法
rows = (added + COLS - 1) // COLS
total_height = START_Y + rows * (COMP_H + MARGIN) + 50
components = bi_utils._page_components[page_id]

raw = bi_utils._request('GET', '/drag/page/queryById', params={'id': page_id})
p = (raw.get('result') or {})
des_raw = p.get('desJson')
des = json.loads(des_raw) if des_raw and isinstance(des_raw, str) else (des_raw if isinstance(des_raw, dict) else {})
des['height'] = total_height
des.setdefault('width', 1920)

info = bi_utils._page_info.get(page_id, {})
payload = {
    'id': page_id,
    'name': info.get('name', ''),
    'template': json.dumps(components, ensure_ascii=False),
    'updateCount': p.get('updateCount', 1),
    'style': info.get('style', 'bigScreen'),
    'theme': info.get('theme', 'dark'),
    'backgroundImage': info.get('backgroundImage', ''),
    'designType': info.get('designType', 100),
    'desJson': json.dumps(des, ensure_ascii=False),
}
bi_utils._request('POST', '/drag/page/edit', data=payload)
print(f'[保存完成] {added}个组件，页面高度 {total_height}px ({rows}行)')
```

**⚠️ 严禁在批量场景下自行构造 comp dict 并 insert 到 template**（config 必须是 JSON 字符串且包含 size/chart/turnConfig 等必要字段，手动构造容易漏字段导致"暂无数据"）

**反模式检查清单（出现任何一条就说明在浪费时间）：**
- **⚠️ 执行 `py script.py` 时不加 `PYTHONIOENCODING=utf-8`**（Windows 默认 GBK 编码，print 含 emoji 或特殊字符必定报 `UnicodeEncodeError`，需要额外一轮修复。**所有 py 命令必须写成 `PYTHONIOENCODING=utf-8 py script.py`**，脚本内部 print 禁用 emoji）
- **⚠️ 直接调用 bi_utils.add_xxx() + save_page() 添加组件**（会覆盖已有组件！必须用 comp_ops.py add）
- **⚠️ 在源码中搜索组件默认配置**（comp_ops.py add 已自动处理，Grep/Read config.ts 浪费大量 token）
- 内心在纠结"这个数据适不适合这种图表"
- 在犹豫用 comp_ops.py 还是自定义脚本（默认用预置脚本）
- 在手写超过 30 行的 config JSON（应从 default_configs.json 加载）
- 两个独立操作串行执行而非并行
- 同一会话中重复读取 API 凭据文件
- 在执行前展开长篇分析或设计讨论
- 启动子代理去探索源码或读取配置文件
- **用 Read 工具读取模板 JSON 文件**（template_ops.py 已封装，无需看原文）
- **启动子代理分析模板中有哪些组件**（占位文本是固定的，见上方表格）
- **template_ops.py copy 能完成时却手写 Python 脚本**（多写 100+ 行 = 浪费 4k+ token）
- **⚠️ 绑定已有数据集前单独调 dataset_ops.py 查询**（comp_ops.py add --dataset-name 已内置自动查询，直接用即可）
- **⚠️ 用户未指定数据来源时擅自使用公开 mock API 创建数据集**（必须先执行 Step 0.1：问①mock系统还是自编接口 ②业务需求描述。直接假设并创建是严重的流程违规）
- **⚠️ 创建 SQL 数据集时跳过询问数据源直接执行**（无论使用 `comp_ops.py add --create-sql` 还是 `dataset_ops.py create-sql`，执行前必须先 `datasource_ops.py list` 列出数据源，再询问用户选择哪个，等待确认后才能执行。已观测违规（2026-04-09）：用户描述"统计 demo 表"时直接执行，被用户指出错误）
- **⚠️ 创建 API 数据集后未回写字段列表**（`getAllChartData` 解析字段后必须构建 `onlDragDatasetItemList` 并调 `edit` 保存回数据集，否则数据集字段配置为空，设计器无法选字段）
- **⚠️ 创建 SQL 数据集后未传字段列表（"查询解析"为空）**（`/add` 时必须同时传 `onlDragDatasetItemList`；若未传则调 `getAllChartData` 后推断字段类型再调 `edit` 补救：`items=[{'fieldName':k,'fieldTxt':k,'fieldType':'Integer' if isinstance(v,(int,float)) else 'String','izShow':'Y','sort':i} for i,(k,v) in enumerate(test_data[0].items())]`，然后 `queryById` + `edit` 回写）
- **dataset_ops.py list --search**（不存在的参数，会报错浪费轮次）
- **⚠️ 创建SQL/API数据集时手写自定义脚本处理分组**（`dataset_ops.py create-sql/create-api` 已内置 `--group "示例数据集"` 默认值，自动查询/创建分组 parentId，无需手写 getAllGroup+add 逻辑）
- **⚠️ 删除大屏时手写 `bi_utils._request('DELETE', ...)`**（直接用 `py page_ops.py delete API TOKEN PAGE_ID`）
- **⚠️ 测试数据源时先 list 查ID再 test --id**（直接用 `py datasource_ops.py test API TOKEN --name "数据源名称"`）
- **读凭据和复制脚本串行执行**（应并行：Read 凭据 + Bash cp 同一轮）
- **⚠️ 使用预置脚本时读取 dataset-guide.md**（`dataset_ops.py` / `comp_ops.py --dataset-name` / `comp_ops.py --create-sql` / `comp_ops.py --sql-params` 已封装全部数据集逻辑含 FreeMarker 参数，无需读 557 行指南文档，浪费 ~5000 token）
- **执行预置脚本前先 `--help` 查看用法**（skill 文档已包含完整参数说明，额外一轮 --help 调用纯属浪费）
- **⚠️ Bash 中用 shell 变量传递 API_BASE/TOKEN 给 py 脚本**（Git Bash 下 `VAR=xxx && py script "$VAR"` 变量可能为空，必须直接内联字面值作为参数）
- **⚠️ 存储过程场景手动 pymysql + comp_ops.py 分步执行**（`proc_ops.py bindcomp` 一条命令完成全流程：获取DB连接→创建SP→创建数据集→绑定组件，分步执行浪费 4+ 轮）
- **⚠️ 存储过程场景读取 dataset-guide.md**（`proc_ops.py` 已封装全部逻辑，无需读 557 行文档）
- **⚠️ FreeMarker 动态SQL通过 bash `--create-sql` 直接传递**（`${age}` 被 shell 解释为空变量，`<#if>` 的 `>` 被解释为重定向，SQL 会被截断。必须用 `--sql-file` 写入文件）
- **⚠️ FreeMarker 判空用 `age??` 或 `age?length`**（JimuReport 只支持内置 `isNotEmpty()` 函数，标准 FreeMarker 判空语法不生效）
- **⚠️ 修改组件配置时读取 `bi-comp-option-config.md`（600行）**（SKILL.md「常用组件配置路径速查」已内联 JStatsSummary/JCapsuleChart/JGauge/JProgress/JColorBlock/JScrollBoard 的完整配置路径表，只有这些组件之外的冷门组件才需读取该文件，浪费 ~6000 token）
- **⚠️ `comp_ops.py edit` 多属性写成位置参数**（多属性必须每个一个 `--set`：`--set "k1=v1" --set "k2=v2"`，不是 `--set "k1=v1" "k2=v2"`）
- **⚠️ 用户明确指定组件名/类型时仍先 `comp_ops.py list`**（用户说"统计概览"/"胶囊图"/"柱状图"等明确组件名时，直接 `comp_ops.py edit --name "xxx"` 或 `--type "JXxx"`，跳过 list，省一轮调用。仅当用户描述模糊如"那个图表"时才需 list 确认）
- **⚠️ 批量绑定数据集时用 GET /drag/view/getAllChartData**（该接口 result 格式不同，`result` 为 None 导致 `'NoneType' object has no attribute 'get'` 报错。**必须用 `POST /drag/onlDragDatasetHead/getAllChartData`**，data 传 `{'id': DS_ID}`，从 `result.data[0]` 的 key 推断字段）
- **⚠️ 批量绑定数据集时 dataMapping.filed 写成字段名**（如 `{"filed":"name","mappingField":"name"}`）。正确：`filed` 是语义槽位标签（`"维度"/"数值"/"分组"`），`mapping` 才是字段名（`{"filed":"维度","mapping":"name"}`）
- **⚠️ dataMapping 按数组索引顺序映射而非语义映射**（`for i, slot in enumerate(slots): {'filed':slot,'mapping':field_names[i]}`）。字段数组顺序与槽位顺序往往不一致，导致多系列图表字段接反（如 `分组→name, 维度→value, 数值→type`）。**必须按语义显式指定**：单系列 `[{'filed':'维度','mapping':'name'},{'filed':'数值','mapping':'value'}]`，多系列 `[{'filed':'分组','mapping':'type'},{'filed':'维度','mapping':'name'},{'filed':'数值','mapping':'value'}]`
- **⚠️ 创建自定义API/Mock数据集后仍调用 queryFieldByApi 解析**。字段是自己定义的，直接在 `/add` 时传 `datasetItemList` 即可，`queryFieldByApi` 对外部URL常返回空且浪费请求
- **⚠️ 批量生成全组件时全局统一 dataMapping**（所有组件用同一个 SLOT_LABELS）。正确：每个组件使用自己的 `SLOT_CONFIGS[comp_type]`，单系列图表=`['维度','数值']`，多系列图表=`['分组','维度','数值']`，仪表盘=`['总计','已用']`
- **⚠️ 批量绑定数据集时缺少 fieldOption**（不设 fieldOption 前端字段选项面板为空，用户无法在设计器中修改字段映射）。格式：`[{"label":"name","text":"name","type":"String","value":"name","show":"Y"},...]`
- **⚠️ 批量绑定数据集时缺少必要字段**（`dataSetName/dataSetType/dataSetApi/dataSetMethod/dataSetIzAgent/paramOption/viewLoading` 缺一不可，用 `cfg.update(copy.deepcopy(DS_CONFIG))` 一次性写入所有字段）
- **⚠️ 批量添加组件时手动构造 comp dict 而非用 bi_utils.add_component()**（手动构造的 config 缺少 size/chart/turnConfig 等必要字段，且 config 必须是 JSON 字符串不是 dict，导致全部"暂无数据"。必须用 `bi_utils.add_component()` 处理结构转换）
- **⚠️ 批量添加 95 个组件却逐个调 comp_ops.py add**（每次调用 = 1次query + 1次save = 2次API请求，95个组件 = 190次请求。正确做法：自定义脚本 1次query + 内存批量add + 1次save = 2次请求，0.6s完成）
- **⚠️ 批量生成时用 compType 作为图层名**（如 componentName='JBar'，用户在设计器看不懂。必须用 `menu-hierarchy.md` 中的中文名：JBar→基础柱形图、JPie→饼图、JStatsSummary_1→统计概览(卡片式) 等，脚本中维护 key→中文名映射表）
- **⚠️ 全组件生成时为每个分类生成 JText 标题组件**（分类如"柱形图"、"饼状图"等仅用于代码注释分组，不应渲染为 JText 组件。用户在设计器中会看到 20 个多余的文本图层，且占用额外高度。组件应扁平排列，分类仅作注释）
- **⚠️ 全组件生成时包含装饰边框/装饰条/天气预报**（JDragBorder 13种 + JDragDecoration 12种 = 25个纯装饰组件，JWeatherForecast 需特殊API，都不是业务数据组件，应排除）
- **⚠️ 全组件批量生成时读取 menu-hierarchy.md**（SKILL.md 组件速查表已包含全部 compType→中文名映射，脚本中直接硬编码 categories 列表即可，读 253 行文档浪费 ~2500 token + 1 轮调用）
- **⚠️ default_configs.json 中新增变体 key 用中文后缀**（变体 key 必须用 `_数字` 格式如 `JScrollList_2`，禁止用中文如 ~~`JScrollList_滚动列表(多行+序号)`~~。`_resolve_comp_type` 需要从 key 提取实际 Vue 组件类型名，中文后缀虽然正则也能匹配但不规范，且容易引发编码问题）
- **⚠️ 生成全组件大屏时自行 Write 脚本**（`gen_all_comps.py` 已固化在 skill `references/scripts/` 中，直接 cp 3 个文件 + py 即可，Write 脚本浪费 1 轮调用 + 约 60-90s 用户等待时间）
- **⚠️ 全组件批量生成超过 2 轮工具调用**（凭据已知时仅需 1 轮：`cp gen_all_comps.py+bi_utils.py+default_configs.json && ls && py gen_all_comps.py API TOKEN && rm`；凭据未知时 2 轮：Read凭据 → cp+py+rm。Write 脚本、清理分开、cp 放轮次1 都是浪费）
- **⚠️ 更新预置脚本时先 Read 整个源文件（2026-04-09 实测浪费 4 轮）**（gen_all_comps.py / files_ops.py 等预置脚本的结构已在 SKILL.md 中完整记录，无需 Read 源文件再修改。需要新增功能时（如 `--page-id`）**直接 Write 完整新版本**，禁止先 Read 再 Edit——已观测：Read gen_all_comps.py 分 4 段 + Read files_ops.py 分 4 段，共浪费 8 轮 Read 调用）
- **⚠️ 需确认 files_ops.py create-bind 输出的 DS_ID 时再去 Read 源码**（输出格式已固定：`[1] 数据集已创建: {DS_ID}`，直接用 Python 正则 `re.search(r'数据集已创建: (\S+)', out)` 提取，无需读源码确认）
- **⚠️ FILES + 已有页面全组件：gen_all_comps.py 已支持 `--page-id` / `--ds-id`（2026-04-09 新增）**（正确流程 3 轮：轮次1 Read凭据+openpyxl分析Excel → 轮次2 Write更新gen_all_comps.py（仅当需要新功能时）→ 轮次3 cp 4个文件 + py files_ops.py create-bind --no-chart 捕获DS_ID + py gen_all_comps.py --page-id PAGE_ID --ds-id DS_ID --ds-type FILES --ds-fields "name:String,type:String,sales:Integer" + rm。**现在此场景已有完整支持，直接执行即可，无需再 Read/Write 脚本**）
- **⚠️ Write 脚本时写占位符 TOKEN/API_BASE 再单独 Edit 更新**（凭据已在 MEMORY.md 上下文中，必须在 Write 时直接填入最终值，禁止先写 `PLACEHOLDER_TOKEN` 等占位符再用 Edit 替换——每次额外 Edit = 多 1 轮调用 = 多约 60-90s 用户等待时间）
- **⚠️ cp 放在轮次1 而非轮次2**（已观测到：cp bi_utils.py 在轮次1 ls 显示存在，但轮次2 py 执行时文件不存在，导致 ModuleNotFoundError 浪费 2 轮修复。**必须把 cp 放在轮次2 与 py 同一命令链**，cp 失败立即终止不执行 py）
- **⚠️ Git Bash 下 cp 目标路径用 `C:/Users/` 格式静默失败（2026-04-09 实测）**（`cp "C:/Users/25067/.../bi_utils.py" "C:/Users/25067/"` 在 Git Bash 中无报错、ls 看似通过，但文件实际不存在，随后 `py` 报 `ModuleNotFoundError` 或 `No such file or directory`，浪费 2-3 轮修复。**必须用 Unix 格式**：`cp "/c/Users/25067/.../bi_utils.py" "/c/Users/25067/bi_utils.py"`，验证也用 `ls /c/Users/25067/bi_utils.py`）
- **⚠️ 写自定义脚本用 bash heredoc 而非 Write 工具**（bash heredoc `<< 'EOF'` 在 Python 内容含单引号时必定报 `unexpected EOF`；`b"\r\n"` 通过 bash 写入文件时变成实际 CR+LF 字节导致 `SyntaxError: unterminated string literal`。**必须用 Write 工具写脚本**：先 Read 目标路径，再 Write 完整内容，无任何转义问题——已观测：本次因 heredoc 问题浪费 4 次失败 + 约 3 分钟）
- **⚠️ singleFile 上传后用 `result.tableName` 提取表名**（上传响应 `result` 无 `tableName`/`table`/`code` 字段；真实表名含 `jmf.` 前缀，在 `result.dbUrl` JSON 字符串的 `name` 字段中。正确提取：`json.loads(result.get("dbUrl","[]"))[0].get("name")` → 如 `jmf.Sheet1_default_excel`。猜测表名缺少 `jmf.` 前缀 → SQL 报错 → `getAllChartData` 返回0行 → 字段为空 → 图表无数据）
- **⚠️ singleFile 数据集 `code` 字段禁止去掉 `jmf.` 前缀**（系统用 `code` 字段拼接图表聚合SQL：`SELECT ... FROM {code} GROUP BY ...`。若写成 `code='Sheet1_default_excel'` 则运行时报 `uncategorized SQLException for SQL [... FROM Sheet1_default_excel ...]`。**必须**：`code = table_name`，即直接用从 `dbUrl` 提取的完整值 `jmf.Sheet1_default_excel`，禁止做 `replace('jmf.', '')` 或其他裁剪操作）
- **⚠️ singleFile 文件上传必须用 `requests.post()` 直接调用，禁止用 `bi_utils._request()`**（`bi_utils._request()` 底层只处理 JSON/form 请求，没有 `files` 参数，调用时报 `TypeError: _request() got an unexpected keyword argument 'files'`。**正确写法**：`import requests; requests.post(f'{API_BASE}/jmreport/source/datasource/files/add', headers={'X-Access-Token': TOKEN}, params={'reportId': PAGE_ID, 'isSingle': 'true'}, files={'file': (filename, file_data, 'application/octet-stream')})`（2026-04-09 实测）
- **⚠️ comp_ops.py 参数顺序：子命令在前，API_BASE/TOKEN/PAGE_ID 随后（禁止颠倒）**（正确：`py comp_ops.py add API_BASE TOKEN PAGE_ID --comp JPie ...`；错误：`py comp_ops.py API_BASE TOKEN PAGE_ID add --comp JPie ...`。argparse 把 API_BASE 当作 subcommand 解析，报 `argument command: invalid choice: 'http://...'`，浪费 1 轮修复（2026-04-09 实测））
- **⚠️ 自定义脚本内用 subprocess 调用其他脚本时禁止使用 `/c/Users/...` Unix 路径**（Python subprocess 在 Windows 下不经过 Git Bash 路径转换，`/c/Users/25067/.../comp_ops.py` 会被直接拼成 `C:\c\Users\25067\...`，报 `No such file or directory`。**正确做法**：提前 `cp` 脚本到当前工作目录，subprocess 直接调用脚本名，无需绝对路径（2026-04-09 实测））
- **⚠️ singleFile 数据集创建时 `parentId` 不能为空**（`parentId=''` 导致数据集落入根目录，与其他数据集管理规范不一致。**必须**先 `getAllGroup` 找"示例数据集"分组取 `id`，不存在则 `addGroup` 创建，再将该 `id` 传给 `/add` 的 `parentId`。本环境固定 id=`1516743332632494082`，可直接硬编码）
- **⚠️ 全组件批量生成：先 save_page() 再单独 queryById+edit 更新高度（5次API）**。正确做法：跳过 `bi_utils.save_page()`，直接 1次 `queryById`（取 updateCount + desJson）+ 1次 `edit`（同时传 template + desJson），共3次API。`save_page` 内部会再 `queryById` 一次，与后续高度更新合计5次，浪费2次往返
- **⚠️ 任何场景下 cp 依赖文件和 Write 脚本串行执行**（cp 和 Write 是独立操作，必须在同一轮并行发出，串行多花 1 轮）
- **⚠️ 自定义脚本创建组件时 `size` 字段为空 `{}`**（config 内部必须设 `size: {width: w, height: h}`，顶层也必须设 `size: {width, height}`，同理 `chart: {subclass, category}`、`turnConfig: {}`、`linkageConfig: []`。缺失 size 会导致组件初始不渲染，需手动拖拽才显示。`bi_utils.add_component()` 第 420 行已自动处理，仅自定义脚本直接操作 template 时需手动补全）

## 交互流程

### Step 0: 解析用户需求

| 信息 | 默认值 |
|------|--------|
| 页面名称 | 用户指定 |
| 主题 | dark |
| 背景图 | `/img/bg/bg4.png` |
| 组件列表 | 从描述中解析 |

### Step 0.1: 数据来源确认（强制，用户未明确指定时必须询问）

**触发条件：** 用户没有明确说明数据来自哪里（没有给出接口地址、没有指定数据集、没有说用 SQL/mock/自己写代码），则**必须先问用户以下两个问题，不得擅自假设或跳过**：

> **问题一：接口来源**
> 使用 **mock 系统** 还是 **自己编写代码**？
> - **mock 系统**：请提供 mock 服务地址 + 账号密码（如 YApi：`https://xxx.com/login`）
> - **自己编写代码**：请提供代码存放路径（Java Controller 文件全路径）
>
> **问题二：接口需要实现什么业务需求？**
> 描述各组件要展示的数据内容（参考大屏中的静态数据格式设计接口）

**可跳过询问、直接执行的情况：**
- 用户已明确说"使用 mock 系统"并提供了地址和账号
- 用户已明确说"写接口"并提供了文件路径
- 用户指定了已有数据集名称或 SQL 数据源
- 任务不涉及数据集创建（如纯样式修改、组件位移、删除等）

### Step 0.5: 模板匹配（优先使用模板布局）

**生成整个大屏时，必须先匹配模板，复用已有布局。**

**模板目录**：`references/templates/bigScreen/`（10 个经典大屏模板 JSON）

**模板名→文件名索引（直接使用，禁止 Glob 搜索）：**

| 模板名称 | 文件名 |
|---------|--------|
| 乡村振兴普惠金融服务平台 | `乡村振兴普惠金融服务平台_1024608431274250240.json` |
| 北京市污水排放总量 | `北京市污水排放总量_1022392593179791360.json` |
| 北京科技数字化云平台 | `北京科技数字化云平台_1014376428645961728.json` |
| 医院实时数据监控 | `医院实时数据监控_1011800681234354176.json` |
| 旅游数据分析中心大屏 | `旅游数据分析中心大屏_1016994272231608320.json` |
| 杭州房地产市场宏观监控 | `杭州房地产市场宏观监控_1024545852833189888.json` |
| 警务监控系统 | `警务监控系统_1024545264544305152.json` |
| 车辆分布图 | `车辆分布图_1017325669831987200.json` |
| 集团综合数据大屏 | `集团综合数据大屏_1151069555267260416.json` |
| 香山公园客流大数据 | `香山公园客流大数据_1027085484978388992.json` |

| 用户需求关键词 | 推荐模板 |
|---------------|---------|
| 销售/订单/交易/通用/综合/驾驶舱 | 集团综合数据大屏 |
| 医院/医疗/医药/机构/校园/人员管理 | 医院实时数据监控 |
| 监控/安防/警务 | 警务监控系统 |
| 旅游/公园/客流 | 旅游数据分析中心大屏、香山公园客流大数据 |
| 房地产/城市/环境 | 杭州房地产市场宏观监控、北京市污水排放总量 |
| 科技/数字化/IoT/能源/电力/风电/光伏 | 北京科技数字化云平台 |
| 金融/银行/乡村 | 乡村振兴普惠金融服务平台 |
| 车辆/交通/地图 | 车辆分布图 |

**模板选择优先级（强制，按顺序匹配）：**

1. **精确匹配** → 从上方关键词表找到直接对应的模板，使用「模板复制方式」创建大屏，保留布局和装饰，仅替换业务数据和标题文字
2. **备选三模板** → 没有精确匹配时，从以下三个模板中选最合适的：
   - `北京科技数字化云平台`（科技/工业/设备/IoT/能源类）
   - `北京市污水排放总量`（环境/城市/数据监测类）
   - `医院实时数据监控`（机构/人员/综合管理类）
3. **兜底模板** → 以上都不合适时，才选择 `集团综合数据大屏`

模板复制的详细流程（ID 映射、JTabToggle/JGroup 引用更新、边界检查等）见 `references/template-copy-guide.md`。

> **重要**：只有在用户明确要求"不使用模板"或"从零创建"时，才跳过模板匹配，直接使用 bi_utils 默认组件函数逐个添加。

### Step 1: 识别组件并选择类型

**完整组件名称→类型速查（按分类）：**

> 用户说组件名时直接查此表获取 compType，**禁止 Grep 搜索源码**。

**柱形图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 基础柱形图 | `JBar` | `comp_ops.py add --comp JBar` |
| 堆叠柱形图 | `JStackBar` | `comp_ops.py add --comp JStackBar` |
| 动态柱形图 | `JDynamicBar` | `comp_ops.py add --comp JDynamicBar` |
| 胶囊图 | `JCapsuleChart` | `comp_ops.py add --comp JCapsuleChart` |
| 基础条形图 | `JHorizontalBar` | `comp_ops.py add --comp JHorizontalBar` |
| 背景柱形图 | `JBackgroundBar` | `comp_ops.py add --comp JBackgroundBar` |
| 对比柱形图 | `JMultipleBar` | `comp_ops.py add --comp JMultipleBar` |
| 正负条形图 | `JNegativeBar` | `comp_ops.py add --comp JNegativeBar` |
| 百分比条形图 | `JPercentBar` | `comp_ops.py add --comp JPercentBar` |
| 折柱图 | `JMixLineBar` | `comp_ops.py add --comp JMixLineBar` |

**饼状图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 饼图 | `JPie` | `comp_ops.py add --comp JPie` |
| 南丁格尔玫瑰图 | `JRose` | `comp_ops.py add --comp JRose` |
| 旋转饼图 | `JRotatePie` | `comp_ops.py add --comp JRotatePie` |

**折线图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 基础折线图 | `JLine` | `comp_ops.py add --comp JLine` |
| 平滑曲线图 | `JSmoothLine` | `comp_ops.py add --comp JSmoothLine` |
| 阶梯折线图 | `JStepLine` | `comp_ops.py add --comp JStepLine` |
| 面积图 | `JArea` | `comp_ops.py add --comp JArea` |
| 对比折线图 | `JMultipleLine` | `comp_ops.py add --comp JMultipleLine` |
| 双轴图 | `DoubleLineBar` | `comp_ops.py add --comp DoubleLineBar` |

**进度图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 基础进度图 | `JCustomProgress` | `comp_ops.py add --comp JCustomProgress` |
| 进度图 | `JProgress` | `comp_ops.py add --comp JProgress` |
| 列表进度图 | `JListProgress` | `comp_ops.py add --comp JListProgress` |
| 圆形进度图 | `JRoundProgress` | `comp_ops.py add --comp JRoundProgress` |
| 水波图 | `JLiquid` | `comp_ops.py add --comp JLiquid` |

**象形图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 象形柱图 | `JPictorialBar` | `comp_ops.py add --comp JPictorialBar` |
| 象形图 | `JPictorial` | `comp_ops.py add --comp JPictorial` |
| 男女占比 | `JGender` | `comp_ops.py add --comp JGender` |

**仪表盘类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 基础仪表盘 | `JGauge` | `comp_ops.py add --comp JGauge` |
| 多色仪表盘 | `JColorGauge` | `comp_ops.py add --comp JColorGauge` |
| 渐变仪表盘 | `JAntvGauge` | `comp_ops.py add --comp JAntvGauge` |
| 半圆仪表盘 | `JSemiGauge` | `comp_ops.py add --comp JSemiGauge` |

**散点图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 普通散点图 | `JScatter` | `comp_ops.py add --comp JScatter` |
| 象限图 | `JQuadrant` | `comp_ops.py add --comp JQuadrant` |
| 气泡图 | `JBubble` | `comp_ops.py add --comp JBubble` |

**漏斗图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 普通漏斗图 | `JFunnel` | `comp_ops.py add --comp JFunnel` |
| 金字塔漏斗图 | `JPyramidFunnel` | `comp_ops.py add --comp JPyramidFunnel` |
| 3D金字塔 | `JPyramid3D` | `comp_ops.py add --comp JPyramid3D` |

**雷达图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 普通雷达图 | `JRadar` | `comp_ops.py add --comp JRadar` |
| 圆形雷达图 | `JCircleRadar` | `comp_ops.py add --comp JCircleRadar` |

**环形图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 饼状环形图 | `JRing` | `comp_ops.py add --comp JRing` |
| 多色环形图 | `JBreakRing` | `comp_ops.py add --comp JBreakRing` |
| 基础环形图 | `JRingProgress` | `comp_ops.py add --comp JRingProgress` |
| 动态环形图 | `JActiveRing` | `comp_ops.py add --comp JActiveRing` |
| 玉珏图 | `JRadialBar` | `comp_ops.py add --comp JRadialBar` |

**矩形图/3D图表：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 矩形图 | `JRectangle` | `comp_ops.py add --comp JRectangle` |
| 3D柱形图 | `JBar3d` | `comp_ops.py add --comp JBar3d` |
| 3D分组柱形图 | `JBarGroup3d` | `comp_ops.py add --comp JBarGroup3d` |

**表格/列表类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 轮播表 | `JScrollBoard` | `comp_ops.py add --comp JScrollBoard` |
| 表格 | `JScrollTable` | `comp_ops.py add --comp JScrollTable` |
| 发展历程 | `JDevHistory` | `comp_ops.py add --comp JDevHistory` |
| 数据表格 | `JCommonTable` | `comp_ops.py add --comp JCommonTable` |
| 数据列表 | `JList` | `comp_ops.py add --comp JList` |
| 排行榜 | `JScrollRankingBoard` | `comp_ops.py add --comp JScrollRankingBoard` |
| 个性排名 | `JFlashList` | `comp_ops.py add --comp JFlashList` |
| 气泡排名 | `JBubbleRank` | `comp_ops.py add --comp JBubbleRank` |
| 滚动列表(单行) | `JScrollList` | `comp_ops.py add --comp JScrollList_1` |
| 滚动列表(多行+序号) | `JScrollList` | `comp_ops.py add --comp JScrollList_2` |
| 滚动列表(带表头) | `JScrollList` | `comp_ops.py add --comp JScrollList_3` |

**统计/轮播类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 日历 | `JPermanentCalendar` | `comp_ops.py add --comp JPermanentCalendar` |
| 卡片滚动(横向) | `JCardScroll` | `comp_ops.py add --comp JCardScroll_1` |
| 卡片滚动(竖向+序号) | `JCardScroll` | `comp_ops.py add --comp JCardScroll_2` |
| 卡片滚动(高亮) | `JCardScroll` | `comp_ops.py add --comp JCardScroll_3` |
| 卡片轮播 | `JCardCarousel` | `comp_ops.py add --comp JCardCarousel` |
| 统计概览(卡片) | `JStatsSummary` | `comp_ops.py add --comp JStatsSummary_1` |
| 统计概览(背景) | `JStatsSummary` | `comp_ops.py add --comp JStatsSummary_2` |
| 统计概览(高亮) | `JStatsSummary` | `comp_ops.py add --comp JStatsSummary_3` |

**装饰类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 边框1~13 | `JDragBorder` | `comp_ops.py add --comp JDragBorder` |
| 装饰1~12 | `JDragDecoration` | `comp_ops.py add --comp JDragDecoration` |
| 图片 | `JImg` | `comp_ops.py add --comp JImg` |
| 轮播图 | `JCarousel` | `comp_ops.py add --comp JCarousel` |
| 图标 | `JCustomIcon` | `comp_ops.py add --comp JCustomIcon` |

**文字类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 文本 | `JText` | `comp_ops.py add --comp JText` |
| 翻牌器 | `JCountTo` | `comp_ops.py add --comp JCountTo` |
| 颜色块 | `JColorBlock` | `comp_ops.py add --comp JColorBlock` |
| 当前时间 | `JCurrentTime` | `comp_ops.py add --comp JCurrentTime` |
| 数值 | `JNumber` | `comp_ops.py add --comp JNumber` |
| 轨道环形文字 | `JOrbitRing` | `comp_ops.py add --comp JOrbitRing` |

**字符云类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 字符云 | `JWordCloud` | `comp_ops.py add --comp JWordCloud` |
| 图层字符云 | `JImgWordCloud` | `comp_ops.py add --comp JImgWordCloud` |
| 闪动字符云 | `JFlashCloud` | `comp_ops.py add --comp JFlashCloud` |

**地图类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 散点地图 | `JBubbleMap` | `comp_ops.py add --comp JBubbleMap` |
| 飞线地图 | `JFlyLineMap` | `comp_ops.py add --comp JFlyLineMap` |
| 柱形地图 | `JBarMap` | `comp_ops.py add --comp JBarMap` |
| 时间轴飞线地图 | `JTotalFlyLineMap` | `comp_ops.py add --comp JTotalFlyLineMap` |
| 柱形排名地图 | `JTotalBarMap` | `comp_ops.py add --comp JTotalBarMap` |
| 热力地图 | `JHeatMap` | `comp_ops.py add --comp JHeatMap` |
| 区域地图 | `JAreaMap` | `comp_ops.py add --comp JAreaMap`（读 `references/map-guide.md`） |
| 高德地图 | `JGaoDeMap` | `comp_ops.py add --comp JGaoDeMap` |

**视频类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 播放器 | `JVideoPlay` | `comp_ops.py add --comp JVideoPlay` |
| RTMP播放器 | `JVideoJs` | `comp_ops.py add --comp JVideoJs` |

**其他类：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 选项卡 | `JSelectRadio` | 自定义脚本（需配置 `compShowConfig` 关联组件，见下方专节） |
| 导航切换 | `JTabToggle` | `comp_ops.py add --comp JTabToggle` |
| 表单 | `JForm` | `comp_ops.py add --comp JForm` |
| Iframe | `JIframe` | `comp_ops.py add --comp JIframe` |
| 按钮 | `JRadioButton` | `comp_ops.py add --comp JRadioButton` |
| 富文本 | `JDragEditor` | `comp_ops.py add --comp JDragEditor` |
| 通用组件 | `JCommon` | `comp_ops.py add --comp JCommon` |
| 自定义组件 | `JCustomEchart` | `comp_ops.py add --comp JCustomEchart` |
| 统计进度图 | `JTotalProgress` | `comp_ops.py add --comp JTotalProgress` |
| 透视表 | `JPivotTable` | `comp_ops.py add --comp JPivotTable` |
| 排行榜(自定义) | `JRankingList` | `comp_ops.py add --comp JRankingList` |

**天气预报（特殊，不支持 comp_ops.py）：**

| 组件名称 | compType | 添加方式 |
|---------|----------|---------|
| 天气预报(滚动版) | `JWeatherForecast` | 自定义脚本（template=11） |
| 天气预报(横线版) | `JWeatherForecast` | 自定义脚本（template=34） |
| 天气预报(带背景) | `JWeatherForecast` | 自定义脚本（template=21） |
| 天气预报(好123版) | `JWeatherForecast` | 自定义脚本（template=12） |
| 天气预报(温度计版) | `JWeatherForecast` | 自定义脚本（template=27） |
| 天气预报(列表文字版) | `JWeatherForecast` | 自定义脚本（template=94） |

### JWeatherForecast 天气预报组件（特殊组件，需自定义脚本）

> **comp_ops.py 不支持 JWeatherForecast**，必须用自定义脚本直接操作 template 数组添加。
> **dataType 必须为 1**（自动获取天气数据），不是 0。chartData 为 `"[]"`。

| 版本 | template 值 | 默认尺寸 (w×h) | fontColor | bgColor |
|------|------------|----------------|-----------|---------|
| 滚动版 | 11 | 311×47 | #fff | #ffffff00 |
| 横线版 | 34 | 300×30 | #fff | #ffffff00 |
| 带背景 | 21 | 415×131 | #000 | #ffffff00 |
| 好123版 | 12 | 318×61 | #fff | #ffffff00 |
| 温度计版 | 27 | 400×266 | #fff | #ffffff |
| 列表文字版 | 94 | 257×47 | #fff | #ffffff00 |

**添加模板（直接复制修改 template 和坐标即可）：**
```python
import sys, json
sys.path.insert(0, '.')
import bi_utils
bi_utils.API_BASE = API_BASE
bi_utils.TOKEN = TOKEN

page = bi_utils.query_page(PAGE_ID)
tmpl = page.get('template', [])

comp = {
    "i": bi_utils._gen_uuid(),
    "component": "JWeatherForecast",
    "componentName": "天气预报-滚动版",
    "x": 50, "y": 280, "w": 311, "h": 47,
    "dataType": 1,
    "chartData": "[]",
    "config": {
        "background": "#FFFFFF00",
        "borderColor": "#FFFFFF00",
        "card": {"title": ""},
        "option": {
            "city": "",
            "template": 11,  # 改这个值切换版本
            "num": 2,
            "fontSize": 16,
            "fontColor": "#fff",
            "bgColor": "#ffffff00",
            "url": ""
        }
    },
    "dataMapping": {}
}

tmpl.insert(0, comp)
bi_utils._page_components[PAGE_ID] = tmpl
bi_utils.save_page(PAGE_ID)
```

### JSelectRadio 选项卡组件（需自定义脚本实现组件关联）

> **选项卡的核心功能是「组件可见性控制」**：点击不同选项卡，显示/隐藏关联的组件。
> comp_ops.py 可以添加基础选项卡，但**无法配置 `compShowConfig`（组件关联）**，需自定义脚本。
> 参考文档：https://help.jimureport.com/biScreen/componentconfig/other/tab/

**两种显示风格**：选项卡样式（默认）、下拉菜单样式

**chartData 格式**（定义选项卡标签）：
```json
[
  {"label": "折线图", "value": "1"},
  {"label": "柱形图", "value": "2"}
]
```

**核心配置：`config.compShowConfig`**（控制组件显示/隐藏）：
```json
[
  {"selectVal": "1", "compVals": ["目标组件ID_1"]},
  {"selectVal": "2", "compVals": ["目标组件ID_2", "目标组件ID_3"]}
]
```

| 字段 | 说明 |
|------|------|
| `selectVal` | 对应 chartData 中某项的 `value` |
| `compVals` | 组件 ID 数组（`comp['i']`），选中该选项时这些组件显示，其余隐藏 |

**显隐逻辑（源码 `SelectRadio.vue`）**：
- 遍历 `compShowConfig`，对每条规则中 `compVals` 引用的组件设置 `visible = (selectVal == 当前选中值)`
- 支持组合组件（JGroup）内部的子组件查找
- 每个选项可关联**多个组件**同时显示

**option 常用配置路径：**

| 说明 | 配置路径 | 示例值 |
|------|---------|--------|
| 类型（选项卡/下拉） | `option.type` | `radio` / `select` |
| 字体颜色（未选中） | `option.fontColor` | `#ffffff` |
| 字体大小 | `option.fontSize` | `14` |
| 选项卡间距 | `option.tabGap` | `10` |
| 未选中背景色 | `option.bgColor` | `#FFFFFF00` |
| 未选中边框颜色 | `option.borderColor` | `#0f66ff` |
| 未选中边框宽度 | `option.borderWidth` | `1` |
| 选中文字颜色 | `option.activeFontColor` | `#ffffff` |
| 选中背景色 | `option.activeBgColor` | `#0f66ff` |
| 选中边框颜色 | `option.activeBorderColor` | `#0f66ff` |
| 背景图片（未选中） | `option.bgImgUrl` | URL |
| 背景图片（选中） | `option.activeBgImgUrl` | URL |

**添加选项卡并关联组件的脚本模板（自定义脚本，非 comp_ops.py）：**

```python
import sys, json
sys.path.insert(0, '.')
import bi_utils
bi_utils.API_BASE = API_BASE
bi_utils.TOKEN = TOKEN

defaults = json.load(open('default_configs.json', 'r', encoding='utf-8'))

# 查询页面现有组件
page = bi_utils.query_page(PAGE_ID)
tmpl = page.get('template', [])

# 生成关联组件的 ID（新增场景）
comp1_id = bi_utils._gen_uuid()
comp2_id = bi_utils._gen_uuid()

# --- 创建关联组件（折线图、柱形图等） ---
# 使用 bi_utils.add_component 的逻辑构建 comp dict，此处略
# 关键：记录每个组件的 i（UUID）

# --- 创建选项卡 ---
tab_cfg = json.loads(json.dumps(defaults.get('JSelectRadio', {})))
tab_w, tab_h = tab_cfg.pop('w', 300), tab_cfg.pop('h', 40)
tab_cfg['chartData'] = json.dumps([
    {"label": "选项一", "value": "1"},
    {"label": "选项二", "value": "2"}
], ensure_ascii=False)
tab_cfg['background'] = '#FFFFFF00'
tab_cfg['borderColor'] = '#FFFFFF00'
# ⚠️ 核心：组件可见性配置
tab_cfg['compShowConfig'] = [
    {"selectVal": "1", "compVals": [comp1_id]},
    {"selectVal": "2", "compVals": [comp2_id]}
]
tab_config_str = json.dumps(tab_cfg, ensure_ascii=False)
tab_cfg['size'] = {'width': tab_w, 'height': tab_h}  # ⚠️ 必须设置，否则组件不渲染
tab_cfg['chart'] = {'subclass': 'JSelectRadio', 'category': 'other'}
tab_cfg.setdefault('turnConfig', {})
tab_cfg.setdefault('linkageConfig', [])
tab_config_str = json.dumps(tab_cfg, ensure_ascii=False)
tab_comp = {
    "i": bi_utils._gen_uuid(),
    "component": "JSelectRadio",
    "componentName": "选项卡",
    "x": 100, "y": 250, "w": tab_w, "h": tab_h,
    "dataType": 0,
    "config": tab_config_str,
    "dataMapping": {},
    "size": {"width": tab_w, "height": tab_h},  # ⚠️ 顶层也要设置
    "chart": {"subclass": "JSelectRadio", "category": "other"},
    "turnConfig": {}, "linkageConfig": []
}

# 插入组件（选项卡置顶，关联组件紧随其后）
tmpl.insert(0, tab_comp)
tmpl.insert(1, comp1_dict)  # 折线图
tmpl.insert(2, comp2_dict)  # 柱形图

bi_utils._page_components[PAGE_ID] = tmpl
bi_utils.save_page(PAGE_ID)
```

**为已有组件添加选项卡关联（无需新建图表）：**

```python
# 查询页面，找到已有组件的 ID
page = bi_utils.query_page(PAGE_ID)
tmpl = page.get('template', [])

# 按名称查找已有组件 ID
line_id = next(c['i'] for c in tmpl if c.get('componentName') == '基础折线图')
bar_id = next(c['i'] for c in tmpl if c.get('componentName') == '基础柱形图')

# 创建选项卡并关联已有组件
tab_cfg = json.loads(json.dumps(defaults.get('JSelectRadio', {})))
tab_cfg.pop('w', None); tab_cfg.pop('h', None)
tab_cfg['chartData'] = json.dumps([
    {"label": "折线图", "value": "1"},
    {"label": "柱形图", "value": "2"}
], ensure_ascii=False)
tab_cfg['compShowConfig'] = [
    {"selectVal": "1", "compVals": [line_id]},
    {"selectVal": "2", "compVals": [bar_id]}
]
# ... 构建 tab_comp 并 insert(0, tab_comp)
```

**⚠️ 易错点：**

| 错误 | 正确做法 |
|------|---------|
| `compShowConfig` 放在 `option` 内 | `compShowConfig` 与 `option` 同级，在 `config` 顶层 |
| `compVals` 用组件名称 | `compVals` 必须用组件 `i` 字段（UUID），不是 `componentName` |
| 关联组件和选项卡位置不重叠 | 被关联组件应放在**相同坐标区域**（x/y/w/h 一致），由选项卡控制显隐切换 |
| 忘记设置 config 为 JSON 字符串 | `config` 必须是 `json.dumps()` 后的字符串 |
| `size` 字段为空 `{}` | config 内部和顶层都必须设置 `size: {width, height}`，否则组件初始不渲染（需手动拖拽才显示）。同理 `chart`/`turnConfig`/`linkageConfig` 也要设置 |

### Step 2: 展示设计摘要并确认

**可跳过确认直接执行的情况：**
- 用户说「直接生成」「不用确认」
- 模板名称与需求精确匹配
- 同一会话中已确认过类似方案

### 快捷操作：comp_ops.py（增删改查）

> **⚠️ 添加/编辑/删除组件必须使用 comp_ops.py，严禁直接调用 bi_utils.add_xxx() + save_page()。**
> 原因：bi_utils.add_component() 内部将 `_page_components[page_id]` 初始化为空列表，save_page 时会用空列表覆盖页面已有的全部组件，造成不可恢复的数据丢失。comp_ops.py 会先加载已有模板再操作，安全无损。

**脚本位置**：`C:\Users\25067\.claude\skills\jimubi-bigscreen\references\scripts\comp_ops.py`

**使用前准备（comp_ops.py add 专用，需额外复制 default_configs.json）：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/comp_ops.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/default_configs.json" .
# 执行完后清理
rm comp_ops.py bi_utils.py default_configs.json
```

**核心命令：**
```bash
# 查看组件
py comp_ops.py list $API_BASE $TOKEN $PAGE_ID

# 删除组件
py comp_ops.py delete $API_BASE $TOKEN $PAGE_ID --name "组件名"

# 编辑组件属性（单属性）
py comp_ops.py edit $API_BASE $TOKEN $PAGE_ID --name "组件名" --set "option.title.text=新标题"

# 编辑组件属性（多属性：每个属性一个 --set）
py comp_ops.py edit $API_BASE $TOKEN $PAGE_ID --name "胶囊图" --set "option.showValue=true" --set "option.unit=333"

# 添加组件（静态数据）
py comp_ops.py add $API_BASE $TOKEN $PAGE_ID --comp "JBar" --title "柱形图" --x 50 --y 500 --w 450 --h 300

# 一键：创建SQL数据集 + 字典翻译 + 添加图表
py comp_ops.py add $API_BASE $TOKEN $PAGE_ID --comp "JPie" --title "男女比例" --x 735 --y 365 --w 450 --h 350 --create-sql "SELECT sex as name, COUNT(*) AS value FROM demo WHERE sex IS NOT NULL GROUP BY sex" --ds-name "男女比例统计" --fields "name:String,value:String" --dict "name=sex"

# 移动/缩放组件
py comp_ops.py move $API_BASE $TOKEN $PAGE_ID --name "组件名" --x 100 --y 200
```

**四种数据模式：**

| 模式 | 参数 | 说明 |
|------|------|------|
| 静态数据（默认） | 无额外参数 | 从 `default_configs.json` 加载默认配置 |
| 绑定已有数据集 | `--dataset-name "名称"` | 内置自动查询数据集、设 dataType=2，**无需单独调 dataset_ops.py 查询** |
| 一键创建SQL+绑定 | `--create-sql "SQL"` | 创建数据集+绑定+字典，支持 `--dict`、`--fields` |
| 带查询参数的SQL | `--create-sql` + `--sql-params` | `comp_ops.py add --sql-file sql.txt --sql-params "age:年龄::"` 或自定义 Python 脚本 |

**⚠️ 带 FreeMarker 动态参数的 SQL 必须用 `--sql-file`，禁止通过 bash 命令行传递。** 原因：`${age}` 会被 shell 解释为变量（值为空），`<#if>` 中的 `>` 会被解释为重定向，导致 SQL 被截断或参数丢失。

**动态SQL查询参数完整示例（强制规范）：**

```bash
# Step 1: 将含 FreeMarker 语法的 SQL 写入文件
cat > sql.txt << 'SQLEOF'
SELECT sex as name, COUNT(*) AS value FROM demo WHERE sex IS NOT NULL
<#if isNotEmpty(age)>
  AND age = '${age}'
</#if>
GROUP BY sex
SQLEOF

# Step 2: 用 --sql-file + --sql-params 创建
py comp_ops.py add $API_BASE $TOKEN $PAGE_ID \
  --comp "JPie" --title "男女比例" --x 735 --y 365 --w 450 --h 350 \
  --sql-file sql.txt --ds-name "男女比例统计" \
  --fields "name:String,value:String" --dict "name=sex" \
  --sql-params "age:年龄::"

# Step 3: 清理
rm sql.txt
```

**FreeMarker 动态条件语法规则（强制）：**

| 规则 | 正确写法 | 错误写法 |
|------|---------|---------|
| 参数判空 | `<#if isNotEmpty(age)>` | ~~`<#if age?? && age?length gt 0>`~~ |
| 参数占位 | `'${age}'` | ~~`#{age}`~~（`#{}` 是系统变量专用） |
| 条件结束 | `</#if>` | - |
| 系统变量 | `#{sys_user_code}` | ~~`${sys_user_code}`~~（`${}` 和 `#{}` 不可混用） |

**`--sql-params` 格式**：`paramName:paramTxt:defaultValue:dictCode`（后三项可省略，多个逗号分隔）

| 示例 | 说明 |
|------|------|
| `"age:年龄::"` | 年龄参数，无默认值，无字典 |
| `"sex:性别:1:sex"` | 性别参数，默认值 1，字典编码 sex |
| `"age:年龄::,sex:性别:1:sex"` | 多参数逗号分隔 |

**SQL 含 `!=` 等特殊字符时**：同样禁止通过 bash 传递，必须用 `--sql-file` 或写 Python 脚本在内部定义 SQL。

**自定义脚本添加图表的强制规则：**
1. **图表 config 必须从 `default_configs.json` 深拷贝**：`json.loads(json.dumps(defaults['JPie']))`，再覆盖动态数据字段。禁止手写 option/series 配置
2. **字典翻译用 jimu_dict**：`/jmreport/dict/*` API，不是 `/sys/dict/*`（系统字典需签名且表不同）
3. **dictOptions 从 `getAllChartData` 获取**：创建数据集后调 `getAllChartData`，将返回的 `dictOptions` 写入组件 config，禁止手动构建
4. **datasetItemList 中绑定 dictCode**：如 `{'fieldName': 'name', ..., 'dictCode': 'sex'}` 实现字段级字典翻译

### 全部预置脚本一览

脚本目录：`C:\Users\25067\.claude\skills\jimubi-bigscreen\references\scripts\`

| 脚本 | 功能 | 常用命令 |
|------|------|---------|
| `yapi_ops.py` | YApi Mock 接口管理（固定 claude AI 项目） | `create-mock`（`--template single/multi/pie/gauge/table/bar_multi` 或 `--body JSON`），`list`，`delete`，`update`。Mock URL 自动含 basepath `/claude` |
| `comp_ops.py` | 组件增删改查 | `list`, `delete`, `edit`, `add`, `move` |
| `page_ops.py` | 页面配置 | `info`, `set-bg`, `set-bgimg`, `set-theme`, `watermark`, `rename`, `delete`。**rename 参数格式：** `py page_ops.py rename API_BASE TOKEN PAGE_ID --name "新名称"`（`--name` 是命名参数，不是位置参数）。**delete 用法：** `py page_ops.py delete API_BASE TOKEN PAGE_ID` |
| `dataset_ops.py` | 数据集管理 | `list`, `create-sql`, `create-api`, `edit`, `test`, `delete`, `bind` |
| `template_ops.py` | 模板操作 | `list`, `preview`, `search`, `copy` |
| `linkage_ops.py` | 联动/钻取 | `show`, `add-linkage`, `remove-linkage`, `add-drill` |
| `link_ops.py` | 外部链接跳转 | `show`, `set`, `remove` |
| `map_ops.py` | 地图数据 | `list`, `check`, `upload`, `add-map` |
| `style_ops.py` | 批量样式 | `show-colors`, `set-title-color`, `set-palette`, `batch-edit` |
| `backup_ops.py` | 备份恢复 | `export`, `import`, `clone`, `diff` |
| `datasource_ops.py` | 数据源管理（JDBC + NoSQL） | `list`, `detail`, `create`, `edit`, `test`, `delete`, `parse-sql`。**create 参数：** `--db`（非 --db-name）、`--user`（非 --username）；**test 支持 `--id` 或 `--name`**（按名称自动查找后测试）。**edit 参数：** `--name 数据源名称`（或 `--id`）定位，`--add-jdbc-param "key=value"` 追加/替换 JDBC 参数（如 `trustServerCertificate=true`），`--set-url` 替换完整 URL，`--user`/`--password` 修改凭据。**SQLSERVER create 已自动包含 `trustServerCertificate=true`**，旧数据源用 `edit --add-jdbc-param` 修复。**支持 NoSQL：** `--db-type mongodb/redis/es`，自动生成 `host:port/db` 格式的 dbUrl（不带协议前缀），dbDriver 自动置空。**NoSQL 数据集 SQL 语法：** MongoDB 表名加 `mongo.` 前缀（`select * from mongo.表名`），ES 加 `es.` 前缀。**API 接口（需签名）：** `GET /drag/onlDragDataSource/getOptions`（list，返回 `[{value:id,label:name,text:name}]`）、`GET /drag/onlDragDataSource/queryById`、`POST /drag/onlDragDataSource/add`、`POST /drag/onlDragDataSource/edit`（传完整对象）、`POST /drag/onlDragDataSource/testConnection`、`DELETE /drag/onlDragDataSource/delete` |
| `group_ops.py` | 组合管理 | `list`, `create`, `ungroup` |
| `dict_ops.py` | 字典管理 | `list`, `create`, `items`, `bind` |
| `files_ops.py` | 多文件数据集（FILES）| `create-bind`（建数据集+上传+推断SQL+加图表），`upload`，`list-tables`，`add-chart` |
| `proc_ops.py` | 存储过程管理 | `create`, `list`, `drop`, `bindcomp`（一键：创建存储过程+数据集+组件）。**前置条件：`py -m pip install pymysql`**，通过 pymysql 直连数据库执行 DDL |
| `files_ops.py` | 多文件数据集（FILES）管理 | `create-bind`（一键：建数据集+上传文件+自动推断JOIN SQL+添加图表），`upload`（仅上传），`list-tables`（查表名），`add-chart`（单独绑图表）。**见下方「快捷操作：files_ops.py」章节**。`create-bind` JOIN 模式必须传 `--group-by <列名1,列名2> --join-on <关联列> --agg <聚合列>`；不传则脚本走 fallback 分支报 `UnboundLocalError`。**列名未知时先问用户，不要盲目执行** |

**通用使用流程：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/脚本名.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" .
# 执行命令...
rm 脚本名.py bi_utils.py
```

### Step 3: 调用 API 创建大屏

**工具库**：`C:\Users\25067\.claude\skills\jimubi-bigscreen\references\bi_utils.py`（复制到当前工作目录使用）

**执行步骤（最少 2 轮工具调用）：**

**方式一：template_ops.py copy（模板场景，最优）：**
```
轮次1: cp template_ops.py + bi_utils.py
轮次2: 执行 copy --replace && echo URL | clip.exe && rm
```

**方式二：自定义脚本（复杂逻辑）：**
```

---

## 备选方式：从模板复制创建大屏

> **注意：模板复制方式仅作为备选。** 模板 JSON 中的 config 结构复杂且样式耦合严重，批量文本替换容易破坏配置完整性，生成效果往往不理想。仅在需要精确还原某个已有模板的视觉布局时才考虑使用。

### 模板目录

`references/templates/bigScreen/` 下有 40 个大屏模板 JSON 可供选择。

### 模板复制完整流程

```python
import sys, json
sys.path.insert(0, '.')
from bi_utils import *
import bi_utils

init_api('http://api3.boot.jeecg.com', 'your-token')

# 1. 读取模板 JSON
tpl_path = r'C:/Users/zhang/.claude/skills/jimubi-bigscreen/references/templates/bigScreen/集团综合数据大屏_1151069555267260416.json'
with open(tpl_path, 'r', encoding='utf-8') as f:
    tpl_data = json.load(f)
template_components = tpl_data.get('template', [])

# 2. 建立旧 ID → 新 ID 映射（关键！）
id_mapping = {}
for comp in template_components:
    old_i = comp['i']
    id_mapping[old_i] = bi_utils._gen_uuid()

# 3. 更新组件 ID 和清理
for comp in template_components:
    comp['i'] = id_mapping[comp['i']]
    comp.pop('pageCompId', None)
    # config 字符串转 dict
    config = comp.get('config', {})
    if isinstance(config, str):
        try: config = json.loads(config)
        except: config = {}
    comp['config'] = config

# 4. 更新 JTabToggle 的 compVals 引用（否则页签切换不工作）
for comp in template_components:
    if comp['component'] == 'JTabToggle':
        for item in comp['config'].get('option', {}).get('items', []):
            item['compVals'] = [id_mapping.get(v, v) for v in item.get('compVals', [])]

# 5. 更新 JGroup 内部 props.elements 中的 ID 引用
for comp in template_components:
    if comp['component'] == 'JGroup':
        props = comp.get('props', {})
        elements = props.get('elements', [])
        if elements:
            el_str = json.dumps(elements, ensure_ascii=False)
            for old_id, new_id in id_mapping.items():
                el_str = el_str.replace(old_id, new_id)
            props['elements'] = json.loads(el_str)

# 6. 创建页面并保存
page_id = create_page('我的大屏', style='bigScreen', theme='dark',
                       background_image='/img/bg/bg4.png')
bi_utils._page_components[page_id] = template_components
save_page(page_id)
```

**默认配置与数据加载规则：**

| 数据场景 | 是否读 data.ts |
|---------|---------------|
| 动态数据（SQL/API, dataType=2） | 不需要，chartData='[]'，option 在脚本中构建 |
| 静态数据 + 用户未指定数据 | 需要，从 data.ts 读取 compConfig |
| 静态数据 + 用户指定了数据 | 不需要 |

**数据集「先查后建」规则（强制）：** 指定名称时先查询是否已存在同名数据集，存在则复用。

**Git Bash 注意事项：**
- `python3 xxx.py 2>/dev/null || py xxx.py`（Windows 兼容）
- `/` 开头路径会被转换，脚本内部赋值不受影响
- `!=` 等特殊字符不要通过 bash 传递

## 大屏标题规则

- `option.card.title` 必须为空字符串（避免双重标题）
- 页面主标题用 `add_text()`，fontSize≥40，fontWeight='bold'，letterSpacing=5

## 常用组件配置路径速查（内联）

> 以下组件的 option 路径已内联，修改时**直接使用，无需读取 `bi-comp-option-config.md`**。

### JStatsSummary（统计概览）

| 说明 | 配置路径 | 示例值 |
|------|---------|--------|
| 卡片最小宽度 | `option.card.minWidth` | 250 |
| 卡片圆角 | `option.card.borderRadius` | 16 |
| 卡片边框宽度 | `option.card.borderWidth` | 1 |
| 卡片边框颜色 | `option.card.borderColor` | #0f66ff59 |
| 卡片阴影 | `option.card.shadow` | 0 16px 48px #0b76ff59 |
| 卡片模糊度 | `option.card.blur` | 24 |
| 卡片内边距(垂直) | `option.card.padding.vertical` | 24 |
| 卡片内边距(水平) | `option.card.padding.horizontal` | 24 |
| 卡片填充类型 | `option.card.fill.type` | none/color/gradient/image |
| 卡片填充颜色 | `option.card.fill.color` | #0b2b63 |
| 卡片填充渐变启用 | `option.card.fill.gradient.enabled` | true/false |
| 卡片填充渐变起始色 | `option.card.fill.gradient.startColor` | #05336a |
| 卡片填充渐变结束色 | `option.card.fill.gradient.endColor` | #0bb2ff |
| 卡片填充图片 | `option.card.fill.image.url` | /img/xxx.png |
| 外层间距 | `option.layout.gap` | 16 |
| 外层内边距 | `option.layout.padding.top/right/bottom/left` | 16 |
| 外层排列方式 | `option.layout.justify` | space-between |
| 外层圆角 | `option.layout.borderRadius` | 0 |
| 外层边框宽度 | `option.layout.borderWidth` | 0 |
| 外层填充类型 | `option.layout.fill.type` | none/color/gradient/image |
| 外层填充颜色 | `option.layout.fill.color` | #0b2b63 |
| 数值字号 | `option.sections.top.value.fontSize` | 34 |
| 数值字重 | `option.sections.top.value.fontWeight` | 600 |
| 数值颜色 | `option.sections.top.value.fontColor` | #d8f1ff |
| 单位字号 | `option.sections.top.value.unit.fontSize` | 18 |
| 标签字号 | `option.sections.bottom.label.fontSize` | 14 |
| 标签颜色 | `option.sections.bottom.label.fontColor` | #9ed3ff |

### JCapsuleChart（胶囊图）

| 说明 | 配置路径 | 示例值 |
|------|---------|--------|
| 显示数值 | `option.showValue` | true/false |
| X轴名称 | `option.unit` | 个 |

### JGauge（仪表盘）

| 说明 | 配置路径 |
|------|---------|
| 刻度值显隐 | `option.series[0].axisLabel.show` |
| 刻度值颜色 | `option.series[0].axisLabel.color` |
| 刻度线显隐 | `option.series[0].axisTick.show` |
| 分割线显隐 | `option.series[0].splitLine.show` |
| 分割线颜色 | `option.series[0].splitLine.lineStyle.color` |
| 指标字号 | `option.series[0].detail.fontSize` |

### JProgress（进度条-ECharts）

| 说明 | 配置路径 |
|------|---------|
| 显示标题 | `option.yAxis.axisLabel.show` |
| 标题字体颜色 | `option.yAxis.axisLabel.color` |

### JColorBlock（色块指标卡）

| 说明 | 配置路径 |
|------|---------|
| 行数 | `option.lineNum` |
| 边距 | `option.padding` |

### JScrollBoard（轮播表）

| 说明 | 配置路径 |
|------|---------|
| 悬浮暂停 | `option.hoverPause` |
| 等待时间 | `option.waitTime` |

## 图层顺序机制

**核心：`template` 数组索引决定 z-index，不是 orderNum。** 索引 0 = 最顶层。

### 新增组件必须置顶（强制）

> **新添加的组件必须插入到 `template` 数组的索引 0 位置（即最顶层），确保不会被已有组件遮挡。**
> `bi_utils.add_component()` 已使用 `insert(0, comp)` 实现自动置顶。自定义脚本操作模板时也必须用 `insert(0, comp)` 而非 `append(comp)`。

```python
# 置顶
element = tmpl.pop(target_idx)
tmpl.insert(0, element)
# 保存
bi_utils._page_components[PAGE_ID] = tmpl
save_page(PAGE_ID)
```

## 可用快捷函数（bi_utils.py）

**页面管理：** `create_page`, `query_page`, `list_pages`, `save_page`, `delete_page`, `recover_page`, `copy_page`

**添加组件：** `add_number`, `add_chart`(JBar/JLine/JPie/JRing/JRose/JFunnel/JRadar/JHorizontalBar/JSmoothLine/JStackBar/JMixLineBar), `add_table`, `add_scroll_table`, `add_ranking`, `add_text`, `add_image`, `add_gauge`, `add_liquid`, `add_countdown`, `add_border`, `add_decoration`, `add_current_time`, `add_word_cloud`, `add_color_block`, `add_progress`, `add_total_progress`, `add_component`

## Step 4: 输出结果

**必须将预览地址作为单独一行返回，并用 clip.exe 复制到剪贴板。**

**⚠️ 每次任务完成后必须输出总耗时（强制）：**

- **脚本中**：开头记录 `import time; t0 = time.time()`，末尾输出 `print(f'耗时: {time.time()-t0:.1f}s')`
- **多轮调用/纯API操作**：在最终回复文字末尾补充一行 `耗时：约 Xs`
- 可利用 API 响应中的 `timestamp` 字段估算（首尾两次响应时间戳之差）
- **禁止输出每个步骤的耗时**，只输出整个任务从开始到结束的总耗时

```
## 大屏创建成功

- 页面ID：{id}
- 页面名称：{name}
- 组件数量：{count} 个

预览地址：
{API_BASE}/drag/share/view/{id}?token={TOKEN}&tenantId=2
```

```bash
echo -n "{完整URL}" | clip.exe
```

**⚠️ 写了 Java 接口时，脚本末尾必须额外输出（强制）：**

```python
# 脚本末尾固定追加此段输出，让用户知道后续操作
print("\n" + "="*60)
print("大屏组件已生成完成！")
print("="*60)
print("\n【API 接口地址】（需重启后端后生效）：")
print(f"  {API_BASE}/drag/mock/xxxFlow")           # 根据实际接口路径替换
print(f"  {API_BASE}/drag/mock/xxxFlowMulti")       # 如有多个接口逐行列出
print("\n【重要提示】请重启 Spring Boot 后端服务！")
print("  重启后 API 数据集将自动拉取接口数据，图表即可显示。")
print("\n【大屏预览地址】")
print(f"  {API_BASE}/drag/share/view/{PAGE_ID}?token={TOKEN}&tenantId=2")
print("="*60)
```

## bi_utils 使用规则（强制）

### 初始化方式

```python
# 正确：直接赋值模块级全局变量
bi_utils.API_BASE = 'http://...'
bi_utils.TOKEN = '...'

# 错误：没有 init() 方法
# bi_utils.init(API_BASE, TOKEN)  # ← AttributeError
```

### 页面数据与组件字段映射（query_page 返回值）

| 正确字段 | 常见误猜 | 说明 |
|---------|---------|------|
| `page['template']` | ~~`page['pageTemplate']`~~ | 组件列表，**已经是 list**，无需 `json.loads` |
| `comp['i']` | ~~`comp['id']`~~ | 组件唯一标识（UUID） |
| `comp['componentName']` | ~~`comp['label']`~~, ~~`comp['name']`~~ | 组件显示名称（中文） |
| `comp['component']` | - | 组件类型（JBar, JText 等） |
| `comp['pageCompId']` | - | 后端数据库 ID |
| `comp['isLock']` | - | 锁定状态（true/false） |

### 自定义脚本操作模板的正确模式

```python
import bi_utils
bi_utils.API_BASE = '...'
bi_utils.TOKEN = '...'
PAGE_ID = '...'

page = bi_utils.query_page(PAGE_ID)
tmpl = page.get('template', [])  # 已经是 list，不需要 json.loads

# 按组件名查找（字段是 componentName，不是 label/name）
target_idx = next(i for i, c in enumerate(tmpl) if c.get('componentName') == '目标名称')

# 修改后保存
bi_utils._page_components[PAGE_ID] = tmpl
bi_utils.save_page(PAGE_ID)
```

### Windows Python 命令

- 用 `py` 不是 `python`（Git Bash 下 `python` 找不到）
- **必须加 `PYTHONIOENCODING=utf-8` 前缀**（Windows 默认 GBK 编码，脚本中含 emoji（✅⚠️等）或中文输出时报 `UnicodeEncodeError: 'gbk' codec can't encode character`。**所有 `py script.py` 调用必须写成 `PYTHONIOENCODING=utf-8 py script.py`**，且脚本内部 `print` 禁止使用 emoji 字符）

### 快捷操作：linkage_ops.py（组件联动/钻取）

> **组件联动 = 点击源组件，将参数传递给目标组件的数据集查询参数，目标组件自动刷新数据。**

**脚本位置**：`C:\Users\25067\.claude\skills\jimubi-bigscreen\references\scripts\linkage_ops.py`

**使用前准备：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/linkage_ops.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" .
# 执行完后清理
rm linkage_ops.py bi_utils.py
```

**核心命令：**
```bash
# 查看页面所有联动配置
py linkage_ops.py show $API_BASE $TOKEN $PAGE_ID

# 添加联动（--mapping 格式：src=tgt，多个逗号分隔）
py linkage_ops.py add-linkage $API_BASE $TOKEN $PAGE_ID --source "源组件名" --target "目标组件名" --mapping "value=age"
py linkage_ops.py add-linkage $API_BASE $TOKEN $PAGE_ID --source "柱形图" --target "饼图" --mapping "name=name,value=keyword"

# 删除联动
py linkage_ops.py remove-linkage $API_BASE $TOKEN $PAGE_ID --source "源组件名" --target "目标组件名"

# 添加钻取
py linkage_ops.py add-drill $API_BASE $TOKEN $PAGE_ID --source "源组件名" --target "目标组件名" --mapping "name=category"
```

**⚠️ 易错点（强制记忆）：**

| 错误写法 | 正确写法 | 说明 |
|---------|---------|------|
| `--param "value:age"` | `--mapping "value=age"` | 参数名是 `--mapping` 不是 `--param` |
| `--mapping "value:age"` | `--mapping "value=age"` | 映射用 `=` 分隔，不是 `:` |
| `--mapping "a=b c=d"` | `--mapping "a=b,c=d"` | 多个映射用逗号分隔 |

**联动前提：** 目标组件必须已绑定数据集，且数据集 SQL 中有对应的查询参数（如 `${age}`）。

### 快捷操作：link_ops.py（外部链接跳转）

> **组件外部链接 = 点击图表跳转到外部 URL，并将点击参数带到链接地址上。**

**脚本位置**：`C:\Users\25067\.claude\skills\jimubi-bigscreen\references\scripts\link_ops.py`

**使用前准备：**
```bash
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/scripts/link_ops.py" .
cp "C:/Users/25067/.claude/skills/jimubi-bigscreen/references/bi_utils.py" .
# 执行完后清理
rm link_ops.py bi_utils.py
```

**核心命令：**
```bash
# 查看页面所有外部链接配置
py link_ops.py show $API_BASE $TOKEN $PAGE_ID

# 设置外部链接（按名称/类型/ID 三选一定位组件）
py link_ops.py set $API_BASE $TOKEN $PAGE_ID --name "饼图名" --url "https://www.baidu.com/s?wd=\${name}&value=\${value}"
py link_ops.py set $API_BASE $TOKEN $PAGE_ID --type "JPie" --url "https://example.com/detail?category=\${name}"
py link_ops.py set $API_BASE $TOKEN $PAGE_ID --id "538804ec..." --url "https://www.baidu.com/s?wd=\${name}" --target "_self"

# 删除外部链接
py link_ops.py remove $API_BASE $TOKEN $PAGE_ID --name "饼图名"
```

**URL 参数占位符（来自 ECharts 点击事件 params）：**

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `${name}` | 维度名称 | 饼图扇区名、柱子 x 轴标签 |
| `${value}` | 数值 | 饼图扇区值、柱子高度 |
| `${type}` | 系列名称 | 多系列图表的系列标识 |

**打开方式（--target）：** `_blank`（新窗口，默认）、`_self`（当前窗口）

**技术原理：** 组件 config 中 `linkType='url'` + `turnConfig={url:'...', type:'_blank'}`。点击时前端从 ECharts params 中提取 name/value/type，替换 URL 中的 `${...}` 占位符后执行跳转。参考文档：https://help.jimureport.com/biScreen/base/interactive/jumpto

### 快捷操作：自定义JS脚本（config.jsConfig）

> **自定义JS脚本 = 点击组件时执行用户编写的 JavaScript 代码，可实现自定义跳转、弹窗、数据处理等业务逻辑。**
> 参考文档：https://help.jimureport.com/biScreen/base/interactive/customJS

**存储字段**：`config.jsConfig`（字符串，存储 JS 代码）

**执行原理（源码级）：**

| 组件类型 | 源码位置 | 执行方式 | 可用参数 |
|---------|---------|---------|---------|
| ECharts 图表（JBar/JPie/JLine 等） | `packages/hooks/charts/useEChartsNew.ts` | `new Function('params', 'option', fixedScript)` | `params`（ECharts 点击事件对象）、`option`（图表配置） |
| 非 ECharts 组件（JStatsSummary/JColorBlock 等） | `packages/hooks/common/useCommon.ts` | `new Function('params', config.jsConfig)` | `params`（当前点击的数据项） |

**执行顺序**：`jsHandler(params)` → 返回 `true` 则继续 → 外部链接跳转 → 联动配置 → 钻取配置

**返回值规则**：
- `return true`（或无返回）→ 继续执行系统内置后续方法（联动、跳转、钻取）
- `return false` → 阻断所有后续方法

**ECharts 组件的自动修复**（`useEChartsNew.ts`）：
- `)return` → `); return`（修复缺少分号导致语法错误）
- `；`（中文分号）→ `;`

**脚本参数 `params` 常用属性（ECharts 图表）：**

| 属性 | 含义 | 示例 |
|------|------|------|
| `params.name` | 维度名称 | 柱子 x 轴标签、饼图扇区名 |
| `params.value` | 数值 | 柱子高度、饼图扇区值 |
| `params.data` | 原始数据对象 | `{name:'北京', value:100}` |
| `params.dataIndex` | 数据索引 | 0, 1, 2... |
| `params.seriesName` | 系列名称 | 多系列图表的系列标识 |
| `params.seriesIndex` | 系列索引 | 0, 1, 2... |

> 更多属性参考 [ECharts 事件文档](https://echarts.apache.org/zh/api.html#events.%E9%BC%A0%E6%A0%87%E4%BA%8B%E4%BB%B6)

**设置方式（三种，按复杂度选择）：**

```bash
# 方式1：link_ops.py set-js（推荐，支持多行脚本）
py link_ops.py set-js $API_BASE $TOKEN $PAGE_ID --name "基础柱形图" --js 'window.open("http://jeecg.com");return false;'

# 方式1b：从文件读取复杂脚本
py link_ops.py set-js $API_BASE $TOKEN $PAGE_ID --type "JBar" --js-file script.js

# 方式2：comp_ops.py edit（简单单行脚本）
py comp_ops.py edit $API_BASE $TOKEN $PAGE_ID --name "基础柱形图" --set "jsConfig=window.open(\"http://jeecg.com\");return false;"

# 方式3：自定义 Python 脚本（需要精确控制换行符等）
# cfg['jsConfig'] = 'window.open("http://jeecg.com");\nreturn false;'
```

```bash
# 查看页面所有自定义JS脚本配置
py link_ops.py show $API_BASE $TOKEN $PAGE_ID    # show 命令同时展示外部链接和JS脚本

# 删除自定义JS脚本
py link_ops.py remove-js $API_BASE $TOKEN $PAGE_ID --name "基础柱形图"
```

**常用脚本示例：**

```javascript
// 1. 跳转到外部网站（带点击参数）
window.open("https://example.com/detail?name=" + params.name + "&value=" + params.value);
return false;

// 2. 弹窗显示点击数据
alert("你点击了: " + params.name + ", 值: " + params.value);
return true;  // 继续执行联动等后续逻辑

// 3. 条件跳转（根据点击值跳不同页面）
if (params.value > 100) {
  window.open("https://example.com/high?name=" + params.name);
} else {
  window.open("https://example.com/low?name=" + params.name);
}
return false;

// 4. 自定义联动逻辑（修改其他组件）
console.log("点击数据:", params.data);
return true;  // 继续走系统联动
```

**jsConfig 与 linkType/turnConfig 的关系：**
- `jsConfig` 和 `linkType`/`turnConfig`（外部链接）是**独立配置**，可同时存在
- 点击时先执行 `jsConfig`，返回 `true` 才继续执行外部链接/联动/钻取
- 返回 `false` 则阻断所有后续交互

**UI 位置**：设计器 → 选中组件 → 右侧面板 → 事件配置 → 自定义JS脚本（CodeEditor 编辑器）

## 核心踩坑速查

| 问题 | 说明 |
|------|------|
| **⚠️ 锁定组件字段是 `disabled`，不是 `isLock`** | 右键"锁定"操作切换的是组件顶层的 `disabled` 字段（`disabled=True`），`isLock` 是无效字段。锁定后组件不可拖拽/编辑，右键只显示"解锁"。正确做法：`comp['disabled'] = True; comp['selected'] = False`（在顶层设置，不是 `config` 内部） |
| **⚠️ 解锁组件禁止用 `comp_ops.py edit --set "disabled=false"`** | `comp_ops.py edit --set` 会将字段写入组件的 `config` JSON 字符串内部，而锁定/解锁的 `disabled` 字段在组件**顶层**（与 `x/y/w/h/componentName` 同级）。用 `--set` 解锁后前端仍显示锁定状态。**必须用自定义脚本**：`target['disabled'] = False; target['selected'] = False`（直接操作 template 数组中组件的顶层字段） |
| **⚠️ 严禁直接 bi_utils.add_xxx + save_page** | `add_component` 初始化空列表，save_page 会覆盖已有组件造成数据丢失。必须用 `comp_ops.py add` |
| `POST /drag/page/edit` 乐观锁 | 必须传 `updateCount` |
| **chartData 必须是 JSON 字符串** | `json.dumps(...)` 后的字符串 |
| **dataMapping 的 filed 拼写** | `filed` 不是 `field`（少一个 d） |
| **严禁 `rgba(0,0,0,0)`** | 用 `#FFFFFF00` |
| **background 字段位置** | `config` 顶层（与 `option` 同级） |
| **图表标题去重** | `card.title=''`，只用 `option.title.text` |
| **图层顺序** | 数组索引 0=最顶层，`orderNum` 不控制 z-index |
| **⚠️ 新增组件必须置顶** | `bi_utils.add_component()` 已用 `insert(0, comp)` 自动置顶；自定义脚本也必须 `insert(0,...)` 而非 `append()`，否则新组件在最底层被遮挡 |
| **新增组件不显示** | config 不完整或被遮挡，`insert(0,...)` 到数组开头 |
| **组件 ID 字段是 `i` 不是 `id`** | `template` 数组中每个组件的唯一标识字段名为 `i` |
| **组件名称是 `componentName`** | 不是 `label` 也不是 `name`，中文名在 `componentName` |
| **模板数据在 `template` 字段** | `query_page` 返回的组件列表在 `template` 中，已是 list；`pageTemplate` 是空字符串 |
| **bi_utils 无 init() 方法** | 直接赋值 `bi_utils.API_BASE` 和 `bi_utils.TOKEN` |
| **Windows 用 `py` 不是 `python`** | Git Bash 下 `python` 命令不存在 |
| **⚠️ Windows 执行 py 脚本必须加 `PYTHONIOENCODING=utf-8`** | Windows 默认 GBK 编码，脚本 print 含 emoji（✅⚠️）或中文时报 `UnicodeEncodeError: 'gbk' codec can't encode character`（已观测：singleFile 脚本首次执行失败，修复编码后才成功）。**强制规范**：所有 py 命令统一写成 `PYTHONIOENCODING=utf-8 py script.py`；脚本内部 `print` 禁止使用 emoji，改用纯文本（`[OK]` 代替 ✅，`[WARN]` 代替 ⚠️）|
| **存储过程：JimuReport API 无法执行 DDL** | `getAllChartData` 用 `executeQuery()` 只支持 SELECT/CALL，CREATE PROCEDURE 会报错。必须通过 pymysql 直连数据库创建存储过程 |
| **⚠️ FreeMarker 判空必须用 `isNotEmpty()`** | 正确：`<#if isNotEmpty(age)>`。错误：~~`<#if age?? && age?length gt 0>`~~（JimuReport 不支持标准 FreeMarker 语法，条件不生效） |
| **⚠️ 带 FreeMarker 的 SQL 禁止 bash 命令行传递** | `${age}` 被 shell 解释为空变量，`<#if>` 的 `>` 被解释为重定向。必须用 `--sql-file` 写入文件或 Python 脚本内部定义 SQL |
| **`${}` 和 `#{}` 不可混用** | `${param}` 是查询参数，`#{sys_user_code}` 是系统变量，混用导致解析失败 |
| **存储过程：数据集 SQL 写 CALL 语法** | `querySql` 填 `CALL sp_name()` 或带参 `CALL sp_name('${param}')`，参数用 FreeMarker 语法 |
| **存储过程：自定义脚本绑定组件缺字段** | 直接操作 config 会缺 `dataSetId`/`dataMapping`/`fieldOption`，导致组件显示静态数据。必须用 `comp_ops.py add --dataset-id` 或 `--create-sql "CALL ..."` |
| **⚠️ JWeatherForecast 不能用 comp_ops.py** | 天气预报组件（JWeatherForecast）不在 comp_ops.py 支持范围内，必须用自定义脚本直接操作 template 数组添加。**dataType 必须为 1**（不是 0），option.template 值决定版本样式（11=滚动版, 34=横线版, 21=带背景, 12=好123版, 27=温度计版, 94=列表文字版）。**不要误用 JScrollList 等其他组件替代** |
| **⚠️ WebSocket 数据集不能用 dataset_ops.py / comp_ops.py** | WebSocket 数据集（`dataType='websocket'`）不在预置脚本支持范围内，必须自定义脚本通过 `_request('POST', '/drag/onlDragDatasetHead/add')` 创建。`querySql` 存放 WebSocket 地址（`ws://host:port/context-path/websocket/drag`），无需 `dbSource`。详见 `references/dataset-guide.md`「创建 WebSocket 数据集」章节 |
| **⚠️ WebSocket 推送 socketId = chartId + '_' + md5(token)** | 前端 socketId 生成逻辑（socketId.ts）：`const wsClientId = md5(token); socketId = chartId + '_' + wsClientId`。服务端 `DragWebSocket.sendMessage` 的 key 必须是完整 `socketId`，不能只传 `chartId`，否则找不到对应连接推不出去。Java 推送：`String socketId = chartId + "_" + Md5Util.md5Encode(token, "UTF-8"); webSocket.sendMessage(socketId, obj.toJSONString())` |
| **⚠️ WebSocket 推送消息结构必须用 {chartId, result} 包装** | 正确格式：`obj.put("chartId", chartId); obj.put("result", dataJsonStr); webSocket.sendMessage(socketId, obj.toJSONString())`。data 放在 `result` 字段，不能直接传 JSON 数组字符串。参考 `DragWebSocketController.sendData()` 的实现 |
| **⚠️ WebSocket 组件 config 手动注入必须包含完整字段** | 手动绑定 WebSocket 数据集时 config 需设：`dataType=2, dataSetType='websocket', dataSetIzAgent='0', fieldOption=[...], viewLoading=True, paramOption=[]`。`config` 字段为 JSON 字符串，顶层和 config 内部都需设 `size/chart/turnConfig/linkageConfig` |
| **⚠️ 批量添加组件时手动构造 comp dict** | 必须用 `bi_utils.add_component()` 而非自行构造 `{'component':..., 'config':...}` 并 insert。手动构造会遗漏 `size`/`chart`/`turnConfig`/`linkageConfig` 等必要字段，且 config 必须是 JSON 字符串（不是 dict），否则全部组件显示"暂无数据" |
| **⚠️ option.title 可能是 str 类型** | `default_configs.json` 中部分组件的 `option.title` 是字符串（如 `"基础折线图"`），直接 `option['title']['text'] = x` 会报 `TypeError: 'str' object does not support item assignment`。必须先检查类型：`if isinstance(title, str): title = {'text': title}` |
| **全组件排除装饰类** | "生成全组件"时排除 JWeatherForecast（特殊API组件）、JDragBorder（13种装饰边框）、JDragDecoration（12种装饰条），这些是纯视觉装饰不是业务数据组件 |
| **⚠️ componentName 必须用中文名** | 批量生成时图层名（componentName）必须使用 `menu-hierarchy.md` 中的中文名称（如"基础柱形图"、"饼图"、"统计概览(卡片式)"），禁止直接用 compType（如 JBar、JPie）作为图层名，否则用户在设计器中无法识别组件 |
| **⚠️ 全组件生成禁止为分类生成 JText 标题** | 分类（柱形图/饼图/折线图/...）仅在代码中作注释分组，不要为每个分类生成 JText 组件作为标题。分类标题不是业务组件，会在设计器中产生 20 个多余文本图层，占用额外高度。组件应扁平排列在 all_comps 列表中 |
| **⚠️ 批量绑定数据集：3 处典型错误（已修正 2026-04-01）** | ①API 端点：必须用 `POST /drag/onlDragDatasetHead/getAllChartData`，不是 `GET /drag/view/getAllChartData`（后者 result 为 None 报错）。②dataMapping 格式：`filed` 是槽位标签（`"维度"/"数值"/"分组"`），`mapping` 才是字段名，不是 `{"filed":"name","mappingField":"name"}`。③必须设 `fieldOption`（前端字段选项面板用）及完整字段 `dataSetName/dataSetType/dataSetApi/dataSetMethod/dataSetIzAgent/paramOption/viewLoading`，用 `cfg.update(copy.deepcopy(DS_CONFIG))` 一次写入 |
| **⚠️ linkage_ops.py 参数名和格式** | 联动映射参数是 `--mapping`（不是 `--param`），格式是 `src=tgt`（等号分隔，不是冒号），多个用逗号：`--mapping "name=name,value=keyword"` |
| **⚠️ 模板索引表只有 10 个实际文件** | SKILL.md 模板索引表只列实际存在的 10 个模板文件，禁止使用不存在的模板名（已修正，2026-04-01） |
| **⚠️ default_configs.json 变体 key 必须用 `_数字` 格式** | 组件变体 key 统一用 `CompType_数字`（如 `JStatsSummary_1`、`JScrollList_2`），禁止用中文后缀（如 ~~`JScrollList_滚动列表(多行+序号)`~~）。`_resolve_comp_type` 通过正则 `^(J[A-Za-z]+)_(.+)$` 提取实际组件类型，中文后缀虽然现在也能匹配，但 `_数字` 格式更规范一致。新增变体时遵循此规范（已修正，2026-04-01） |
| **⚠️ 变体组件的 option 和 chartData 必须各自独立** | 同一 compType 的不同变体（如 JCardScroll_1/2/3、JScrollList_1/2/3）在 `default_configs.json` 中必须有各自独立的完整 option 和 chartData，禁止共用同一份。变体之间的关键差异包括：`direction`（horizontal/vertical）、`scrollDirection`（left/up）、`showIndex`、`contentFieldMapping`（字段列表）、`cardStyle.bgHighlightImage`、chartData 的数据结构等。配置来源必须是源码 `packages/components/xxx/config.ts` 中对应的导出变量（如 `cardScrollOption1`、`ScrollListData2`），禁止自行编写或复制横向版配置充当其他变体（已修正，2026-04-01） |
| **⚠️ template_ops.py copy 集团综合数据大屏报错** | 该模板所有组件 y<150，边界缩放 `min(... if y>=150)` 返回空序列。已修复：加 fallback 取全局最小 y（2026-04-01） |
| **⚠️ JPyramid3D / JOrbitRing 默认尺寸太小** | 3D金字塔（JPyramid3D）和轨道环形文字（JOrbitRing）默认尺寸不够，生成时建议使用 **750×500**，否则显示效果不佳 |
| **⚠️ jsConfig 多行脚本不能用 bash `\n`** | `comp_ops.py edit --set "jsConfig=..."` 中的 `\n` 是字面两个字符（`\` 和 `n`），不是换行符。简单脚本用分号连接为单行（`window.open(...);return false;`），复杂多行脚本用 `link_ops.py set-js --js-file script.js` 从文件读取 |
| **jsConfig 与 linkType 独立** | `config.jsConfig` 不依赖 `linkType` 的值，两者可同时配置。执行顺序：jsHandler → (return true?) → 外部链接 → 联动 → 钻取 |
| **ECharts vs 非ECharts 的 jsHandler 差异** | ECharts 组件传 `(params, option)` 两个参数且有自动分号修复；非 ECharts 组件（useCommon.ts）只传 `(params)` 一个参数，无自动修复 |
| **⚠️ JSelectRadio 组件关联需自定义脚本** | comp_ops.py 可添加基础选项卡但无法配置 `compShowConfig`（组件显隐关联），必须用自定义脚本设置。`compShowConfig` 在 config 顶层（与 option 同级），`compVals` 必须用组件 UUID（`i` 字段）不是组件名称 |
| **⚠️ 选项卡关联组件位置应重叠** | 被选项卡控制的多个组件应放在相同坐标（x/y/w/h 一致），通过选项卡切换显隐实现"同位切换"效果 |
| **⚠️ 自定义脚本创建组件必须设置 `size`** | config 内部必须包含 `size: {width: px_w, height: px_h}`，顶层也要设置 `size: {width, height}`。`SelectRadio.vue` 通过 `props.size.height` 和 `props.config.size` 计算渲染高度，缺失则组件初始不显示（需手动拖拽才触发渲染）。同理 `chart`/`turnConfig`/`linkageConfig` 也必须设置。`bi_utils.add_component()` 已自动处理这些字段（第 420 行），自定义脚本直接操作 template 时必须手动补全 |
| **⚠️ JTabToggle 自定义脚本创建时 chartData/dataType 必须在 config 内部** | 前端通过 `:config="item.config"` 传 props，`useDataSource` 通过 `props.config.chartData` 和 `props.config.dataType` 读取数据。**所有数据字段（`chartData`、`dataType`、`dataMapping`、`containDataType`、`timeOut`、`turnConfig`、`linkType`、`linkageConfig`、`size`、`chart`）必须放在 `config` JSON 内部**，和 JBar 等组件一致。放在组件顶层会导致数据源绑定失败、显隐关联不生效。`containDataType` 固定为 `[1]`，`dataType` 为 `1`（静态数据）。`chartData` 格式为 `[{"label":"Tab名","value":"1"},...]`，`option.items` 中每项的 `value` 必须与 `chartData` 中的 `value` 匹配，`compVals` 为关联组件的 `i`（UUID）数组 |
| **⚠️ 写Java接口时自行用 find/Glob 搜索 Controller 文件路径** | 禁止自行搜索 Controller 文件，必须直接询问用户提供完整路径。已知路径：`D:\jeecgboot2025\...\controller\OnlDragMockController.java`（含 visitorFlow / visitorFlowMulti / salesFlowDouble 等接口） |
| **⚠️ 写了 Java 接口后未输出接口地址和重启提示** | 脚本末尾必须打印所有接口完整 URL + "请重启 Spring Boot 后端服务" 提示，否则用户不知道下一步操作，API 数据集始终拿不到数据 |
| **⚠️ 多图表类型未针对各自设置正确的 slot_labels** | 单系列 `['维度','数值']`；对比折线图/分组柱形图 `['维度','数值','分组']`；双轴图 `['维度','数值','数值2']`。错用 slot_labels 导致绑定后图表显示"暂无数据" |
| **⚠️ 自定义脚本内 subprocess 调用 comp_ops.py 后又在脚本外二次调用** | 两次都会成功，导致大屏出现位置和名称完全相同的两个重复组件。正确分工：**脚本只负责创建数据集，组件添加统一在脚本外用 `py comp_ops.py add --dataset-name` 完成，二者禁止混用**。 |
| **⚠️ cp 脚本文件后未验证是否成功** | cp 可能静默失败（显示 done 但文件不存在），必须在 cp 命令中加 `&& ls bi_utils.py` 或 `&& ls *.py *.json` 验证所有文件存在，否则后续 py 脚本报 `ModuleNotFoundError` |
| **⚠️ `response.get('result', {})` 接收 null 时返回 None** | JSON `"result": null` 场景下 `.get('result', {})` 返回 None 不是 {}，后续 `.get(...)` 报 AttributeError。**必须用 `(response.get('result') or {})`**，这是自定义脚本中处理任何 API 响应的强制规范 |
| **⚠️ `.get('data', [])` 对值为 null 的 key 无效（TypeError: NoneType has no len）** | `response.get('result') or {}` 只解决 result=null 的问题；若 result 是 dict 但内部字段 `{"data": null}`，则 `.get('data', [])` 返回 `None`（key 存在，default 不触发），后续 `len(rows)` 报 `TypeError`。**必须在链末加 `or []`**：`rows = ((resp.get('result') or {}).get('data') or [])`。适用所有嵌套响应字段（getAllChartData、list、queryById 等）|
| **⚠️ 自写 Java API 接口场景：getAllChartData 必须 try/except 包裹** | 自写接口需重启后端才能访问；重启前调 `getAllChartData` 会因接口 404/500 返回 `{"result":{"data":null}}`，若不处理会导致脚本崩溃，后续图表组件全部无法创建。**必须用 try/except 包裹，失败时打印警告并继续**：`try: parse_r = bi_utils._request(...); rows = ((parse_r.get('result') or {}).get('data') or []); print(f'解析:{len(rows)}条') except Exception as e: print(f'解析跳过({e})，重启后端后生效')`。绝对不能让 getAllChartData 失败阻断整个脚本 |
| **⚠️ API 数据集 /add 已含字段列表，禁止再 queryById+edit 回写（浪费 4 个请求）** | 创建 API 数据集时 `/add` 请求体已包含 `datasetItemList`（⚠️ 正确字段名，不是 `onlDragDatasetItemList`），字段已保存，无需再做 `queryById` → 修改 → `edit` 三步回写。这 4 个请求完全多余，在后端未重启时还会因 getAllChartData 返回空而得到无意义结果。**正确流程：`/add` 一次性传字段 → try/except 包裹的 getAllChartData（触发缓存）→ 结束**，共 2 个请求，不是 6 个 |
| **⚠️ API返回值可能是list不是dict** | 某些API（如 `getAllGroup`）返回的是 `list`（数组），不是 `dict`（对象），调用 `.get()` 会报 `AttributeError: 'list' object has no attribute 'get'`。**正确处理**：先判断类型 `if isinstance(result, dict): ... elif isinstance(result, list): ...`，或者直接用硬编码ID跳过查询 |
| **⚠️ 创建SQL/API数据集时必须同时传字段列表** | 创建数据集时只传 `querySql` 没有传字段列表，系统不会自动解析字段，导致字段列表为空。**正确做法**：`/add` 时同步传 `datasetItemList`（⚠️ 正确字段名，不是 `onlDragDatasetItemList`）；若字段未知，则创建后立即调用 `getAllChartData` 获取数据 → 推断字段类型（int/float→Integer，其余→String）→ `queryById` 获取实体 → 填入 `datasetItemList` → `edit` 保存 |
| **⚠️ 用 `groupCode` 设置数据集分组** | 正确字段是 `parentId`（= 分组节点的 id），`groupCode` 是另一套无关字段（通常 null）。见 pitfalls.md「数据集分组：用 parentId」条目 |
| **⚠️ `getAllGroup` 分组节点字段是 `name` 不是 `groupName`** | 用 `item.get('groupName')` 永远 None →匹配失败→重复调 `addGroup`→报唯一约束错误→`parentId=''`→数据集落入根目录（显示"0"）。**必须用 `item.get('name')`** 且过滤分组节点（`dataType is None AND dbSource is None`）。本环境"示例数据集"固定 id=`1516743332632494082`，可直接硬编码跳过查询 |
| **⚠️ SQL 错误后尝试用 information_schema 查表名** | BI 系统 SQL 注入防护会拦截 information_schema/SHOW TABLES，只能逐一尝试候选表名（`SELECT COUNT(*) FROM 候选表 LIMIT 1`） |
| **⚠️ 自定义脚本创建SQL数据集用 `dbKey` 而非 `dbSource`** | 直接调 `/drag/onlDragDatasetHead/add` 时字段名是 `dbSource`（值为数据源数字 ID），`dbKey` 是无效字段会被忽略，导致 getAllChartData 报"数据源不存在"。`comp_ops.py --db-source` 只是 CLI 参数名，不等于 API 字段名 |
| **⚠️ SQL 数据集创建后字段为空（手动点查询解析才出现）** | `/add` 时未传 `onlDragDatasetItemList`，系统不自动解析 SQL 字段。**修复①（推荐）**：`add` 请求体中同时传 `onlDragDatasetItemList`（含 fieldName/fieldTxt/fieldType/izShow/sort）。**修复②（事后补救）**：`getAllChartData` 拿返回数据第一行 → 推断类型（int/float→Integer，其余→String）→ `queryById` 获取完整实体 → 填入 `onlDragDatasetItemList` → `edit` 保存 |
| **⚠️ 直接调 `/drag/onlDragDataSource/add` 时 result 是字符串 ID** | 响应体 `result` 直接是字符串（如 `"1199640199413063680"`），直接用 `add_resp.get('result')` 即可。禁止用 `getOptions` 按名称查找（同名重复时会取到错误的旧数据源） |
| **⚠️ `/drag/onlDragDatasetHead/add` 的 result 是完整对象 dict，不是 ID 字符串** | `add_r.get('result')` 返回 `{'id':'xxx','name':'...','querySql':'...'}` 整个对象，直接当 ID 用会导致 `dataSetId` 存入 dict 字符串，图表绑定失效。**必须用** `DS_ID = (add_r.get('result') or {}).get('id')` |
| **⚠️ 数据集 `edit` 接口保存字段列表必须用 `datasetItemList`** | 后端 VO 字段名是 `datasetItemList`，用 `onlDragDatasetItemList` 静默忽略，字段不会保存。`queryAllById` 和 `queryById` 返回的也都是 `datasetItemList`。永远不要用 `onlDragDatasetItemList` 作为 key |
| **⚠️ `queryFieldByApi` 对外部 mock URL 可能返回空列表** | 解析到空时不要停止。fallback：调 `getAllChartData` 拿实际数据第一行，按值类型推断（`int/float→Integer`，其余→`String`），再调 `edit` 回写 `datasetItemList` |
| **⚠️ `queryAllById` 需要签名验证，bi_utils 不兼容** | bi_utils 签名与 `queryAllById` 的 `@SignatureValidation` 验证机制不匹配，总报"时间戳为空"。用 `queryById`（无签名要求）获取实体头部信息，子表字段用 `getAllChartData` 验证 |
| **⚠️ MySQL 8.x 的 db-type 是 `MYSQL8`，不是 `MYSQL5.7`** | datasource_ops.py create 和直接调 add API 时，MySQL 8 的 `dbType` 必须填 `MYSQL8`；MySQL 5.7 用 `MYSQL5.7`。填错会导致连接失败或 JDBC URL 生成错误 |
| **⚠️ 从其他数据库的数据集推断当前库的表名** | 不同数据源对应不同库，A 库有 `jmreport_big_screen` 不代表 B 库（jeecg-boot 主库）也有。jeecg-boot 主库大屏相关表：`onl_drag_page`（页面记录），优先尝试这个，`jmreport_big_screen` 通常在单独统计库中 |
| **⚠️ 用 `py -` heredoc 执行动态脚本** | `py -` 读 stdin 时 `sys.path.insert(0,'.')` 绑定进程启动目录而非当前 shell 工作目录，导致 `import bi_utils` 报 ModuleNotFoundError。**必须 Write 脚本到文件再 `py script.py`**，这是已知 Windows Git Bash 行为 |
| **⚠️ 执行 Python 代码前必须先复制脚本文件** | 执行 `py -c "..."` 或 `py script.py` 前，必须先 `cp bi_utils.py .`（和 comp_ops.py/dataset_ops.py 等），否则报 `ModuleNotFoundError: No module named 'bi_utils'`。**正确流程**：第一步并行复制所有需要的脚本，第二步执行操作，第三步最后清理 |
| **⚠️ 替换图表类型时共通配置未继承** | delete+add 替换图表类型时，必须从旧 `config.option` 中提取共通字段（`title/xAxis/yAxis/legend/grid/color/tooltip`）并用 `merge_common_option()` 合并到新图表的 option 中，否则用户在旧图表中定制的标题文字、坐标轴显隐、图例配置等全部丢失。`series` 字段是图表类型专属，**严禁继承**。详见 `references/pitfalls.md`「图表类型替换」章节 |
| **⚠️ JBar 柱体颜色：`option.color[0]` 不生效，必须同时改 `series[0].itemStyle.color`** | ECharts 中 `series[0].itemStyle.color` 的优先级高于 `option.color` 数组。JBar 默认配置中 `itemStyle.color` 硬编码了蓝色（`#64b5f6`），仅设置 `option.color[0]` 无法覆盖柱体颜色。**正确做法**：必须同时设置两个字段：`--set "option.color[0]=目标色" --set "option.series[0].itemStyle.color=目标色"`（2026-04-09 实测） |
| **⚠️ Online/设计器表单图表：`compStyleConfig` 不能为空 `{}`** | `useEChartsNew.ts:396` 直接访问 `config.compStyleConfig.summary`（无 optional chaining），若 `compStyleConfig={}` 则 `summary` 是 `undefined`，导致渲染报 `Cannot read properties of undefined (reading 'showTotal')`。必须包含完整 `summary: {showY,showTotal,showField,totalType,showName}` 子对象。完整模板见 `references/online-design-form-chart-guide.md` 第八节 |
| **⚠️ Online/设计器表单图表：`filter.conditionFields` 不能缺失** | `ChartSetModal.initCompConfig` 调用 `dateConditionFormat(config.filter.conditionFields)`，该函数直接 `condition.forEach()`，传 `undefined` 时报 `Cannot read properties of undefined (reading 'forEach')`（编辑弹窗打不开）。`filter` 中必须加 `'conditionFields': []`，同时建议加 `'conditionMode': 'AND'` |
| **⚠️ Online 表单图表 chart.category 必须与 constant.ts 完全一致** | 填错 category 导致后端 `getTotalDataByCompId` 数据格式错误，图表一直加载中。关键错误：**JRing 的 category 是 `Pie`（不是 `Ring`）**，JBar3d/JBarGroup3d 是 `threeD`（不是 `Bar3d`），JHorizontalBar/JRankingList/JTotalProgress 是 `HorizontalBar`（不是 `Bar`）。权威来源：`constant.ts` 的 `chartConfig` 函数中父级节点的 `type` 字段 |
| **⚠️ Online 表单图表 onlyValueChart 必须严格按源码判断** | 仅 `['Gauge','Number'].includes(category) \|\| ['JTotalProgress','JLiquid'].includes(subclass)` 时 nameFields 才为空。**Ring 和 Progress 分类的图表不是 onlyValueChart**：JRingProgress/JActiveRing/JRadialBar（Ring 类）、JCustomProgress/JProgress/JListProgress/JRoundProgress（Progress 类）都需要 nameFields。错误地将它们设为仅值会导致图表无法渲染 |
| **⚠️ Online 表单图表 isGroup 图表必须设置 typeFields** | `FieldConfig.vue` 第 217 行：`v-if="valueFields.length < 2 && isGroup && !onlyValueChart"`。isGroup=true 的组件（JStackBar/JMultipleBar/JNegativeBar/JMixLineBar/JPercentBar/JMultipleLine/JRadar/JCircleRadar/JBubble/JQuadrant/JBarGroup3d/JPivotTable）若 typeFields 为空，前端无数据显示。批量生成时应设 `typeFields=[维度字段]` |
| **⚠️ DoubleLineBar 双轴图 nameFields 必须为空，需设 assistYFields** | 切换到 DoubleLineBar 时前端清空 nameFields，正确结构：`nameFields=[], valueFields=[值], typeFields=[维度(主分组)], assistYFields=[值(辅助Y轴)], assistTypeFields=[维度(辅助分组)]`。assistYFields 显示条件：`component == 'DoubleLineBar'`；assistTypeFields 显示条件：`assistYFields.length < 2 && subclass == 'DoubleLineBar'` |
| **⚠️ Online 表单图表：`record_count` 的 `fieldType` 必须是 `'count'`，绝不能是 `'int'`** | 后端按 `fieldType` 选聚合函数：`'count'`→`COUNT(*)`（正确）；`'int'`/`'String'`→`SUM(record_count)`→展开为 `SUM(*)`→SQL 语法错误（报错 `SELECT sum(*) record_count ...`）。**强制写法**：`{'fieldName':'record_count','fieldTxt':'记录数量','fieldType':'count','widgetType':'count','dictCode':''}`。凡将 `record_count` 放入任意字段列表时必须遵守（2026-04-09 实测）|
| **⚠️ FILES 多文件数据集场景写临时脚本而非用 `files_ops.py`** | `files_ops.py create-bind` 已封装全部流程（创建数据集→上传文件→自动探查列名→推断JOIN SQL→绑定图表），1轮命令搞定，禁止再现写 `files_chart.py` 等临时脚本浪费 2 轮 Write |
| **⚠️ `files_ops.py create-bind` 不传 JOIN 参数直接报 `UnboundLocalError`（2026-04-09 实测）** | `create-bind` 有三种执行路径：①`len(files)==2 and group_by and agg` → JOIN 模式，自动探查列名推断SQL；②`len(files)==1` → 单表简单查询；③其余情况 → 走 `else` 分支引用未初始化的 `ordered_tables` 变量，必定报 `UnboundLocalError`。**必须传足三个参数**才能进入 JOIN 模式：`--group-by "列名1,列名2" --join-on "关联列名" --agg "聚合列名"`（列名是 Excel 实际列名，如 `--group-by "region,category" --join-on "product_id" --agg "amount"`）。**列名未知时：禁止盲目执行，必须先询问用户 Excel 各列名称** |
| **⚠️ 文件数据集 value 是 SQL 关键字** | SQL 解析器将 `value` 识别为关键字而非字段别名，报错 `Encountered "value" at line 2`。**必须避开 value 关键字**，用 `sales`、`amount`、`total` 等作为字段别名 |
| **⚠️ 文件数据集重复文件冲突** | 同一页面多次上传同名文件会导致 H2 表名冲突（`Multiple entries with same key`），H2 内存数据库损坏后所有 SQL 查询返回 null。**`files_ops.py` 已在 `upload_file` 中自动给文件名加 HHMM 时间戳后缀**（如 `products_1733.xlsx` → 表名 `jmf.Sheet1_products_1733_excel`），确保每次上传文件名唯一，无需手动处理 |
| **⚠️ 文件数据集字段映射用实际字段名** | dataMapping 中 `mapping` 必须与 SQL 字段别名一致（如 SQL 用 `as sales`，dataMapping 用 `mapping: 'sales'`），不能用 `value` |
| **⚠️ files/get 的 result 是 dict 不是 list（2026-04-09 实测）** | `GET /jmreport/source/datasource/files/get` 返回 `{"result":{"id":"...","dbUrl":"[{...}]",...}}`，`result` 是对象不是数组。**必须**：`file_list = json.loads((files_resp.get('result') or {}).get('dbUrl', '[]'))`，不能用 `(files_resp.get('result') or [])` 当 list 迭代 |
| **⚠️ queryFileFieldBySql 需签名验证，bi_utils 调用报"时间戳为空"** | bi_utils 的签名机制与 `queryFileFieldBySql` 的 `@SignatureValidation` 不兼容，始终报 500"时间戳为空"。**替代方案**：字段已知时直接在 `edit` 时手动传 `datasetItemList`；用 `POST /drag/onlDragDatasetHead/getAllChartData` 验证数据是否正常返回 |
| **⚠️ FILES 多文件 SQL 别名不能用单引号（2026-04-09 实测）** | `SELECT col as 'type'` 会导致 SQL 解析错误。**必须用不带引号的别名**：`SELECT col as type`（MySQL 风格单引号别名在文件数据集 SQL 解析器中不支持） |
| **⚠️ Excel 文件表名推断规律** | `xxx.xlsx` 上传后系统生成表名格式为 `jmf.Sheet1_{xxx}_excel`（如 `products.xlsx` → `jmf.Sheet1_products_excel`，`orders.xlsx` → `jmf.Sheet1_orders_excel`）。当 files/get 解析失败或表名列表为空时，可用此规律作为 fallback 直接构造 SQL |
| **⚠️ api.jeecg.com 是 YApi mock 服务器，禁止尝试 JeecgBoot 登录** | `api.jeecg.com` 根路径返回 YApi HTML，不是 JeecgBoot 后端。用户给出该域名 + email/password 时，这是 **YApi 凭据**（登录路径 `/api/user/login`，传 `{email, password}`），**不是** JeecgBoot `/sys/login`。已有公开 mock 接口（`/mock/51/*`、`/mock/26/*`）无需任何登录即可访问。大屏操作始终用内存中存储的本地 JeecgBoot token（192.168.1.66），不要对 api.jeecg.com 尝试 JeecgBoot 路径 |
| **⚠️ page_ops.py rename 的 --name 是命名参数** | 正确：`py page_ops.py rename API_BASE TOKEN PAGE_ID --name "新名称"`。错误：~~把名称作为第4个位置参数~~，会报 `error: unrecognized arguments` |
| **⚠️ page_ops.py 不支持 delete 命令** | `page_ops.py` 只支持 `info/set-bg/set-bgimg/set-theme/watermark/rename`，**没有 delete 命令**。删除大屏必须直接调 API：`bi_utils._request('DELETE', '/drag/page/delete', params={'id': PAGE_ID})` |
| **⚠️ comp_ops.py delete 报成功但组件实际未删除（已修复 2026-04-08）** | 根因：`bi_utils.save_page` 第 303 行 `if not components:` 把有意清空的 `[]` 当作"未缓存"，用服务端旧模板覆盖。已修复：改为 `if page_id not in _page_components:`，区分"主动清空"与"未设置"。同步修复：移除 save_page 内部的多余 `query_page` 调用（节省 1 次 API）；新增 `desJson` 字段回传避免页面宽高丢失；`query_page` 同步缓存 `desJson`。**当前版本 comp_ops.py delete/edit/move 均可直接使用，无需自定义脚本** |

> 完整踩坑记录见 `references/pitfalls.md`

## 错误处理

| 错误 | 解决方案 |
|------|---------|
| Token 过期（401） | 重新获取 X-Access-Token |
| `updateCount` 不匹配 | 重新查询页面获取最新值 |
| 组件不显示 | 检查 dataType、chartData、option 完整性 |
| 中文乱码 | 使用 Python（不要用 curl） |

## 参考文档

- `references/bi-component-types.md` — 完整组件类型清单
- `references/bi-comp-option-config.md` — 组件样式配置路径
- `references/bi_utils.py` — 工具库源码
- `references/core-configs/data.ts` — 组件面板菜单树 + 初始化 config（原始源码，353KB）
- `references/core-configs/optionData.ts` — 组件属性面板配置项列表（原始源码，57KB）
- `references/core-configs/component-defaults.md` — 82+ 组件默认配置速查（尺寸/chartData/option/dataMapping）
- `references/core-configs/addPageComp-logic.md` — 组件创建流程（addPageComp 函数、newItem 结构、位置计算）
- `references/core-configs/menu-hierarchy.md` — 组件菜单分类树（完整层级 + 统计）
- `references/templates/bigScreen/` — 10 个大屏模板 JSON
- `references/scripts/` — 12 个预置操作脚本 + default_configs.json
