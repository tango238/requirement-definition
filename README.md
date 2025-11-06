# requirement-difinition

> **Note**: These commands require [Claude Code](https://claude.com/claude-code) to run.

## Commands

### `/def:check-features <path-to-features.yml>`

Validates feature specifications in YAML format and appends review results to a `check.md` file.

**Purpose**: Inspect feature specifications from logical, descriptive, and coverage perspectives, listing correction points as TODOs for human review.

**Key behaviors**:
- Checks for logical contradictions, duplications, ambiguous expressions, and missing considerations
- Creates `check.md` in the same directory as the YAML file (or appends to existing)
- Outputs issues in `# TODO` checklist format with details in `# Summary`
- Does NOT automatically rewrite the YAML file

### `/def:apply-checks <yaml-file> [check-md-file]`

Reads TODOs from `check.md` and applies checked items (marked with `[o]`) to the YAML file.

**Arguments**:
- `<yaml-file>`: Target YAML file path (required)
- `[check-md-file]`: Path to check.md (optional, defaults to `check.md` in same directory as YAML file)

**TODO Status Markers**:
- `[ ]`: Not processed yet
- `[o]`: Ready to apply (this command processes these)
- `[x]`: Already applied
- `[~]`: Skipped/declined

**Key behaviors**:
- Shows diff preview of changes before applying
- Requires user confirmation before modifying files
- Updates applied items from `[o]` to `[x]` in check.md
- Preserves YAML structure (comments, spacing, indentation)

## Workflow Example

```bash
# 1. Validate features.yml
/def:check-features ./specs/features.yml

# 2. Edit check.md to mark items for processing with [o]

# 3. Apply checked items
/def:apply-checks ./specs/features.yml

# Or specify check.md path explicitly
/def:apply-checks ./specs/features.yml ./specs/check.md
```

## Structure of features.yml

```yaml
features:
  - feature:
      name:
      description:

  - feature:
      name:
      description:
```
