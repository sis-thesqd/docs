# Scheduling Page Options

## Overview
**Workflow ID:** 3zd8vPa6fydv3P5h  
**Status:** Active  
**Created:** July 16, 2025  
**Last Updated:** December 16, 2025

## Purpose
Provides intelligent designer scheduling options by analyzing historical data, current workload, designer preferences, and availability to return optimal assignment choices.

## Trigger
**Type:** Webhook  
**Method:** GET  
**Path:** `/webhook/scheduling-options`  
**URL:** https://prf.thesqd.com/webhook/scheduling-options  
**Response Mode:** Last node (returns JSON)

## Workflow Process

### 1. Receive Schedule Request
Query parameters:
- `memberId`: Account member number
- `projectTypeId`: Project type to schedule

### 2. Get Account Details
- Fetches account from Supabase `accounts` table
- Retrieves:
  - Church name
  - High usage flag
  - Account number

### 3. Get Project Configuration
- Retrieves project type from `prf_selection_types`
- Gets associated tag and configuration

### 4. Get Tag Configuration
- Fetches tag config from `clickup_tags` table
- Includes:
  - Department
  - Grouping
  - Time estimation settings
  - Days future limits (min/max)

### 5. Calculate Time Estimate
Uses historical data from `get_tag_time_analysis_v2`:
- **If task count < 3:** Uses ALL ACCOUNTS average or min/max average
- **If task count ≥ 3:** Uses account-specific historical estimate
- Rounds to nearest 5 minutes
- Falls back to 30 minutes if no data

### 6. Format Search Parameters
Creates object with:
- Time estimate
- Project type/grouping
- High usage flag
- Account link
- Department
- Min/max days future
- Tag name

### 7. Parallel Data Retrieval
Simultaneously queries:

#### a. Available Designers
- RPC: `get_available_designers_optimized_v3`
- Filters by:
  - Department
  - Date range (min/max days)
  - Minimum time available
- Returns up to 50 options

#### b. Workload by Type
- RPC: `get_assignee_workload_by_project_type`
- Gets task count per designer for project type
- Within date range

#### c. Designer Preferences
- RPC: `get_designer_preferences_by_account`
- Account-specific designer preferences

#### d. Active Task Count
- RPC: `get_account_active_squad_tasks`
- Current active tasks per designer

#### e. Execution Data
- Saves grouping and account for tracking

### 8. Find Frequent Assignees
- Analyzes current active tasks
- Identifies designers with 3+ tasks for this account
- Extracts assignee IDs and names

### 9. Merge All Designer Data
Complex SQL merge combining:
- Designer availability (time windows)
- Project type workload
- Account preferences
- Previous designer relationship
- Active task assignments

Calculates:
- **Preference score:**
  - 4: Preferred + Recommended
  - 3: Recommended only
  - 2: Preferred only
  - 1: Neutral
  - 0: Not preferred/recommended
- **Previous designer flag:** 1 if worked with account before
- **Workload exclusions:** Filters out invalid assignments

Filters out:
- Designers with preference_score = 0
- Designers with active tasks (unless days_out > min + 1)

### 10. Generate Three Option Sets

#### Option 1: Fastest
- Designers available soonest
- Sorted by days_out ascending
- Limited to 1 option

#### Option 2: Plan Ahead
- Designers further out (more time)
- Sorted by days_out descending
- Limited to top 20 pool
- Randomized for variety
- Returns 1 option

#### Option 3: Preferred
- Designers with preference_score ≥ 2
- Only if not high_usage account
- Sorted by:
  - Active tasks (ascending)
  - Days out (descending)
- Returns 1 option

### 11. Format Response
Returns JSON with:
```json
{
  "fastest": { ...designer data... },
  "planAhead": { ...designer data... },
  "preferred": { ...designer data... },
  "final_estimate": 45,
  "high_usage": false
}
```

## Designer Data Fields
Each option includes:
- `user_id`: ClickUp user ID
- `username`: Designer name
- `draft_date`: Projected completion date
- `sched`: Total scheduled minutes
- `minutes_day`: Minutes per day
- `capacity`: Available capacity
- `time_avail`: Time available
- `days_out`: Days until availability
- `project_type`: Task count for this type
- `preference_score`: Preference level (0-4)
- `prev_des`: Previous designer flag
- `task_ids`: Current task IDs
- `active_tasks_assigned`: Active task count

## Key Features
- **AI-Powered Scheduling:** Data-driven designer selection
- **Three-Tier Options:** Balanced, varied, and preferred choices
- **Historical Analysis:** Learns from past projects
- **Workload Balancing:** Considers current assignments
- **Preference Integration:** Respects account-designer preferences
- **Capacity Management:** Accounts for time availability

## Database Functions (RPC)

### get_tag_time_analysis_v2
Analyzes historical time data for tags

### get_available_designers_optimized_v3
Finds designers with capacity in date range

### get_assignee_workload_by_project_type
Gets current workload per designer per type

### get_designer_preferences_by_account
Account-specific designer preferences

### get_account_active_squad_tasks
Current active tasks for account

## Error Handling
- **Continue on Error:** Designer preferences query
- **Retry Logic:** Historical data queries (5 retries, 5-second wait)
- **Always Output Data:** Ensures response even with missing data

## Preference System
- **preferred**: Designer prefers this account
- **recommended**: Account recommends this designer
- **not_preferred**: Designer doesn't prefer account
- **not_recommended**: Account doesn't recommend designer

## Notes
- Highly optimized for performance
- Complex SQL logic for accurate scoring
- Randomization prevents always picking same "best" designer
- High usage accounts skip preferred filtering
- Execution data enables tracking/analytics
- Pushes date range when needed for availability

