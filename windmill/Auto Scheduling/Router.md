---
title: "AA26 Router"
author: "Jacob Vendramin"
date: "2025-12-17"
last_updated: "2025-12-17"
tags: ["auto-scheduler", "router", "classification"]
version: "1.0.0"
emoji: "👥"
---

# 🚦 AA26 Router

## 🎯 Beginner's Guide: Quick Reference

### What the Router Does

The Router is the **first step** in the AA26 auto-scheduling pipeline. It analyzes incoming tasks and determines:
1. **Classification**: What type of task is this? (auto_schedule, queued, has_subtasks, etc.)
2. **Process Decision**: Should this task proceed to the main AA26 flow? (true/false)
3. **Narrative**: Human-readable explanation of the decision

### Classification Types at a Glance

| Classification | process | Meaning |
|---------------|---------|---------|
| `auto_schedule` | ✅ true | Task ready for auto-scheduling |
| `queued` | ❌ false | No room on account, task queued |
| `skip` | ❌ false | Auto-assign override is enabled |
| `process_queued_tasks` | ✅ true | Parent completed, account has room, process queued tasks |
| `queued_no_room` | ❌ false | Parent completed, has queued tasks but no room to process |
| `completed_no_queued` | ❌ false | Parent completed, no queued tasks |
| `move_subtasks` | ✅ true | Parent assigned, subtasks need scheduling |
| `move_dependent_tasks` | ✅ true | Deliverables needed, dependent tasks ready |
| `has_subtasks` | ❌ false | Has subtasks (blocking) |
| `has_dependencies` | ❌ false | Has dependent tasks (blocking) |
| `has_employee_assignees` | ❌ false | Already has employee assigned (blocking) |
| `is_subtask` | ❌ false | Task is a subtask (blocking) |
| `is_dependent_on_another` | ❌ false | Depends on another task (blocking) |
| `send_feedback_form` | ✅ true | Status is "final files delivered" |

### Quick Config Reference

| What You Want to Change | Config Key | Location |
|------------------------|------------|----------|
| Completion statuses | `router_config.statuses.completed` | sis_config table |
| "Open" status check | `router_config.statuses.needs_processing` | sis_config table |
| Deliverables trigger status | `router_config.statuses.deliverables_trigger` | sis_config table |
| Actionable classification types | `router_config.classifications.actionable` | sis_config table |
| Skip classification types | `router_config.classifications.skip` | sis_config table |
| Blocking classification types | `router_config.classifications.blocking` | sis_config table |
| HTTP timeout | `router_config.limits.http_timeout_seconds` | sis_config table |
| Narrative messages | `router_config.narratives.*` | sis_config table |

### Quick Troubleshooting

**Task classified as `queued` when it shouldn't be?**
1. Check account capacity via `get_task_auto_assign_data` RPC
2. Verify `room > 0` for the account
3. Review `aa_log` table for the task

**Task classified as blocking (`has_subtasks`, `has_dependencies`)?**
1. Check if subtasks/dependent tasks are assigned
2. Verify subtask/dependent task status is "open"
3. Check if parent task is assigned (for `move_subtasks`)

**Wrong narrative message?**
1. Check `router_config.narratives` in sis_config
2. Verify template variables ({count}, {parent}, {room}) are correct

---

## 📚 Complete Technical Guide

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        ROUTER FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐    ┌─────────────────────┐    ┌────────────────┐  │
│  │  Input   │───▶│  Parallel Fetch     │───▶│  Classification │  │
│  │ task_id  │    │  - ClickUp Task     │    │  Logic          │  │
│  └──────────┘    │  - Auto-assign Data │    └────────────────┘  │
│                  │  - Router Config    │            │            │
│                  └─────────────────────┘            ▼            │
│                                              ┌────────────────┐  │
│                                              │  Output        │  │
│                                              │  - classifications│
│                                              │  - process      │  │
│                                              │  - narrative    │  │
│                                              └────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Input**: `task_id` (ClickUp task ID)
2. **Parallel Fetches**:
   - `fetch_router_config()` - Gets config from sis_config
   - `fetch_clickup_task_full()` - Gets task with subtasks from ClickUp API
   - `fetch_task_auto_assign_data()` - Gets account capacity from RPC
