## Description

A command that reads TODOs from `{{arg2}}` (check.md), and applies items marked as "scheduled for processing" to update `{{arg1}}` (YAML file).

---

## Arguments

* `{{arg1}}`: Path to the target YAML file to update (required)
* `{{arg2}}`: Path to check.md file (optional; if omitted, uses `check.md` in the same directory as `{{arg1}}`)

---

## Mission

Apply modification items (TODOs) from check.md that are marked as "scheduled for processing" to the original YAML file, then mark those TODOs as "completed" after application.

---

## Success Criteria

1. `{{arg2}}` (or check.md in the same directory) is correctly loaded
2. TODO item status (unprocessed/scheduled/completed/skipped) is correctly interpreted
3. Concrete action methods are considered for items marked as scheduled `[o]`
4. Change diff is clearly displayed before applying to `{{arg1}}`
5. After user confirmation, diff is applied to `{{arg1}}`
6. Applied items are marked as completed `[x]` in check.md

---

## TODO Status Markers

TODOs in check.md use the following status markers:

* `- [ ]`: Unprocessed (not yet addressed)
* `- [o]`: Scheduled (to be applied this time)
* `- [x]`: Completed (already applied)
* `- [~]`: Skipped (decided not to apply)

This command only processes items marked with `[o]`.

---

## Execution Steps

### 1. Load Check File

* If `{{arg2}}` is specified, load that file.
* If `{{arg2}}` is omitted, load `check.md` from the same directory as `{{arg1}}`.
* If the file doesn't exist or cannot be read, report error and exit.

### 2. Parse TODO Items

* Parse the `# TODO` section in check.md.
* Identify the status marker for each TODO item:
  * `[o]`: Extract as scheduled for processing
  * `[ ]`, `[x]`, `[~]`: Skip (not processed this time)
* If there are no scheduled `[o]` items, report "No items to process" and exit.

### 3. Load Summary Details

* Load detailed information from the `# Summary` section corresponding to each `[o]` marked TODO item.
* Extract the following information:
  * **Target**: Which YAML path/key to modify
  * **Issue**: What is the problem
  * **Recommended Action**: How to fix it
  * **Notes**: Other important considerations

### 4. Load Target YAML

* Load `{{arg1}}`.
* Parse as YAML and understand the current structure.
* If it cannot be loaded or YAML is broken, report error and exit.

### 5. Plan Modifications

* For each `[o]` TODO item, determine specific modification content:
  * Consider modification method based on Summary's "Recommended Action"
  * Specify which part of the YAML structure to change and how
  * Select the most appropriate method if multiple modification approaches exist
* Display a list of modification plans.

### 6. Show Diff Preview

* Display diff when modifications are applied:
  * Compare YAML before and after changes
  * Added lines displayed with `+ `
  * Deleted lines displayed with `- `
  * Changed lines displayed as `- ` and `+ ` pair
* Display diff in an easy-to-read format (unified diff or side-by-side).

### 7. Confirm and Apply

* **Important**: Request user confirmation:
  * "Do you want to apply this diff? (yes/no)"
  * Do not apply until user explicitly approves
* Only apply changes to `{{arg1}}` if user approves.
* After application, overwrite and save the original YAML file.

### 8. Update Check.md Status

* For each applied TODO item, update status in check.md from `[o]` to `[x]`:
  * `- [o] item name` → `- [x] item name`
* Save check.md.
* Do not change existing other items (`[ ]`, `[~]`, already `[x]` items).

### 9. Report Results

* Report processing results:
  * Number of applied items
  * Application result for each item (success/failure)
  * Updated file paths
* If problems occurred, report details.

---

## Output Examples

### Diff Display Example

```diff
# Diff for features.yml

## Change 1: Clarify preconditions for reservation feature

--- features.yml (before)
+++ features.yml (after)

 reserve:
   name: Reservation feature
-  precondition: Logged in
+  precondition: 
+    - Must be logged in
+    - Available seats must exist
   steps:
     - Select date and time
     - Select seat

## Change 2: Add cancellation feature

+cancel:
+  name: Cancellation feature
+  precondition:
+    - Reservation must exist
+  steps:
+    - Select reservation
+    - Execute cancellation
+  postcondition: Reservation will be deleted
```

### check.md Update Example

```markdown
# TODO
- [x] Clarify preconditions for reservation feature
- [x] Add cancellation feature
- [ ] Add error handling
- [~] Review overly detailed granularity
```

---

## Important Constraints

* **Always obtain user approval before applying**: Do not rewrite automatically. Display diff and only apply when user answers "yes".
* **Only process items marked `[o]`**: Ignore `[ ]`, `[x]`, `[~]`.
* **Preserve original YAML structure**: Maintain comments, blank lines, indentation, etc. as much as possible.
* **Always mark as `[x]` after application**: To prevent double application.
* **If Summary information is insufficient**: Ask user for additional information or skip that item.
* **If multiple `[o]` items exist**: Display all diffs together and apply in batch (or confirm individually).

---

## Error Scenarios & Fallback

* **If check.md doesn't exist**:
  * Report `check.md not found. Please run check-features command first to perform validation.` and exit.

* **If no `[o]` marked items**:
  * Report `No scheduled items. Please mark items as [o] in check.md.` and exit.

* **If YAML is broken**:
  * Report `YAML format in {{arg1}} is invalid. Please fix it first.` and exit.

* **If no corresponding details in Summary**:
  * Skip that item and report "Skipped due to insufficient detailed information."

* **If error occurs during application**:
  * Abort changes and preserve original state.
  * Report error details.
  * Do not update check.md (leave as `[o]`).

---

## Workflow Example

```bash
# 1. First, validate features.yml
/def:check-features ./specs/features.yml

# 2. Edit check.md and mark items to process as [o]
# (Open check.md in editor and edit)

# 3. Apply changes
/def:apply-checks ./specs/features.yml

# Or explicitly specify check.md path
/def:apply-checks ./specs/features.yml ./specs/check.md
```

---

## Additional Notes

* This command uses a **semi-automatic** approach:
  * AI proposes modification content
  * User selects items to process in check.md (`[o]` mark)
  * User reviews diff and approves
  * AI actually applies changes
* Safety is ensured by incorporating human judgment rather than full automation.
* When processing many items at once, the diff becomes large, so it's recommended to process important changes individually.