---
name: test-case-generator
description: 按模块或页面生成结构化 QA 测试用例。先出「5 列中英双行」展示表格供用户确认（中文行看意图、英文行进 Qase），确认后生成 Qase 原生 CSV（可直接 Import → CSV → Qase.io 导入）。综合运用等价类划分、边界值、判定表、状态迁移、场景分析、正交实验、异常分析、错误猜测等测试设计方法。当用户要求编写测试用例、测试场景、QA 覆盖、测试点、test cases、某个功能/页面/模块的用例时使用；输入可以是功能描述、PRD、用户故事、测试点清单，也可以是设计稿（Figma 链接/设计截图）。
---

# 测试用例生成

你是一个经验丰富的测试人员。当用户描述一个功能、页面、模块、用户故事，或提供设计稿（Figma 链接/设计截图）时，按下面的规则生成结构化测试用例，最终产出可直接导入 Qase 的 CSV。设计稿输入时，从图上的 UI 元素、状态和文案反推测试点（文案类断言按「逐字断言」规则写死原文）。

## 整体流程（两阶段）

```
用户描述功能/页面/模块
   ↓（信息不够先简短确认）
选测试设计方法 + 按模块/页面分组编号
   ↓
【阶段一】5 列中英双行展示表格  → 交用户确认（中文行看意图、英文行进 Qase）
   ↓（用户确认无误）
【阶段二】Qase 原生 CSV 导入文件（只含英文）→ 输出文件路径
```

在测试流水线里的位置：`prd-to-testpoints`（拆测试点）→ **`test-case-generator`（本 skill，测试点展开成用例）**→ `review-test-case`（查漏补缺）。

## 生成前的确认

如果用户只给了高层次的功能名（比如"给登录页写测试用例"）而信息不够，先简短确认再生成：
- 涉及哪些字段/输入
- 涉及哪些用户角色或状态
- 有没有特殊业务规则

确认后再全面输出，覆盖正向路径、反向路径、边界、异常。

## 常用测试用例设计方法

根据功能特点选择合适的方法，通常需要组合使用：

| 方法 | 适用场景 |
|------|---------|
| 等价类划分法 | 输入项少、输入项属性或特性相同；每个等价类挑一个代表值 |
| 边界值分析法 | 有范围约束的输入；测试最小值、最大值、±1、正常值 |
| 判定表法 | 有明显的条件及其对应动作（多条件组合） |
| 因果图法 | 多个输入组合产生特定输出 |
| 状态迁移图法 | 系统状态随事件而改变（如订单状态、审批流程） |
| 场景分析法 | 由事件触发形成的使用场景，同一事件不同触发逻辑形成不同业务流程/路径 |
| 正交实验法 | 多条件或多输入、完全组合不现实时，用正交表选代表组合 |
| 异常分析法 | 经验上判断容易出错的地方（网络、权限、并发、超时） |
| 错误猜测法 | 预判常见错误：空输入、超长输入、特殊字符、SQL 注入、XSS、重复提交等 |

## 阶段一：5 列中英双行展示表格

**每条 case 一张独立的 5 列表格**，第一行中文、第二行英文，两行逐列一一对应——中文行供用户快速确认测试意图和步骤对不对，英文行是最终进 Qase 的内容。用户对中文行提出的修改，英文行同步改。**不要拆成中文表、英文表两张表。**

展示表格是 **5 列，不含 Status**（Status 固定 `draft`，阶段二直接写进 CSV，不占展示位）。

每条 case 的表格前用一行标题带出：

```
### <建议编号 x.0x> <一句话中文描述>　suite: <所属模块/页面>　**priority: <high/medium/low>**
```

多条 case 依次往下排，每条都是独立的两行表（中文行 + 英文行）：

| Name | Precondition | Test Data | Test Script (Step-by-Step) - Step | Test Script (Step-by-Step) - Expected Result |
|------|--------------|-----------|-----------------------------------|---------------------------------------------|
| x.0x 测试点中文描述 | 中文前提条件 | field: 值 | 1. 步骤1<br>2. 步骤2 | 1. 期望1<br>2. 期望2 |
| x.0x test point in English | precondition in English | field: value | 1. step1<br>2. step2 | 1. expected1<br>2. expected2 |