3. **Classification Logic**: Applies rules based on task state
4. **Output**: Classification result with process decision and narrative

---

### Entry Point

**Path**: `f/aa26_v2/router.py`

**Function Signature**:
```python
def main(task_id: str = None, body: dict = None) -> dict
```

**Parameters**:
- `task_id`: Direct ClickUp task ID (optional if body provided)
- `body`: Dictionary containing `{"task_id": "..."}` (optional if task_id provided)

**Returns**:
```json
{
  "task_id": "86dyjhcya",
  "classifications": ["auto_schedule"],
  "process": true,
  "narrative": "Task 'Design bulletin' (86dyjhcya) with status 'open' was analyzed. Found: task is ready for auto-scheduling with no blocking conditions. Account 1234 capacity: 5/10 active tasks, room for 5 more. Decision: process=True because task is ready for auto-scheduling.",
  "account_details": {
    "account": "1234",
    "cap": 10,
    "active_tasks": 5,
    "room": 5
  }
}
```

---

### Classification Logic

#### Phase 1: Completed Parent Check

If task status is in `completed_statuses` (default: "closed", "final files delivered"):

```
task_status in completed_statuses?
    │
    ├─▶ YES: Check aa_queued_count from RPC
    │       │
    │       ├─▶ aa_queued_count > 0: Check room capacity
    │       │       │
    │       │       ├─▶ room > 0
    │       │       │       └─▶ Return: process_queued_tasks + optional send_feedback_form
    │       │       │                  (includes queued_task_ids array)
    │       │       │
    │       │       └─▶ room = 0
    │       │               └─▶ Return: queued_no_room + optional send_feedback_form
    │       │                          (process=False unless send_feedback_form applies)
    │       │
    │       └─▶ aa_queued_count = 0
    │               └─▶ Return: completed_no_queued + optional send_feedback_form
    │
    └─▶ NO: Continue to Phase 2
```

#### Phase 2: Blocking Conditions Check

```python
# Check in order:
1. has_employee_assignees  # Task already has employee assigned
2. has_subtasks / move_subtasks  # Subtask handling
3. has_dependencies / move_dependent_tasks  # Dependency handling
4. is_subtask  # Task is itself a subtask
5. is_dependent_on_another  # Task depends on another
```

#### Phase 3: Account Capacity Check

If no blocking conditions found:

```
auto_assign_override enabled?
    │
    ├─▶ YES: Return: skip (process=False)
    │
    └─▶ NO: Check room
            │
            ├─▶ room <= 0: Return: queued (process=False)
            │
            └─▶ room > 0: Return: auto_schedule (process=True)
```

#### Phase 4: Process Decision

```python
# Classification sets (from config):
actionable_classifications = {"auto_schedule", "move_dependent_tasks", "move_subtasks"}
skip_classifications = {"skip", "queued"}
blocking_classifications = {"has_employee_assignees", "has_subtasks", ...}

# Decision logic:
for classification in matching_types:
    if classification in actionable_classifications:
        process = True
        break
    elif classification in skip_classifications:
        process = False
        break
    elif classification in blocking_classifications:
        process = False

# Room override: if room <= 0, always process = False
```

---

### Helper Functions

#### `fetch_router_config()`
Fetches configuration from `sis_config` table where `name = 'router_config'`.

```python
async def fetch_router_config() -> dict:
    """Fetch router configuration from sis_config table.
    Returns merged config with defaults for any missing keys.
    """
```

#### `fetch_task_auto_assign_data(task_id)`
Calls `get_task_auto_assign_data` RPC to get account capacity info.

