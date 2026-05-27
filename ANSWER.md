# Issue Management Automation Workflow

An intelligent, robust, and fully-automated Issue Management system has been successfully implemented and deployed for this Node.js project.

## Workflow Overview (`.github/workflows/issue-automation.yml`)

The automation is configured as a GitHub Actions workflow that responds to `issues` events (specifically `opened` and `labeled`). To prevent any infinite loops from automated labeling, the workflow includes robust checks to ignore actions triggered by any bot account (e.g. `github-actions[bot]`).

The workflow consists of three major jobs with explicit dependencies:

### 1. **issue-triage** Job
- **Purpose**: Automatically categorizes and prioritizes issues upon creation or labeling.
- **Categorization**: Inspects the issue title (case-insensitive) for keywords:
  - Title contains `"bug"` → Adds `bug` label
  - Title contains `"epic"` → Adds `epic` label
  - Title contains `"maintenance"` → Adds `maintenance` label
- **Prioritization**: Inspects both the issue title and body (case-insensitive) to assign priority labels. If multiple keywords are present, the highest priority wins:
  - `"critical"`, `"urgent"`, `"production"`, `"outage"` → `priority-critical`
  - `"important"`, `"high"`, `"blocking"` → `priority-high`
  - `"medium"`, `"normal"` or if no keywords found → `priority-medium` (default)
  - `"low"`, `"nice-to-have"`, `"minor"` → `priority-low`
- **Initial Status**: Ensures every new issue receives the `needs-triage` label.
- **Label Creation**: Automatically checks for and creates any of the required category, priority, or status labels if they do not already exist on the repository.

### 2. **task-breakdown** Job
- **Purpose**: Generates subtask issues for parent issues that represent an "Epic".
- **Condition**: Triggers when the parent issue's title contains `"Epic"` (case-insensitive).
- **Execution**:
  - Automatically creates exactly 4 sub-issues following the pattern: `[SUBTASK] [Original Title] - Task N: [Task Name]`
  - Task Names:
    1. `Requirements Analysis`
    2. `Design and Architecture`
    3. `Implementation`
    4. `Testing and Documentation`
  - Links each sub-issue back to the parent by putting `"Related to #[parent-number]"` in the sub-issue body.
  - Automatically tags all sub-issues with the `enhancement` and `needs-review` labels.
  - Updates the parent issue's body dynamically with a `"## Epic Tasks"` markdown checklist containing links to the newly created sub-issues (e.g., `- [ ] #<number> Requirements Analysis`).

### 3. **auto-response** Job
- **Purpose**: Engages with contributors and routes issues based on metadata.
- **First-Time Contributor Greeting**:
  - Checks if the issue author has previously created an issue in this repository.
  - If it is their first time, adds the `first-time-contributor` label and comments a welcoming message greeting them to the project.
- **Guidelines Comments**: Posts relevant process guidelines based on the issue type:
  - `bug` issues: Comment contains `"Bug Report Guidelines"`
  - `epic` issues: Comment contains `"Feature Request Process"`
  - `maintenance` issues: Comment contains `"Maintenance Guidelines"`
- **Milestone Assignment**: Sets the milestone to `"v1.0.0"` for all `priority-high` and `priority-critical` issues, automatically creating the milestone if it doesn't already exist.
- **Status Update**: Removes the initial `needs-triage` label and adds the `needs-review` label to indicate the issue is ready for maintainer review.

---

## Supporting Files Created

1. **Bug Report Template** (`.github/ISSUE_TEMPLATE/bug_report.md`): Standardized template to collect bug reports.
2. **Feature Request Template** (`.github/ISSUE_TEMPLATE/feature_request.md`): Standardized template for feature proposals/epics.
3. **Maintenance Report Template** (`.github/ISSUE_TEMPLATE/maintenance_report.md`): Standardized template for maintenance tasks.

---

## Verified Test Cases

The system was extensively tested with the following real issues:

1. **Bug Issue (#2)**: *"Bug: Login form validation not working"*
   - **Triage Result**: Assigned `bug` and `priority-high`.
   - **Auto-Response**: Posted a comment with "Bug Report Guidelines".
   - **Milestone**: Automatically assigned to milestone `v1.0.0`.
   - **Status**: Successfully transitioned from `needs-triage` to `needs-review`.

2. **Epic Issue (#3)**: *"Epic: Redesign user dashboard interface"*
   - **Triage Result**: Assigned `epic` and `priority-high`.
   - **Task Breakdown**: Created 4 sub-issues (#5, #6, #7, #8) linked back to parent #3.
   - **Checklist**: Parent issue body updated with a `"## Epic Tasks"` checklist.
   - **Auto-Response**: Posted a comment with "Feature Request Process".
   - **Milestone**: Automatically assigned to milestone `v1.0.0`.
   - **Status**: Transitioned to `needs-review`.

3. **Maintenance Issue (#4)**: *"Weekly maintenance cleanup and refactor"*
   - **Triage Result**: Assigned `maintenance` and `priority-medium`.
   - **Auto-Response**: Posted a comment with "Maintenance Guidelines".
   - **Milestone**: Left unassigned (since priority is medium).
   - **Status**: Transitioned from `needs-triage` to `needs-review`.
