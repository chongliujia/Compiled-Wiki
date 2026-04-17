# Compiled Wiki

一个基于编译器原则构建的知识工作区。

默认文档在本英文文件 [README.md](README.md) 中维护。中文文档见本文件。

Compiled Wiki 将来源材料、结构化中间表示和人类可读页面分层管理：

- `raw/`：存放不可变的来源 bundle。
- `ir/`：存放 claims、entities、conflicts、extracts 和 candidate claims。
- `wiki/`：存放渲染后的人类可读视图。
- `logs/changes.log`：存放追加式历史记录。

自动化代理在 ingest 来源或回答查询前，必须先遵循 [AGENTS.md](AGENTS.md)。

---

## 架构

| 层 | 路径 | 作用 |
|-------|------|------|
| 来源层 | `raw/` | 原始 PDF、Markdown 导出、转录、快照；代理只读 |
| IR 层 | `ir/` | Ground-truth 结构化 claims、entities 和 conflicts |
| Wiki 层 | `wiki/` | 人类可读渲染页面 |
| 日志层 | `logs/` | 追加式 ingest 与维护历史 |
| Schema 层 | `compiler/schemas/` | IR 文件的 JSON Schema 定义 |

wiki 是视图，IR claim 层才是编译后知识的事实层。

---

## 核心模型

`ir/claims.json` 中的 claim 包含 `subject`、`predicate`、`object`、`claim_type`、`source_id`、`evidence_span`、`status`。

规则：

- 没有来源的 claim 不允许存在。
- 推理内容不能标注为 `SourceFact`。
- 不确定时使用 `unknown`。
- 相互冲突的 claim 必须记录到 `ir/conflicts.json`，不能直接合并抹平。

`ir/entities.json` 中的实体会把别名解析为规范名，并把 claim 关联到 wiki 页面。

---

## 工作流

当在 `raw/` 下加入新来源时：

1. 解析候选 claims 与 entities。
2. 规范化命名并去重。
3. 与已有 entities 和 claims 做对齐解析。
4. 校验归因和 claim 类型。
5. 写入前先规划受影响的实体、页面和冲突。
6. 仅 patch 受影响的 IR/wiki 局部。
7. 从 IR 渲染 wiki 视图。
8. 追加写入 `logs/changes.log`。

查询应从 `wiki/index.md` 开始，然后沿相关 wiki 页面继续；当需要引用或争议信息时，再检查 IR claims。

---

## CLI

使用现有 `rag` conda 环境：

```bash
source /Users/jiachongliu/anaconda3/etc/profile.d/conda.sh
conda activate rag
pip install -e .
cw info
```

常用命令：

| 命令 | 作用 |
|---------|---------|
| `cw raw init <source_id> --title "..."` | 创建 source bundle 目录 |
| `cw raw bundle-pdf <pdf> --source-id <id> --title "..." --execute` | 把已放在 `raw/` 下的 PDF 打包为 source bundle |
| `cw index build [--overwrite]` | 从 IR 构建 `ir/index.json` 检索索引 |
| `cw extract markdown <source_id>` | 在 `ir/extracts/` 下生成规范化 Markdown 抽取件 |
| `cw extract llm-claims <source_id> --provider deepseek` | 在 `ir/candidates/` 下生成模型候选 claims |
| `cw ask "question" --limit 5` | 基于 wiki/IR 问答并显示检索引用（自动模式：若 `.env` 可用则使用 DeepSeek；若 LLM 调用失败则直接报错） |
| `cw chat` | 交互式命令行对话（与 `cw ask` 相同自动模式） |
| `cw validate` | 按 schema 校验 IR JSON |
| `cw lint` | 交叉检查 source id、claim 引用和 wiki 链接 |

---

## 当前 QA 测试结果

`cw ask` 和 `cw chat` 运行在自动模式：如果 `.env` 或 shell 环境里存在 `DEEPSEEK_API_KEY`，会基于检索引用调用 LLM 回答；如果没有 key，则使用本地检索模板回答。  
如果已配置 LLM 但 API 调用失败，命令会直接报错，而不是静默回退。  
`cw extract llm-claims` 仍是独立的 LLM 路径，用于生成候选 claims。
如果要做可重复的纯本地测试，请在当前命令里临时取消 `DEEPSEEK_API_KEY`。

```bash
env -u DEEPSEEK_API_KEY cw ask "DemoWidget standby power" --limit 2
```