**Returns**: `{room, auto_assign_override, account, cap, active_tasks, aa_queued_count}`

#### `fetch_queued_task_ids(account, limit)`
Fetches queued task IDs from `aa_log` table for an account.

```python
async def fetch_queued_task_ids(account: int, limit: int = 100) -> list
```

#### `check_tasks_need_processing(tasks_dict, task_ids, required_status)`
Checks which tasks need processing based on:
- Not assigned (no assignees)
- Status matches `required_status` (configurable, default: "open")
- No due date

```python
def check_tasks_need_processing(
    tasks_dict: dict,
    task_ids: list,
    required_status: str = "open"
) -> list
```

#### `check_employee_assignees(task)`
Queries `designer_scheduling` table to verify if assignees are employees.

#### `has_subtasks(task)` / `has_dependencies(task, task_id)`
Extract subtask/dependency information from ClickUp task data.

---

<details>
<summary><h2>⚙️ Configuration Reference (sis_config)</h2></summary>

### router_config

**ID**: `746a1cf7-1791-459b-9e7d-32595e4f8888`

**Workflows**: `["f/aa26_v2/router"]`

### Full Metadata Structure

```json
{
  "statuses": {
    "completed": ["closed", "final files delivered"],
    "needs_processing": "open",
    "deliverables_trigger": "deliverables needed"
  },
  "classifications": {
    "actionable": ["auto_schedule", "move_dependent_tasks", "move_subtasks"],
    "skip": ["skip", "queued"],
    "blocking": [
      "has_employee_assignees",
      "has_subtasks",
      "has_dependencies",
      "is_subtask",
      "is_dependent_on_another"
    ]
  },
  "limits": {
    "http_timeout_seconds": 30,
    "queued_task_fetch_limit": 100
  },
  "narratives": {
    "has_employee_assignees": "task already has employee assignees",
    "has_subtasks": "task has {count} subtask(s) but none need processing",
    "move_subtasks": "parent is assigned and {count} subtask(s) need scheduling",
    "has_dependencies": "other tasks depend on this one but none need processing",
    "move_dependent_tasks": "status is 'deliverables needed' and {count} dependent task(s) need scheduling",
    "is_subtask": "task is a subtask (parent: {parent})",
    "is_dependent_on_another": "task depends on another task that must complete first",
    "auto_schedule": "task is ready for auto-scheduling with no blocking conditions",
    "queued": "task should be queued because account has no room",
    "skip": "auto-assign override is enabled for this account",
    "decision_process_true_auto": "Decision: process=True because task is ready for auto-scheduling.",
    "decision_process_true_subtasks": "Decision: process=True because subtasks need to be scheduled.",
    "decision_process_true_deps": "Decision: process=True because dependent tasks need to be scheduled.",
    "decision_process_false_skip": "Decision: process=False because auto-assign override is enabled.",
    "decision_process_false_queued": "Decision: process=False because task is queued (no room available).",
    "decision_process_false_blocking": "Decision: process=False because blocking conditions exist that prevent auto-scheduling.",
    "decision_room_override": "Decision: process=False because account has no room (room={room}), even though task would otherwise be processed."
  }
}
```

### Configuration Keys Explained

#### statuses

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `completed` | array | `["closed", "final files delivered"]` | Statuses that trigger completed parent logic |
| `needs_processing` | string | `"open"` | Status required for subtasks/dependent tasks to be processed |
| `deliverables_trigger` | string | `"deliverables needed"` | Status that triggers `move_dependent_tasks` |

#### classifications

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `actionable` | array | `["auto_schedule", "move_dependent_tasks", "move_subtasks"]` | Types that set `process=True` |
| `skip` | array | `["skip", "queued"]` | Types that halt processing |
| `blocking` | array | `["has_employee_assignees", ...]` | Types indicating blocking conditions |

