# Website Support Requests

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `x1clzmVD3YwUrxZn` |
| **Status** | Active |
| **Created** | May 29, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Processes website support request submissions and creates ClickUp tasks for:
- Campus directory setup/additions
- Event management (add/modify/remove)
- Image updates
- New pages and sections
- Sermon posts (with optional blog creation)
- Small group directory setup/additions
- Staff directory management
- Text updates
- Technical support
- Blog setup and posts
- Google Analytics setup

---

## Triggers

### 1. Webhook

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/web-support-requests` |
| **Test** | `https://prf.thesqd.com/webhook-test/web-support-requests` |

### 2. Execute Workflow Trigger

**WebSermon Pack** - Can be called from other workflows with:
- `temp_content`
- `formid`
- `submission`

---

## Workflow Flow

### 1. Initial Data Gathering

- **Detail Submission** - Fetches from Fillout API
- **Project Details** - Checks for existing PD record
- **Guest** - Fetches guest data from Brains
- **ClickUp List** - Gets correct list via clickup-list-checker webhook
- **Web Support Option** - Fetches support type from Strategy base

### 2. Common Fields Setup

Sets up shared fields including:
- `reason_code` - From submission calculation
- `final_content` - Task content template
- Space/Folder/List IDs
- Team member ClickUp IDs (Nathan, Emily, Tia)
- `website` - Selected website name
- `website_record_id` - Airtable record ID

### 3. Reason Code Routing

**Switch Node** routes based on `reason_code`:

| Reason Code | Handler |
|-------------|---------|
| `google_analytics` | Google Analytics setup |
| All others | Create Task Fields code node |

### 4. Task Creation Logic

**Create Task Fields** code node handles all non-GA requests with helper functions:

| Function | Purpose |
|----------|---------|
| `createBaseTask()` | Creates standard task object |
| `pushContent()` | Generates content for multi-item requests |
| `createCampusTask()` | Campus addition tasks |
| `createEventTask()` | Event add/modify tasks |
| `createRemoveEventTasks()` | Individual removal tasks |
| `createSmallGroupTask()` | Small group tasks |
| `createStaffTask()` | Staff directory tasks |

### 5. Time Estimate Calculations

| Request Type | Time Estimate Formula |
|--------------|----------------------|
| Campus | 1: 20min, 2: 30min, 3+: 45min |
| Event Add | 1: 15min, 2: 20min, 3+: 30min |
| Event Modify | 1: 15min, 2: 20min, 3+: 25min |
| Event Remove | 10min per event |
| Image | 1: 5min, 2-3: 10min, 4-5: 15min, 6-8: 20min, 9+: 30min |
| New Page | 90min |
| New Section | 1-2: 40min, 3+: 60min |
| Sermon | 30min |
| Small Group | 1-5: 30min, 5-10: 45min, 10+: 60min |
| Staff Add | 1-5: 20min, 5-10: 35min, 10+: 60min |
| Text | (count × 5) + 15 min |
| Technical | 60min |
| Blog Setup | 90min |
| Blog Post | 30min |

### 6. Content Processing

- **Upload Fields** - Aggregates file upload field data
- **Fix Content** - Processes file uploads and cleans content:
  - Replaces `wsr-header` with submission timestamp
  - Processes upload fields into markdown links
  - Removes empty sections
  - Cleans up formatting

### 7. Task Creation & Updates

- **Create Main Task** - Creates ClickUp task
- **Set Web Field** - Sets website custom field
- **Update Content** - Updates task description via SISX webhook
- **Update Main Task** - Updates Airtable Project Details
- **project_submissions** - Updates Supabase submission record
- **prf_content_history** - Logs content history

### 8. Sermon-Specific Processing

For `reason_code === 'sermon'`:
- **strategy_sermon_data** - Saves sermon data to PostgreSQL
- **if with blog** - Checks if blog post requested
- **Create Subtask** - Creates sermon blog subtask with:
  - SOP link for blog creation
  - Action Items checklist (Generate Blog Post, Upload & Publish)

### 9. Directory Setup Updates

For campus, small_group, or staff requests:
- **Get Website** - Fetches website record from Strategy
- **Update Strategy** - Adds content type to "Website Contents" field

### 10. Google Analytics Deactivation

For GA setup requests:
- **Deactivate Option for Website** - Removes website from allowed GA setup list

---

## Supported Request Types

| Reason Code | Description | Default Assignees |
|-------------|-------------|-------------------|
| `campus` | Campus directory | Emily + Member |
| `event` | Event management | Nathan + Member |
| `image` | Image updates | Nathan + Member |
| `new_page` | New webpage | Emily + Member |
| `new_section` | New section | Nathan + Member |
| `sermon` | Post sermon | Nathan + Member |
| `small_group` | Small group directory | Emily + Member |
| `staff` | Staff directory | Nathan + Member |
| `text` | Text updates | Nathan + Member |
| `technical_support` | Technical issues | Emily + Member |
| `google_analytics` | GA dashboard setup | Nathan + Member |
| `blog_setup` | Blog setup | Emily + Tia |
| `blog_post` | New blog post | Emily + Tia |

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | ⚓ Website Support Upload Fields | Upload field configurations |
| `appXCvcntPYSBZey2` | Project Details | Project submission records |
| `appjHSW7sGtitxoHf` (Strategy) | 🔑 Website Support Options | Support type options |
| `appjHSW7sGtitxoHf` (Strategy) | 🧩 Websites | Website contents tracking |
| `app6LbfcLAbwsfobY` (Brains) | Guests | Guest/member lookup |

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `strategy_sermon_data` | Sermon submission data |
| `prf_project_submissions` | Submission tracking |
| `prf_content_history` | Content version history |

---

## Logging

Operations logged to `task_update_log`:
- `task created`
- `custom field update - web - {website}`
- `content added`
- `create subtask - parent: {id}`
- `create checklist and checklist items`


