# Qase 导入 CSV 格式规范（完整版）

生成 Qase 可导入 CSV 时逐条遵守本文件。来源：Qase 官方 v2 模板 + 实际导入成功的文件
（`MDX-12330-qase-import.csv`、最小验证样例 `qase-test-min.csv`，均在
`/Users/judy/Documents/Claude/JMA testing/feature testing/PLX Contingency Plan - Display Static Coupon/`）。

## 文件级要求

- 编码 UTF-8，**无 BOM**
- 行结束符全程 **`\n`**（包括步骤单元格内部换行），不能是 CRLF
- 逗号分隔；含逗号/换行/引号的字段整体用 `"` 包裹，内部引号按 CSV 规范翻倍成 `""`
- 用例内容**纯英文**（日文 UI 词保留原文并用 `「」` 包裹）
- 导入入口：Qase 项目 → Import → **CSV** → Source type = `Qase.io`

## 结构

```
1 行表头
N 行 suite 定义（可选，需要控制文件夹顺序/嵌套时用）
M 行用例
```

### 表头（26 列小写列名，一字不差照抄，勿改大小写）

```
v2.id,title,description,preconditions,postconditions,tags,priority,severity,type,behavior,automation,status,is_flaky,layer,steps_type,steps_actions,steps_result,steps_data,milestone_id,milestone,suite_id,suite_parent_id,suite,suite_without_cases,parameters,is_muted
```

用展示表格的 TestRail 式列名（`Name` / `Test Script (Step-by-Step)...`）导入会报 **"Data is invalid"**。

## Suite（文件夹）两种写法

**简单式**（suite 少、不关心顺序）：不写 suite 定义行，用例行的 `suite` 列直接填文件夹名、
`suite_id` 留空，Qase 自动建目录。文件夹要排序就用两位数前缀命名（`01 Mode Switch`、`02 Home`）。

**完整式**（suite 多、要保证顺序或嵌套）：所有用例行之前，每个 suite 一行定义，只填 3 个字段，其余全空：

- `suite_id`：自定义数字编号（1, 2, 3…）
- `suite`：suite 名称
- `suite_without_cases`：固定 `1`

```
,,,,,,,,,,,,,,,,,,,,1,,01 Mode Switch — Remote Config,1
,,,,,,,,,,,,,,,,,,,,2,,02 Home,1
```

**⚠️ 嵌套 suite 勿用（2026-07-22 实测）**：定义行填 `suite_parent_id` 做父子嵌套 + 用例行
同时填 suite_id/suite_parent_id，会报 "Invalid file structure"。定义行**只建平铺（顶层）suite**、
parent 留空；用例行填 `suite_id` + `suite` 名称（parent 留空），名称与定义行完全一致。
需要挂到已有目录树下的，导入后在 UI 里手动拖整个文件夹。

## 用例行字段

| 字段 | 填法 |
|---|---|
| `v2.id` | 留空（新建用例） |
| `title` | 用例标题（对应展示表格 Name） |
| `description` | 长备注：实测日期、参考行为、待确认的具体问题（不进步骤文本） |
| `preconditions` | 前置条件；多条时用单元格内真实换行 + `- ` bullet（勿用字面 `\n`） |
| `priority` | `high` / `medium` / `low`（**不是** P0/P1/P2） |
| `status` | 固定 `draft`（合法值：actual/draft/deprecated） |
| `steps_type` | 固定 `classic` |
| `steps_actions` | 步骤（对应 Step），格式见下 |
| `steps_result` | 期望结果（对应 Expected Result） |
| `steps_data` | 测试数据（对应 Test Data） |
| `suite_id` + `suite` | 按上面选定的 suite 写法填 |
| 其余字段 | 全部留空 |

## 步骤三列格式（最容易出错，逐条检查）

三列（`steps_actions` / `steps_result` / `steps_data`）都是**一个单元格内的多行文本**：
每步一行，形如 `N. 步骤文本`。Qase 靠行首的 `N. ` 编号 + 换行切分步骤。

**⚠️ 步骤文本必须用 `N. ""text""` 引号包裹（2026-07-22 实测反转）**：2026-07-22 导入
MDS review supplement 时，**裸写步骤报 "Invalid file structure" 导不进去**；改回官方模板的
引号包裹写法（同 MDX-13157-v7 结构）后导入成功。2026-07-17 曾记录"裸写可导入、引号会原样
显示"，现以 2026-07-22 实测为准：**一律引号包裹**；若导入后 Qase 步骤里显示多余引号，属
显示问题，在 UI 里处理。空位占位也写成 `N. ""`（引号内空）。raw CSV 里长这样：