#### limits

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `http_timeout_seconds` | int | `30` | HTTP client timeout for ClickUp API calls |
| `queued_task_fetch_limit` | int | `100` | Maximum queued tasks to fetch (safety cap) |

#### narratives

Template strings supporting these variables:
- `{count}` - Number of subtasks or dependent tasks
- `{parent}` - Parent task ID (for subtask narrative)
- `{room}` - Available room on account

</details>

---

<details>
<summary><h2>🗄️ Database Dependencies</h2></summary>

### Tables Read

| Table | Purpose |
|-------|---------|
| `sis_config` | Fetch `router_config` configuration |
| `aa_log` | Fetch queued task IDs for account |
| `designer_scheduling` | Verify if assignees are employees |

### RPC Functions Called

| RPC | Purpose |
|-----|---------|
| `get_task_auto_assign_data` | Get account capacity (room, cap, active_tasks, aa_queued_count) |

### External APIs Called

| API | Endpoint | Purpose |
|-----|----------|---------|
| ClickUp | `GET /task/{task_id}?include_subtasks=true` | Fetch task with subtasks and dependencies |

</details>

---

<details>
<summary><h2>🚀 API Usage Examples</h2></summary>

### Basic Task Classification
```bash
curl -X POST https://api.windmill.dev/w/sis/jobs/run/f/aa26_v2/router \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"task_id": "86dyjhcya"}'
```

### Using Body Parameter
```bash
curl -X POST https://api.windmill.dev/w/sis/jobs/run/f/aa26_v2/router \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"body": {"task_id": "86dyjhcya"}}'
```

### Example Response: auto_schedule
```json
{
  "task_id": "86dyjhcya",
  "classifications": ["auto_schedule"],
  "process": true,
  "narrative": "Task 'Design bulletin' (86dyjhcya) with status 'open' was analyzed. Found: task is ready for auto-scheduling with no blocking conditions. Account 1234 capacity: 5/10 active tasks, room for 5 more. Decision: process=True because task is ready for auto-scheduling.",
  "account_details": {
    "account": "1234",
    "cap": 10,
    "active_tasks": 5,
    "room": 5
  }
}
```

### Example Response: queued
```json
{
  "task_id": "86dyjhcya",
  "classifications": ["queued"],
  "process": false,
  "narrative": "Task 'Design bulletin' (86dyjhcya) with status 'open' was analyzed. Found: task should be queued because account has no room. Account 1234 capacity: 10/10 active tasks, room for 0 more. Decision: process=False because task is queued (no room available).",
  "account_details": {
    "account": "1234",
    "cap": 10,
    "active_tasks": 10,
    "room": 0
  }
}
```

### Example Response: process_queued_tasks
```json
{
  "task_id": "86dyjhcya",
  "classifications": ["process_queued_tasks", "send_feedback_form"],
  "process": true,
  "narrative": "Task 86dyjhcya has status 'final files delivered' indicating completion. Account 1234 has 3 task(s) in the queue waiting to be processed. Account capacity check: room=5, so 3 queued task(s) can be processed now. Status is 'final files delivered' so a feedback form should be sent to the client. Decision: process=True because queued tasks need scheduling and/or feedback form needs sending.",
  "queued_task_ids": ["86abc123", "86def456", "86ghi789"],
  "account_details": {
    "account": "1234",
    "cap": 10,
    "active_tasks": 5,
    "room": 5,
    "aa_queued_count": 3
  }
}
```

### Example Response: move_subtasks
```json
{
  "task_id": "86dyjhcya",
  "classifications": ["move_subtasks"],
  "process": true,
  "narrative": "Task 'Design package' (86dyjhcya) with status 'in progress' was analyzed. Found: parent is assigned and 2 subtask(s) need scheduling. Account 1234 capacity: 5/10 active tasks, room for 5 more. Decision: process=True because subtasks need to be scheduled.",
  "subtask_ids": ["86sub001", "86sub002"],
  "account_details": {
    "account": "1234",
    "cap": 10,
    "active_tasks": 5,
    "room": 5
  }
}
```

