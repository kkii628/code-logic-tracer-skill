# Review Patterns and Templates

Detailed methodology for dual-review and pseudocode generation. Read this when performing a code logic review task.

## Table of Contents

- [1. Dual-Review Methodology](#1-dual-review-methodology)
- [2. Data Processing Checklist](#2-data-processing-checklist)
- [3. Pseudocode Templates](#3-pseudocode-templates)
- [4. Priority Definitions](#4-priority-definitions)
- [5. Common Pitfalls](#5-common-pitfalls)

---

## 1. Dual-Review Methodology

### Phase A: Logic Self-Review

Before examining the code, independently review the user's stated logic for internal consistency.

Checklist:

1. **Branch completeness**: Enumerate all if/else conditions. Is every combination handled?
2. **Data flow continuity**: Can each step consume the output of the previous step?
3. **Terminology consistency**: Are field names, file names, and constants used consistently?
4. **Scope conflicts**: Are date ranges, ID ranges, or filter conditions overlapping or conflicting?
5. **Output-input alignment**: Does the final output format match what downstream steps expect?
6. **Contradictions**: Does the logic ask for X in one place and not-X in another?

If a contradiction is found, flag it immediately. Do not blame the code for a logic contradiction.

### Phase B: Code-Logic Alignment Review

Map each requirement to the code implementation.

For each requirement:
1. Find the exact line(s) of code that implement it
2. Verify the implementation matches the requirement semantically
3. Check for off-by-one errors, empty-value handling, edge cases
4. Note if the implementation exceeds the requirement (enhancement) or falls short (gap)

### Distinguishing Issue Types

| Issue Type | Definition | Example |
|-----------|-----------|---------|
| Logic contradiction | User's stated logic is self-contradictory | "Iterate the summary table" but "output application number" |
| Implementation gap | Code does not implement a stated requirement | Missing step 4 entirely |
| Form difference | Code uses a different approach but achieves the same result | Using merge instead of dict lookup |
| Silent error | Code produces wrong results without raising errors | Parsing all values as strings when some are Excel serial dates |
| Enhancement | Code does more than required | Additional validation or logging |

## 2. Data Processing Checklist

Use this checklist when reviewing data processing pipelines.

### File I/O

- [ ] Input path exists and is accessible
- [ ] Output directory is created if not exists
- [ ] File encoding is correct (utf-8-sig for Excel-readable CSV)
- [ ] Temporary files (~$) are skipped
- [ ] File type validation (extension check)

### Reading Strategy

- [ ] dtype=str applied to prevent type inference errors
- [ ] usecols used when only specific columns are needed (memory optimization)
- [ ] Sheet index correctly mapped (e.g., upstream=1, mid/downstream=0)
- [ ] Empty files or missing files handled gracefully

### Filtering

- [ ] Filter conditions match requirements exactly
- [ ] AND vs OR logic is correct (& vs |)
- [ ] Empty/NaN/None handling is consistent
- [ ] Case sensitivity is handled (lower() or exact match as required)
- [ ] Whitespace stripped before comparison

### Splitting/Delimiting

- [ ] Correct delimiter used (e.g., semicolon vs comma)
- [ ] Empty parts handled (skip, not include)
- [ ] Deduplication within a single row (preserve order)
- [ ] Maximum split count correctly calculated

### Deduplication

- [ ] Deduplication key matches requirement
- [ ] Keep-first vs keep-last vs keep-all strategy is correct
- [ ] Tie-breaking rule specified and implemented
- [ ] Cross-file deduplication handled if required

### Date Normalization

- [ ] Excel serial dates handled (origin=1899-12-30)
- [ ] Multiple string formats parsed (YYYY-MM-DD, YYYY/MM/DD, Chinese format, etc.)
- [ ] Invalid dates handled gracefully (NaT or empty string)
- [ ] Output format matches requirement (YYYY/MM/DD)

### Aggregation

- [ ] Group-by keys match requirement
- [ ] Aggregation function matches requirement (count vs nunique vs sum)
- [ ] Empty groups handled

## 3. Pseudocode Templates

### Function Template

```
### `function_name(param1, param2)` (L{start}-{end})

输入: param1 = ..., param2 = ...
输出: ...
行为:
  1. 步骤1
  2. 步骤2
  ...
关键逻辑: 核心判断条件或特殊处理
边界处理: 空值、异常等
```

### Main Entry Template

```
## 主入口 main() (L{start}-{end})

1. 打印日志信息
2. 验证输入目录存在
3. 创建输出目录
4. 收集文件列表
5. 对每个文件循环:
   ├── 调用 process_file()
   ├── 成功: success_count += 1
   └── 失败: 打印错误
6. 打印汇总统计
```

### Data Flow Template

```
输入: {source_path}
  │
  ▼
[{function_1}] ──► {intermediate_1}
  │
  ▼
[{function_2}] ──► {intermediate_2}
  │
  ▼
输出: {output_path}
```

### Processing Behavior Table Template

```
| 步骤 | 代码行为 | 逻辑要求 | 结论 |
|------|---------|---------|------|
| 读取 | ... | ... | ✅/⚠️/❌ |
| 筛选 | ... | ... | ✅/⚠️/❌ |
| 拆分 | ... | ... | ✅/⚠️/❌ |
| 保存 | ... | ... | ✅/⚠️/❌ |
```

## 4. Priority Definitions

| Priority | Color | Action Required | Definition |
|----------|-------|-----------------|------------|
| P0 | 🔴 | Must fix before execution | Results in incorrect output or silent errors |
| P1 | 🟡 | Should fix for completeness | Missing output files, inconsistent naming, etc. |
| P2 | 🟢 | Nice to have | Code style, additional diagnostics, performance |

## 5. Common Pitfalls

### Silent Error Patterns

1. **Hardcoded delimiter**: `SPLIT_SEPARATOR = ";"` when data may contain other delimiters
2. **Type inference**: `pd.read_excel()` without `dtype=str` causing numbers to be parsed as float
3. **Empty string vs NaN**: `== ""` does not catch `NaN`, need `pd.isna()` check
4. **Case sensitivity**: File matching or string comparison without `.lower()` or `.upper()`
5. **Encoding issues**: CSV saved as utf-8 instead of utf-8-sig, causing Excel garbled text
6. **Index misalignment**: Using `df.index` after filtering without resetting, causing `loc`/`iloc` mismatch

### Logic Contradiction Patterns

1. **Aggregation vs detail**: Asking to iterate a summary table but output record-level fields
2. **Multiple delimiters**: Detecting multiple delimiters but only using one for splitting
3. **Key field inconsistency**: Step N requires a field that Step N-1 does not produce
4. **Scope mismatch**: Filter criteria in one step exclude data needed by another step
