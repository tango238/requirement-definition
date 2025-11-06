# check-features

Validate feature specifications in YAML format and append review results to a check.md file.

## Usage

```bash
/check-features <path-to-features.yml>
```

## Arguments

- `<path-to-features.yml>`: Path to the YAML file containing feature specifications (required)

## Mission

Read the feature specifications from the specified YAML file, inspect them from logical, descriptive, and coverage perspectives, and list points that need correction in Markdown format. The purpose is NOT to "let AI rewrite the specifications arbitrarily" but to "leave correction candidates as TODOs so humans can fix them later."

## Success Criteria

1. The entire content of the specified YAML file has been read
2. Logical contradictions, duplications, ambiguous expressions, deficiencies, and overlooked considerations are explained with reasons why they are problems
3. If `check.md` doesn't exist in the same directory as the YAML file, it is newly created
4. If `check.md` already exists, **only newly found issues** are appended without destroying existing content
5. All issues are output in a `# TODO` checklist and corresponding details in `# Summary`
6. The YAML file is NOT automatically rewritten (automatic corrections are NG; only reporting is OK)

## Execution Steps

### 1. Load Context

- Read the specified YAML file
- Interpret it as YAML and understand the top-level structure (list of features, scenarios, conditions, notes, etc.)
- If it cannot be read, report that and exit

### 2. Logical Consistency Check

- Check if preconditions are defined after results, or if preconditions and actions contradict
- Verify that different meanings aren't defined for the same identifier or feature name
- If parent-child relationships (e.g., feature → scenario → step) are broken, identify the position and reason
- If contradictions are found, write the key (path) where they occur as a "YAML path" if possible (e.g., `features.reserve.precondition[1]`)

### 3. Ambiguity & Duplication Check

- Point out phrases that cause implementation judgment to vary, such as "depending on the case," "etc.," "as appropriate," "basically"
- If there are multiple features/scenarios with similar names that should be consolidated for easier maintenance, point that out
- Write which sentence is ambiguous with the original text (or excerpt)

### 4. Improvement Points

- List items with too coarse/fine granularity
- Check if perspectives that are "easy to miss later" such as roles, permissions, error cases, boundary values are missing
- If there are missing perspectives, write them as "recommended additions"

### 5. Coverage & Missing Considerations

- Check if not only success paths but also errors, exceptions, cancellations, updates, deletions are needed
- If there are dependencies on other features but those dependencies aren't described in the YAML, write "missing references"
- If there are improvement suggestions, write them as "proposals," but **do NOT rewrite the actual YAML file**

### 6. Prepare Output File

- The output destination is always `check.md` in the same directory as the input YAML file
- If the file doesn't exist, create it and write the Markdown template described below as initial content
- If it exists, parse the existing `# TODO` and `# Summary` sections, and **treat identical or nearly identical issues as duplicates; don't rewrite them, just report "was duplicate"**

### 7. Diff-like Update

- Append only "newly detected items this time" to the end of `# TODO`
- When appending, strictly use checkbox format `- [ ] ...`
- For each appended TODO, add detailed explanation as a subheading under `# Summary`
- Do NOT rewrite or delete existing parts of the actual file

### 8. Report Changes

- List all added items and clearly separate "this item was newly detected and appended" from "this item was duplicate with existing and not appended"
- Here, **NOT "arbitrarily change" but only report "judged that change is necessary"**

## Output Format

`check.md` should always have the following structure. If an existing file doesn't have this structure, append according to this structure (don't delete existing content):

```markdown
# TODO
- [ ] {{short name of correction item 1}}
- [ ] {{short name of correction item 2}}
- [ ] {{short name of correction item 3}}
<!-- Continue adding newly found items -->

# Summary
## {{short name of correction item 1}}
- Target: {{path or key name in the YAML file}}
- Issue: {{specifically what the problem is}}
- Recommended action: {{how to fix it}}
- Notes: {{if any}}

## {{short name of correction item 2}}
... (continue similarly)
```

- "Short name" should be about 20-60 characters readable in the TODO list
- Summary side can be bullet points, but make it clear which YAML element the issue refers to

## Important Constraints

- Do NOT rewrite the YAML file. **Only report** parts that need rewriting
- When appending to check.md, do NOT delete original headings or lists
- If content is almost the same as existing TODO, don't add it as new; report "similar existing item found"
- If YAML interpretation is ambiguous, make the ambiguity itself a TODO item (e.g., "structure is unclear, recommend split description")

## Error Scenarios & Fallback

- **If the YAML file doesn't exist or can't be read**:
  - Output "Could not validate because the specified YAML file does not exist" and don't create check.md

- **If the YAML file is broken**:
  - Write which line/element was unparseable in Summary and add one TODO item for "YAML formatting"

- **If check.md is broken Markdown**:
  - Without deleting existing content, add new `# TODO` and `# Summary` at the end of the file and append there

## Instructions

Please execute the above specification for the YAML file specified as an argument.