```
"1. ""Set RC to ON in backend""
2. ""Cold-start the App""
"
```

硬规则：

1. **三列条目数必须一一对齐**（编号相同、行数相同），不对齐导入后步骤错位。
2. 某步没有对应结果/数据时该行写**引号包裹的空占位**：`N. ""`（编号 + 空引号，和正文的引号包裹规则一致）。
   常见做法：actions 每步都写；result 只在有校验点的步骤写、其余占位；data 没有就全占位。
3. **步骤文本内容里绝不出现裸 ASCII 双引号 `"`**（会冲乱解析报 "Invalid file structure"）：
   日文 UI 词用 `「」`，其它引用用单引号。
4. **步骤值本质是 JSON 字符串（2026-08-19 实测，Qase 报错原文：step values must be
   JSON-encoded）**。因此：
   - 条目内**禁止真实换行**——会直接报 "not valid JSON" 拒绝导入；
   - 条目内要换行时，写**字面转义 `\n`**（反斜杠+n 两个字符），Qase 渲染成真实换行；
   - `\n` 后以 `- ` 开头的行渲染成真正的 bullet 列表（list icon）；
     行首缩进两个空格 `  - ` 可做嵌套子列表；
   - preconditions 列规则**相反**：它是普通文本列，多条前置用**单元格内真实换行**
     + `- ` bullet；写字面 `\n` 会原样显示成文本。两类列的规则不可混用。
5. 若某次导入后发现多个步骤被并成一步（编号切分失效），再回退到官方模板的
   `N. ""text""` 引号包裹写法试一次，并把结果记回本文件。
6. **英文文风与排版（judy 2026-08-19 定案）**：
   - title 格式 `1.0x <描述>`，**每个模块（suite）的序号都从 1.01 重新开始**，编号后首字母
     大写；标题准则（judy 2026-08-20 定案原文，取代 2026-08-19 的 "x.0x User can ..."
     强制句式）：Title should describe the expected behavior or test objective
     concisely. Start with an action, behavior, or condition rather than forcing
     「User can」 for every case.
   - **一条断言一个 `- ` bullet**：任何字段（步骤动作、期望结果、测试数据、前置条件）
     内容有 2 条及以上并列项时，每项独立一行，一行只说一件事——不写长从句大段落；
     多字段 Test Data 每字段一个 bullet；
   - 控件名写成「英文名词 + 日文原文」（如 The save button 「お届け先を保存」、
     the checkbox 「ピンの位置を確認しました」、the current location link 「現在位置へ移動」）；
   - **规格待确认的用例，在 title 最前面加 `[待确认]` 前缀**（如
     `[待确认]2.04 User can see the pin behavior when the address text is changed`），
     待确认的具体问题写在 description；拿到答复后去掉前缀并写死断言；
   - 实测日期、参考行为等其余长备注放 description 列，不进步骤文本。

## 展示表格 → CSV 转换规则（第二阶段 5 列表格转 Qase 行时逐条执行）

1. **一个 Name 多行场景必须拆分**：展示表格允许一个 Name 下挂多行场景（Name 只在首行显示、
   靠 Test Data 区分）。Qase 一行 CSV = 一条 case，所以 N 行场景拆成 N 条 case，
   title = 原 Name + ` — <场景特征>` 后缀（取该行 Test Data 的特征，如 ` — empty username`、
   ` — 256-char input`），绝不允许多条 case 同 title。
2. **Test Data 整块进第 1 步的 data 条目**：多字段时每字段一个 `- ` bullet、
   用字面 `\n` 分行（例：`1. "- Username: test@example.com\n- Password: Password123!"`），
   其余步骤 `N. ""` 占位。能明确对应到某个输入步骤的数据，放对应步骤的条目。
   Test Data 为空则全部占位。
3. **Expected Result 不足步骤数时补齐占位**：展示表格 Step 与 Expected 编号一一对应，正常无需
   处理；若个别步骤无期望结果，转 CSV 时该步补 `N. ""`，保证三列条目数对齐。

## 完整最小示例（2026-07-22 实测成功配方，MDX-13157-v7 同款结构）

关键点：`v2.id` 填 1..N、枚举列全填、步骤 `N. ""text""` 引号包裹（空位 `N. ""`、每个 steps 单元格末尾带一个换行）、平铺 suite 定义行在用例行之前。

