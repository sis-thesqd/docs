# task-analytics

Provides comprehensive analytics on task performance metrics across different departments and time periods.

## Overview
**Function Slug:** task-analytics  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 12

## Purpose
Aggregates and returns analytics data for task workflow metrics including creation-to-due-date times, status transition times, and deliverables processing times. Data is available overall and broken down by department for both 30-day and 90-day periods.

## Endpoint

```
GET https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/task-analytics
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X GET \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/task-analytics \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/task-analytics', {
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(data)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "data": {
    "taskCreationToDueDate": {
      "overall": {
        "past30Days": {
          "taskCount": 245,
          "avgCalendarDays": 12.5,
          "avgBusinessDays": 8.9,
          "medianCalendarDays": 10.0,
          "medianBusinessDays": 7.0
        },
        "past90Days": {
          "taskCount": 732,
          "avgCalendarDays": 13.2,
          "avgBusinessDays": 9.3,
          "medianCalendarDays": 11.0,
          "medianBusinessDays": 8.0
        }
      },
      "byDepartment": {
        "past30Days": {
          "Design": {
            "task_count": "145",
            "avg_calendar_days": "11.2",
            "avg_business_days": "7.9"
          },
          "Motion": {
            "task_count": "100",
            "avg_calendar_days": "14.1",
            "avg_business_days": "10.2"
          }
        },
        "past90Days": {
          ...
        }
      }
    },
    "needsUpdateToWaitingFeedback": {
      "overall": {
        "past30Days": {
          "transitionCount": 189,
          "avgTotalHours": 24.7,
          "avgBusinessHours": 18.3
        },
        "past90Days": {
          "transitionCount": 567,
          "avgTotalHours": 26.1,
          "avgBusinessHours": 19.2
        }
      },
      "byDepartment": {
        "past30Days": {...},
        "past90Days": {...}
      }
    },
    "deliverablesNeededToFinalFiles": {
      "overall": {
        "past30Days": {
          "transitionCount": 156,
          "avgTotalHours": 36.5,
          "avgBusinessHours": 28.1
        },
        "past90Days": {
          "transitionCount": 478,
          "avgTotalHours": 38.2,
          "avgBusinessHours": 29.5
        }
      },
      "byDepartment": {
        "past30Days": {...},
        "past90Days": {...}
      }
    }
  },
  "generatedAt": "2025-12-16T10:30:00.000Z"
}
```

### Error Response (500)

```json
{
  "error": "Database query failed",
  "details": {
    "error1": null,
    "error2": "...",
    ...
  }
}
```

## Analytics Metrics

### 1. Task Creation to Due Date
Measures time from when a task is created until its due date.

- **taskCount:** Number of tasks analyzed
- **avgCalendarDays:** Average calendar days (includes weekends)
- **avgBusinessDays:** Average business days (Mon-Fri only)
- **medianCalendarDays:** Median calendar days
- **medianBusinessDays:** Median business days

### 2. Needs Update to Waiting Feedback
Measures transition time from "Needs Update" status to "Waiting on Feedback" status.

- **transitionCount:** Number of status transitions
- **avgTotalHours:** Average hours (24/7)
- **avgBusinessHours:** Average business hours only

### 3. Deliverables Needed to Final Files
Measures time from "Deliverables Needed" to "Final Files" status.

- **transitionCount:** Number of transitions
- **avgTotalHours:** Average hours (24/7)
- **avgBusinessHours:** Average business hours only

## Database Functions Called

### RPC Functions
- `get_task_creation_to_due_date_analytics()`
- `get_needs_update_to_waiting_feedback_analytics()`
- `get_deliverables_to_final_files_analytics()`
- `get_task_creation_to_due_date_analytics_by_dept()`
- `get_needs_update_to_waiting_feedback_analytics_by_dept()`
- `get_deliverables_to_final_files_analytics_by_dept()`

## Use Cases

1. **Performance Dashboards:** Display team performance metrics
2. **Department Comparison:** Compare efficiency across departments
3. **Trend Analysis:** Track improvements over time
4. **Capacity Planning:** Understand typical turnaround times
5. **Process Optimization:** Identify bottlenecks in workflows

## Notes

- All data is calculated server-side via database functions
- Business hours are calculated as Monday-Friday only
- Median values help identify typical performance (less skewed by outliers)
- Department breakdown allows targeted improvements
- Response includes generation timestamp for cache management

