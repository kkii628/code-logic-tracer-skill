# Installation Guide

## Download the Packaged Skill

The `.skill` file is a distributable package that can be installed in compatible AI agent environments.

### Option 1: Download from GitHub Repository

1. Clone or download this repository:
   ```bash
   git clone https://github.com/kkii628/code-logic-tracer-skill.git
   ```

2. Use the skill source files directly, or package them yourself:
   ```bash
   # The SKILL.md and references/ directory contain all the skill logic
   # Package into .skill format (zip with .skill extension)
   cd code-logic-tracer-skill
   zip -r code-logic-tracer.skill SKILL.md references/
   ```

### Option 2: Direct File Download

Contact the repository owner to obtain the pre-packaged `code-logic-tracer.skill` file.

## Skill Structure

```
code-logic-tracer/
├── SKILL.md                          # Core skill instructions
└── references/
    └── review-patterns.md            # Detailed review methodology & templates
```

## What This Skill Does

**Code Logic Tracer** provides dual-review capabilities for data processing code:

1. **Logic Review**: Independently review business logic for internal consistency before examining code
2. **Code-Logic Alignment Review**: Map each requirement to implementation, verify correctness
3. **Pseudocode Generation**: Convert code to human-readable pseudocode with data flow diagrams
4. **Processing Behavior Confirmation**: Generate confirmation tables mapping code behavior to requirements

### Trigger Conditions

Use this skill when:
- Reviewing whether code accurately implements given business/task logic
- Tracing through code line-by-line to produce pseudocode
- Verifying data processing pipelines (filtering, splitting, deduplication, transformation)
- Producing processing behavior confirmation tables

## Usage Example

```
User: "Review this code and check if it correctly implements the following logic: [logic description]"

Agent (with code-logic-tracer skill):
1. Read the logic description
2. Perform Phase A: Logic self-review
3. Read the code file
4. Perform Phase B: Code-logic alignment review
5. Output review results with priority classifications (P0/P1/P2)
```

## License

See repository license file for details.