检索在可用时会使用 `ir/index.json` 索引。IR 更新后请重建或刷新索引：

```bash
cw index build --overwrite
```

观测日期：2026-04-17

| 查询 | 结果 | 说明 |
|------|------|------|
| `PECNN 如何提升 MVDC 电压预测？` | 命中 4 条 Sun et al. 2024 claims | 中文 PECNN 主题查询覆盖良好 |
| `How does PECNN improve MVDC voltage prediction?` | 命中 4 条 Sun et al. 2024 claims | 英文检索质量稳定 |
| `PINN 估计了哪些参数，需要额外硬件吗？` | 命中 4 条 Kong et al. 2024 claims | 包含参数与额外硬件需求信息 |
| `Which parameters are estimated by PINN and is extra hardware required?` | 命中 4 条 Kong et al. 2024 claims | 英文检索质量稳定 |
| `What is the experimental maximum error for the three-phase inverter case?` | 命中 4 条 Kong et al. 2024 claims | 包含 `5.2%` 与 `14.7%` 误差 claim |
| `DemoWidget standby power` | 命中 3 条 demo claims | 英文实体基线查询可用 |
| `三相逆变器实验误差是多少？` | 命中 4 条 Kong et al. 2024 claims | 轻量双语 query 扩展可命中 `5.2%` / `14.7%` 误差 claim |

输出示例（节选）：
`cw ask "How does PECNN improve MVDC voltage prediction?" --limit 2`

```text
Retrieved references:
[1] c_sun_2024_004 | score=0.641 | status=supported | source=sun_2024_dc_voltage_prediction_mvdc
    evidence: ... physical relation between voltage, current, and conductance ...
[2] c_sun_2024_003 | score=0.625 | status=supported | source=sun_2024_dc_voltage_prediction_mvdc
    evidence: ... three MVDC distribution networks ...

Answer: Based on 2 retrieved claim(s):
- [1] `PECNN` embeds: the physical relation between voltage, current, and conductance ...
- [2] `PECNN evaluation` uses: three MVDC distribution networks ...
```

输出示例（节选，LLM 已配置但网络/API 不可用）：
`cw ask "Which parameters are estimated by PINN and is extra hardware required?" --limit 2`

```text
LLM call failed (APIConnectionError): Connection error.
```

输出示例（节选）：
`cw ask "这个三相逆变器实验误差是多少？" --limit 4`

```text
Retrieved references:
[1] c_kong_2024_004 | ... | source=kong_2024_pinn_lc_parameter_estimation
[2] c_kong_2024_005 | ... | source=kong_2024_pinn_lc_parameter_estimation

Answer (LLM, deepseek/deepseek-chat):
... maximum error is 5.2% for DC-link capacitance and 14.7% for AC-side inductance ...
```

## 测试截图画廊

点击任意缩略图可查看原图。

<table>
  <tr>
    <td align="center"><a href="images/1.png"><img src="images/1.png" width="420" alt="测试截图 1"></a><br>截图 1</td>
    <td align="center"><a href="images/2.png"><img src="images/2.png" width="420" alt="测试截图 2"></a><br>截图 2</td>
  </tr>
  <tr>
    <td align="center"><a href="images/3.png"><img src="images/3.png" width="420" alt="测试截图 3"></a><br>截图 3</td>
    <td align="center"><a href="images/4.png"><img src="images/4.png" width="420" alt="测试截图 4"></a><br>截图 4</td>
  </tr>
  <tr>
    <td align="center"><a href="images/5.png"><img src="images/5.png" width="420" alt="测试截图 5"></a><br>截图 5</td>
    <td align="center"><a href="images/6.png"><img src="images/6.png" width="420" alt="测试截图 6"></a><br>截图 6</td>
  </tr>
  <tr>
    <td align="center"><a href="images/7.png"><img src="images/7.png" width="420" alt="测试截图 7"></a><br>截图 7</td>
    <td align="center"><a href="images/8.png"><img src="images/8.png" width="420" alt="测试截图 8"></a><br>截图 8</td>
  </tr>
  <tr>
    <td align="center"><a href="images/9.png"><img src="images/9.png" width="420" alt="测试截图 9"></a><br>截图 9</td>
    <td align="center"><a href="images/10.png"><img src="images/10.png" width="420" alt="测试截图 10"></a><br>截图 10</td>
  </tr>
</table>