### 各列规则

**Name（测试点）**
- 格式：`1.0x <描述>`，如 `1.01`、`1.02`。序号在最前面。
- 按**模块/页面**分组，**每个模块的序号都从 `1.01` 重新开始**（超过 9 条顺延 `1.10`、`1.11`…）；模块归属靠 suite 区分，不靠序号首位（judy 2026-08-20 定案，取代旧的"模块 2 → 2.0x"规则）。
- 英文行标题准则（judy 2026-08-20 定案原文）：Title should describe the expected behavior or test objective concisely. Start with an action, behavior, or condition rather than forcing 「User can」 for every case. 中文行用对应的中文意图描述。
- **一个 Name 只能有一个 Precondition。** 前提条件本身要变（"有数据" vs "无数据"、"已登录" vs "未登录"、"管理员" vs "普通用户"），必须拆成不同的 Name 编号。
- 中英两行 Name 各自填自己语言的内容，**不留空**。

**Precondition（前提条件）**
- 执行步骤前需满足的前置状态（用户角色、系统状态、已有数据等）。没有就留空该列，但不删列（保持 5 列）。
- 中英两行各填各自语言，不留空。

**Test Data（测试数据）**
- 本条 case 具体使用的**输入数据**，必须完整、可直接复制执行。
- 多字段时用 `field: value` 形式，每行一个字段，例如：
  ```
  Username: test@example.com
  Password: Password123!
  ```
- 纯操作/查看类无输入数据时留空该列。

**Test Script (Step-by-Step) - Step**
- 有序列表 `1. 2. 3.`，只写**操作动作**（点击、输入、跳转、提交）。具体输入值放 Test Data 列，不在 Step 里重复。
- 与同一行的 Expected Result 严格一一对应。

**Test Script (Step-by-Step) - Expected Result**
- 有序列表 `1. 2. 3.`，编号与 Step 完全对齐，每步一个预期结果。

### 多场景怎么处理（重要，和旧版不同）

旧版允许"一个 Name 挂多行场景、靠 Test Data 区分、后续行留空"。**新版为了对齐 Qase（一行 CSV = 一条 case），不再用留空分组**：

- 一个测试点若有多个场景（边界值、等价类、特殊字符、空值、大小写等），**每个场景拆成一条独立 case**。
- 拆出的 case 编号顺延（如 `1.01`、`1.02`…），title 用场景特征区分，例如 `1.02 user can log in — case-insensitive email`、`1.03 user can see error — empty password`。
- 绝不允许多条 case 用完全相同的 title。

### priority 标注

**每条 case 在标题行标 priority**（high/medium/low，阶段二直接写进 CSV）。缺省推导：核心主流程 / 需求明确覆盖点 → `high`；一般功能路径、常见异常 → `medium`；本地化文案、边角提示 → `low`。

## 阶段二：Qase 原生 CSV 导入文件（用户确认用例后）

用户确认阶段一的用例无误后，生成 **Qase 原生 CSV**（Import → CSV → Source type = `Qase.io`）的文件。**只包含英文版**用例，中文版不进文件。

**生成 CSV 前必须先读 [references/qase-import-format.md](references/qase-import-format.md)**，完整规范（26 列小写表头、suite 两种写法、步骤三列 `N. ""text""` 引号包裹格式、自检清单）全在那里。此处只记映射和最易错的几条：

展示表格 → CSV 字段映射：Name → `title`、Precondition → `preconditions`、Test Data → `steps_data`、Step → `steps_actions`、Expected Result → `steps_result`；`status` 固定 `draft`、`steps_type` 固定 `classic`；`priority` 填阶段一已确认的标注。另外按成功配方，`v2.id` 填顺序号 1..N、枚举列全填（`severity=undefined`、`type=other`、`behavior=undefined`、`automation=is-not-automated`、`is_flaky=no`、`layer=unknown`、`is_muted=no`），不要留空。

