---
name: code-logic-tracer
description: Trace and analyze code logic by performing dual reviews (logic correctness + code-logic alignment) and generating pseudocode with data flow diagrams. Use when the user uploads code files and asks to (1) review whether the code accurately implements given business/task logic, (2) trace through code line-by-line and produce pseudocode, (3) verify data processing pipelines for correctness in filtering, splitting, deduplication, or transformation logic, or (4) produce a processing behavior confirmation table mapping code behavior to requirements.
---

# Code Logic Tracer

Trace code execution, validate logic against requirements, and generate human-readable pseudocode and data flow diagrams.

## Workflow Decision Tree

```
User request
│
├─ 提供代码文件 + 要求"review逻辑/审查代码"
│   └── 进入 [双重Review流程]
│
├─ 提供代码文件 + 要求"逐行解析/梳理伪代码"
│   └── 进入 [逐行解析与伪代码生成流程]
│
└─ 同时要求review + 解析
    └── 先执行 [双重Review流程]，再执行 [逐行解析与伪代码生成流程]
```

## Step 1: 读取输入

1. 读取用户上传的所有代码文件
2. 读取用户提供的业务/任务逻辑描述（如有）
3. 识别代码语言和涉及的框架/库

## Step 2: 双重Review流程

### Phase A: 逻辑审查（先审逻辑本身）

独立审查用户提供的业务逻辑描述，判断其本身是否正确：

- **边界条件检查**：识别所有if/else分支，逐个验证边界处理是否自洽
- **数据依赖检查**：上游输出的字段/格式是否能被下游步骤正确消费
- **口径一致性**：不同时期/分组的数据口径是否一致
- **潜在矛盾**：识别文字逻辑中的自相矛盾（如要求遍历聚合表但输出明细字段）

输出：逻辑审查结论（✅正确 / ⚠️需修正 / ❌存在矛盾），并标注具体问题位置和修正建议。

### Phase B: 代码-逻辑比对审查

逐项比对代码实现与逻辑要求：

| 审查维度 | 检查项 |
|---------|--------|
| **准确性** | 每个步骤的实现是否与逻辑要求一致 |
| **完整性** | 是否覆盖了逻辑要求中的所有步骤，有无遗漏 |
| **系统性** | 文件处理是否完整覆盖，流程是否闭环 |
| **有效性** | 是否会产生静默错误（不报错的错误结果） |
| **格式转换** | 类型转换、编码处理是否正确 |
| **内存优化** | 是否采用了合理的分批/分片/只读策略 |
| **字段清洗** | strip、空值过滤等清洗策略是否会导致有效数据丢失 |

输出：逐项审查表格，标注 ✅/⚠️/❌ 及说明。

### 审查原则

- 区分"逻辑矛盾"和"代码未实现"：逻辑矛盾是用户描述本身的问题，代码未实现是代码的问题
- 区分"形式差异"和"实质错误"：代码形式与描述不完全一致但结果等价，标注为形式差异
- 标明优先级：P0（必须修改）、P1（建议修改）、P2（可选优化）

## Step 3: 逐行解析与伪代码生成流程

### 3.1 函数级解析

对每个函数按以下结构解析：

```
### `function_name(params)`（行号范围）

输入: 参数说明
输出: 返回值说明
行为:
  步骤1: ...
  步骤2: ...
  ...
关键逻辑: 核心判断条件或算法说明
```

### 3.2 全局配置解析

列出所有常量、路径配置、字段名常量，说明其含义。

### 3.3 主流程伪代码

用缩进伪代码描述主入口的执行顺序，包括：
- 文件收集循环
- 逐个文件处理流程
- 异常处理分支
- 最终汇总输出

### 3.4 数据流图

用文字图表描述从输入到输出的完整数据流转：

```
输入目录
  │
  ▼
[步骤1函数] ──► 中间结果
  │
  ▼
[步骤2函数] ──► 中间结果
  │
  ▼
输出文件
```

### 3.5 处理行为确认表

以表格形式列出每个处理步骤的行为与用户要求的映射关系：

| 步骤 | 代码行为 | 用户要求 | 结论 |
|------|---------|---------|------|
| 读取 | 所有字段强制str | 要求按字符串读取 | ✅ 符合 |
| 筛选 | 仅保留发明申请/发明授权 | 要求保留发明类 | ✅ 符合 |

## Step 4: 输出格式规范

### Review输出格式

使用分层审查结论：

1. **总体结论**：一句话概括（通过/需修改/有问题）
2. **严重问题（🔴）**：会导致结果错误的P0问题
3. **中等问题（🟡）**：与规范不一致但不影响结果
4. **形式差异（🟡/🟢）**：策略选择差异，非错误
5. **正确实现（🟢）**：逐项确认正确的功能点
6. **修改建议优先级表**：P0/P1/P2分级

### 伪代码输出格式

按以下顺序输出：
1. 全局配置列表
2. 工具函数伪代码（按依赖顺序）
3. 核心处理函数伪代码
4. 主入口伪代码
5. 整体数据流图
6. 处理行为确认表

每个函数必须标注对应的代码行号范围，方便用户回溯到原始代码。