```
v2.id,title,description,preconditions,postconditions,tags,priority,severity,type,behavior,automation,status,is_flaky,layer,steps_type,steps_actions,steps_result,steps_data,milestone_id,milestone,suite_id,suite_parent_id,suite,suite_without_cases,parameters,is_muted
,,,,,,,,,,,,,,,,,,,,1,,Coupon,1,,
1,01 ON shows static coupon page,,RC=ON,,,high,undefined,other,undefined,is-not-automated,draft,no,unknown,classic,"1. ""Enter Coupon tab""
","1. ""Shows static coupon page (not the normal dynamic list)""
","1. ""
",,,1,,Coupon,,,no
```

## 生成后自检清单

- [ ] 表头 26 列、全小写、逐字符与上文一致
- [ ] 无 BOM、无 CRLF
- [ ] `v2.id` 填顺序号 1..N（导入仍是新建，不会和已有 case 冲突）
- [ ] 枚举列全填：`severity=undefined`、`type=other`、`behavior=undefined`、`automation=is-not-automated`、`is_flaky=no`、`layer=unknown`、`is_muted=no`
- [ ] 每条用例 `status=draft`、`steps_type=classic`、priority 是 high/medium/low
- [ ] 三个 steps 列条目数对齐，空位用 `N. ""`（编号 + 空引号）占位
- [ ] 步骤文本**用 `N. ""text""` 引号包裹**（2026-07-22 实测；裸写会报 "Invalid file structure"），每个 steps 单元格末尾带一个换行
- [ ] 内容无裸 ASCII `"`（包裹引号除外）
- [ ] 一个 Name 多行场景已拆成多条 case，无重复 title
- [ ] 每个步骤条目 `N. ` 之后的部分能被 json.loads 解析（无真实换行、无裸引号）；
      条目内换行用字面 `\n`，bullet 用 `- `，嵌套用 `  - `
- [ ] preconditions 多条时用真实换行 + `- ` bullet，且未混入字面 `\n`
- [ ] suite 定义行（如用完整式）在所有用例行之前；用例行 suite_id + suite 名称与定义一致

## 参考

- 官方示例模板（拿不准逐字节对照）：https://drive.google.com/file/d/1QGcRpFcoQk8tJnObh_u7E-gRVjhsJi9c/view
- 本地实测最小样例（优先参照）：`/Users/judy/Documents/Claude/JMA testing/feature testing/PLX Contingency Plan - Display Static Coupon/qase-test-min.csv`
- 大规模实战文件（完整式 suite + 多 suite 排序）：同目录 `MDX-12330-qase-import.csv`

## 2026-07-22 实测追记（MDS review supplement，5 条一次导入成功的完整配方）

失败两次 → 成功一次，变量收敛结论：

- ❌ 嵌套 suite 定义行（带 parent）+ 用例行填 suite_id/parent → Invalid file structure
- ❌ 简单式（无定义行、v2.id 空、severity/type 等全空、裸写步骤）→ Invalid file structure
- ✅ **MDX-13157-v7 同款结构**（本次成功）：
  - `v2.id` 填顺序号 1..N（不会和项目已有 case 冲突，导入仍是新建）
  - 步骤三列 `N. ""text""` 引号包裹，空位 `N. ""`，每个 cell 末尾带一个换行
  - 枚举列全填：`severity=undefined, type=other, behavior=undefined,
    automation=is-not-automated, status=draft, is_flaky=no, layer=unknown, is_muted=no`
  - 平铺 suite 定义行（suite_id, suite, suite_without_cases=1），用例行填 suite_id+suite 名
- 导入后行为：与项目已有 suite **同名**时可能被自动合并（maintenance window 合并了），
  也可能不合并而是顶层新建（Late night limit / Cross-over 没合并）——两种都会留下顶层
  空壳/新 suite，导入后需手动拖用例进原目录、删空壳。提前告知用户这一步。
- **模块（suite）定义行必须和用例行严格一致（judy 2026-07-22 要求）**：
  1. 定义行只为文件里**实际有用例**的 suite 生成，名称与用例行 `suite` 列逐字符一致，
     不多建、不少建；
  2. Qase 对已有同名 suite 的归属行为不稳定（同一文件里有的合并进旧目录、有的落在新建
     目录），所以**导入完成后必须逐条核对**：每条用例所在的模块 = CSV 里写的模块，
     出现空壳模块/用例落错目录时，指导用户拖用例、删空壳，直到模块树与用例归属一致。
