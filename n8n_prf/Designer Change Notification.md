# Designer Change Notification

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `nswEvjjZXxDaA4nB` |
| **Status** | Active |
| **Created** | September 3, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Handles designer change requests when a client requests a different designer ("fresh eyes") for their project. Notifies:
1. The appropriate Project Manager in `#pm-notifications`
2. The designer's squad lead via direct message

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/designer-change-request` |
| **Test** | `https://prf.thesqd.com/webhook-test/designer-change-request` |

**Request Body**:
```json
{
  "taskURL": "<clickup_task_url>",
  "projectName": "<project_name>",
  "designer": "<designer_full_name>",
  "feedback": "<client_feedback>",
  "workAgain": "<yes/no/maybe>",
  "memberID": "<member_number>",
  "memberName": ["<church_name>"]
}
```

---

## Workflow Flow

### 1. Data Gathering

**Execute a SQL query** - Complex query that:
- Extracts task ID from URL
- Gets task and list info from `tasks` and `clickup_lists` tables
- Gets account info including PM Slack IDs for Design and Video
- Gets designer info (name, job title, department, slack_id)
- Determines correct PM based on designer's squad

**Query Logic for PM Selection**:
```sql
CASE 
  WHEN designer_squad = 'Video Squad' THEN pm_slack_id_video
  WHEN designer_squad = 'Design Squad' THEN pm_slack_id_design
  WHEN list_department IN ('Design & Video', 'Video') THEN pm_slack_id_video
  ELSE pm_slack_id_design
END
```

### 2. Designer Lookup

**Fetch Designer Data** - Searches Airtable "Tagged by Client" base for designer in "Design + Video Squad" view

### 3. PM Determination

**Determined Tagged PM and Reports To** code node:

| Designer Reports To | Tagged PM |
|---------------------|-----------|
| Matthew McCaigue | Mackenzie Moore |
| Michelle Vanderford | Mackenzie Moore |
| Derek Overton | Mackenzie Moore |
| Christina Cochran | Tia Overton |
| Jarrid Westfall | Tia Overton |
| Contractor (title) | Mackenzie Moore |
| Default | Tia Overton |

### 4. Squad Lead Lookup

**Fetch Reports To Data** - Searches "Squad Leads" view for the designer's supervisor

### 5. Notifications

**Notify Tagged PM in #pm-notifications**:
- Channel: `C049QK5NGFJ`
- Uses Block Kit message with:
  - Header: "👀 Designer Change Requested 👀"
  - Tags the determined PM
  - Shows account, project link, feedback
  - Shows "work again" response
  - Includes frog image accessory

**Sends DM to Reports To**:
- Direct message to squad lead
- Same content format
- Notes that PM has also been notified

---

## Slack Message Content

```
Hey, @{PM}! A member has requested a fresh set of eyes on a project. 
Can you help them out?

*Account*: {member_id} - {church_name}
*Project*: {project_link}
*Feedback*: {feedback}

When asked if they'd like to work with *{designer}* again, 
the member said *{workAgain}*.
```

---

## Data Sources

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `tasks` | Task lookup by ID |
| `clickup_lists` | List/account info |
| `accounts` | Account and PM Slack IDs |
| `employees` | Designer and PM info |

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appzRaQdOHuESKr3n` (Tagged by Client) | All current employees | Designer lookup |
| `appzRaQdOHuESKr3n` (Tagged by Client) | All current employees | Squad lead lookup |

---

## Slack Channels

| Channel | ID | Purpose |
|---------|-----|---------|
| #pm-notifications | `C049QK5NGFJ` | PM team notifications |

---

## Notes

- The frog image used in notifications: `https://i.postimg.cc/0NHZHXmj/Firefly-a-frog-with-massive-eyes-examining-a-computer-47761.jpg`
- Feedback text is sanitized (quotes and newlines removed) before sending to Slack