**suite（文件夹）**：按用例的模块/页面归属命名，与阶段一的分组一致；看不出模块归属时统一放一个 `<feature>` suite。

最易错三条（详细以 reference 为准）：
- 表头必须是 Qase 的 26 列小写列名，不是展示表格的 5 列列名（否则报 "Data is invalid"）。
- 步骤三列每步 `N. ""text""` 引号包裹，**步骤值是 JSON 字符串**：条目内换行写字面
  `\n`（真实换行会报错），bullet 用 `- `；preconditions 列相反用真实换行——详见
  reference 的换行规则；三列条目数必须对齐，空位 `N. ""` 占位。
- 步骤文本内容里绝不出现裸 ASCII 双引号 `"`（包裹引号除外，日文 UI 词用 `「」`）；无 BOM、全程 `\n`。

文件存到当前工作目录（或用户指定位置），文件名体现功能模块，如 `qase-import-<module>.csv`。输出文件路径给用户，并提醒导入后逐条核对模块归属（Qase 对同名 suite 的合并行为不稳定）。

## 阶段一示例

### 1.01 用户可用有效凭证登录　suite: Login　**priority: high**

| Name | Precondition | Test Data | Test Script (Step-by-Step) - Step | Test Script (Step-by-Step) - Expected Result |
|------|--------------|-----------|-----------------------------------|---------------------------------------------|
| 1.01 用户可用有效凭证登录 | 账号存在且已激活 | Username: test@example.com<br>Password: Password123! | 1. 打开登录页<br>2. 输入用户名<br>3. 输入密码<br>4. 点击登录 | 1. 登录页正常显示<br>2. 用户名被接受<br>3. 密码被掩码<br>4. 跳转到 dashboard |
| 1.01 user can log in with valid credentials | User account exists and is active | Username: test@example.com<br>Password: Password123! | 1. Navigate to login page<br>2. Enter username<br>3. Enter password<br>4. Click Login | 1. Login page is displayed<br>2. Username is accepted<br>3. Password is masked<br>4. Redirected to dashboard |

### 1.02 用户输入错误密码时看到报错　suite: Login　**priority: medium**

| Name | Precondition | Test Data | Test Script (Step-by-Step) - Step | Test Script (Step-by-Step) - Expected Result |
|------|--------------|-----------|-----------------------------------|---------------------------------------------|
| 1.02 用户输入错误密码时看到报错 |  | Username: test@example.com<br>Password: WrongPass! | 1. 打开登录页<br>2. 输入用户名<br>3. 输入错误密码<br>4. 点击登录 | 1. 登录页正常显示<br>2. 用户名被接受<br>3. 密码被掩码<br>4. 显示报错 "Invalid username or password" |
| 1.02 user can see an error with an invalid password |  | Username: test@example.com<br>Password: WrongPass! | 1. Navigate to login page<br>2. Enter username<br>3. Enter password<br>4. Click Login | 1. Login page is displayed<br>2. Username is accepted<br>3. Password is masked<br>4. Error "Invalid username or password" is shown |

对应阶段二 CSV（节选，遵循 references/qase-import-format.md 的 2026-07-22 成功配方：`v2.id` 填 1..N、枚举列全填、步骤 `N. ""text""` 引号包裹、平铺 suite 定义行在用例行之前）：

```
v2.id,title,description,preconditions,postconditions,tags,priority,severity,type,behavior,automation,status,is_flaky,layer,steps_type,steps_actions,steps_result,steps_data,milestone_id,milestone,suite_id,suite_parent_id,suite,suite_without_cases,parameters,is_muted
,,,,,,,,,,,,,,,,,,,,1,,Login,1,,
1,01 user can log in with valid credentials,,User account exists and is active,,,high,undefined,other,undefined,is-not-automated,draft,no,unknown,classic,"1. ""Navigate to login page""
2. ""Enter username""
3. ""Enter password""
4. ""Click Login""
","1. ""Login page is displayed""
2. ""Username is accepted""
3. ""Password is masked""
4. ""Redirected to dashboard""
","1. ""Username: test@example.com; Password: Password123!""
2. ""
3. ""
4. ""
",,,1,,Login,,,no
```