</details>

---

<details>
<summary><h2>🎓 Common Scenarios</h2></summary>

### Scenario 1: Add a New Completion Status

**Goal**: Treat "archived" as a completion status

**Steps**:
```sql
UPDATE sis_config
SET metadata = jsonb_set(
  metadata,
  '{statuses,completed}',
  '["closed", "final files delivered", "archived"]'
)
WHERE name = 'router_config';
```

No code changes needed.

---

### Scenario 2: Change the Processing Status

**Goal**: Use "ready" instead of "open" for subtask processing

**Steps**:
```sql
UPDATE sis_config
SET metadata = jsonb_set(
  metadata,
  '{statuses,needs_processing}',
  '"ready"'
)
WHERE name = 'router_config';
```

---

### Scenario 3: Add a New Actionable Classification

**Goal**: Add `priority_schedule` as an actionable type

**Steps**:
1. Update config:
```sql
UPDATE sis_config
SET metadata = jsonb_set(
  metadata,
  '{classifications,actionable}',
  '["auto_schedule", "move_dependent_tasks", "move_subtasks", "priority_schedule"]'
)
WHERE name = 'router_config';
```

2. Add logic in router.py to detect and add `priority_schedule` classification

---

### Scenario 4: Customize Narrative Messages

**Goal**: Change the "queued" message

**Steps**:
```sql
UPDATE sis_config
SET metadata = jsonb_set(
  metadata,
  '{narratives,queued}',
  '"task has been added to the queue - account is at capacity"'
)
WHERE name = 'router_config';
```

---

### Scenario 5: Increase HTTP Timeout

**Goal**: Handle slow ClickUp API responses

**Steps**:
```sql
UPDATE sis_config
SET metadata = jsonb_set(
  metadata,
  '{limits,http_timeout_seconds}',
  '60'
)
WHERE name = 'router_config';
```

---

### Scenario 6: Debug Classification Issues

**Steps**:
1. Run the router with the task ID
2. Check the `narrative` field in the response for explanation
3. Check `classifications` array for all matched types
4. Verify `account_details` for capacity info
5. If `process=False` unexpectedly, check:
   - Is task in blocking classifications?
   - Is room = 0?
   - Is auto_assign_override enabled?

</details>

---

<details>
<summary><h2>📊 Performance Optimization</h2></summary>

### Parallel Fetching

The router fetches config first (to get timeout), then fetches ClickUp task and auto-assign data in parallel:

```python
# Config fetched first (needed for timeout value)
router_config = await fetch_router_config()

# Then parallel fetch with configured timeout
timeout = aiohttp.ClientTimeout(total=http_timeout)
async with aiohttp.ClientSession(timeout=timeout) as session:
    task, auto_assign_data = await asyncio.gather(
        fetch_clickup_task_full(session, task_id),
        fetch_task_auto_assign_data(task_id),
        return_exceptions=True
    )
```

**Benefit**: Reduces total execution time by ~40%

### Fail-Safe Defaults

All config values have embedded defaults in `DEFAULT_CONFIG`:

```python
DEFAULT_CONFIG = {
    "statuses": {...},
    "classifications": {...},
    "limits": {...},
    "narratives": {...}
}
```

**Benefit**: Router continues to function even if sis_config fetch fails

### Batch Subtask Fetching

When checking subtasks/dependent tasks, they're fetched in parallel:

```python
related_tasks = await fetch_multiple_tasks(session, all_related_ids)
```

**Benefit**: Single batch request instead of N sequential requests

</details>

---

<details>
<summary><h2>🔗 Related Documentation</h2></summary>

- [AA26 Main Documentation](./AA26.md) - Complete AA26 workflow reference
- `get_task_auto_assign_data` RPC - Account capacity data source
- `aa_log` table - Queue management and execution history

</details>
