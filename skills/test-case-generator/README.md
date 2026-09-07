# test-case-generator

Claude Code QA skill：按模块或页面生成**结构化 QA 测试用例**，最终产出**可直接导入 Qase 的 CSV**。

两阶段输出：

1. **阶段一 — 5 列中英双行展示表格**（Name / Precondition / Test Data / Test Step / Expected Result，无 Status）。每条 case 中文一行、英文一行逐列对应：中文行供快速确认测试意图，英文行是进 Qase 的内容。
2. **阶段二 — Qase 原生 CSV**（用户确认阶段一后生成，只含英文，26 列小写表头，可直接 Import → CSV → `Qase.io` 导入）。

综合运用 等价类划分、边界值、判定表、状态迁移、场景分析、正交实验、异常分析、错误猜测 等测试设计方法。

## 何时用

要求：编写测试用例 / 测试场景 / QA 覆盖 / 某个功能 / 页面 / 模块的用例。

## 在测试流水线中的位置

```
prd-to-testpoints  →  test-case-generator  →  review-test-case
（需求拆测试点）        （测试点展开成用例）      （用例查漏补缺）
```

本 skill 是中间一步：把 `prd-to-testpoints` 拆出的测试点展开成详细用例，再交给 `review-test-case` 查漏。输出格式（5 列中英双行 + Qase CSV）与 `review-test-case` 完全对齐，Qase 导入规范见 `references/qase-import-format.md`。

## 安装到 Claude Code

```bash
git clone <this-repo-url> ~/.claude/skills/test-case-generator
```

克隆后 `SKILL.md` 直接就位，Claude Code 即可加载。
