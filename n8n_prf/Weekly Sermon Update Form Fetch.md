# Weekly Sermon Update Form Fetch

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `SMvPeVbg78RzEziY` |
| **Status** | Active |
| **Created** | June 18, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Provides API endpoints for the Weekly Sermon Update form to fetch:
- Current Sunday's date
- Website ID for a member
- Previous sermon submission data (for pre-filling forms)

---

## Triggers

This workflow has **3 webhook triggers**:

### 1. Get Sunday

| Property | Value |
|----------|-------|
| **Endpoint** | `/webhook/weekly-sermon-update-get-sunday` |
| **Method** | GET |
| **Response Mode** | Respond to Webhook node |

**Returns**: The most recent Sunday's date (or today if Sunday)

**Logic**:
```javascript
$now.minus({days: $now.weekday === 7 ? 0 : $now.weekday}).startOf('day').toISO().split('T')[0]
```

---

### 2. Get Website

| Property | Value |
|----------|-------|
| **Endpoint** | `/webhook/weekly-sermon-update-get-website` |
| **Method** | GET |
| **Response Mode** | Respond to Webhook node |

**Query Parameters**:
- `memberid` - The member's account number

**Flow**:
1. Searches Airtable `⚓ Websites` table for matching Member ID
2. Returns `website_id` if exactly one match found, empty string otherwise

---

### 3. Previous Submission

| Property | Value |
|----------|-------|
| **Endpoint** | `/webhook/weekly-sermon-update-get-prev-submission` |
| **Method** | GET |
| **Response Mode** | Respond to Webhook node |

**Query Parameters**:
- `memberid` - The member's account number
- `email` - The submitter's email

**Flow**:
1. Queries `strategy_sermon_data` table in PostgreSQL
2. Filters by account and email
3. Orders by `created_at` DESC, returns most recent
4. Returns sermon data fields for form pre-fill

**Returns**:
| Field | Description |
|-------|-------------|
| `series_title` | Previous series title |
| `series_description` | Previous series description |
| `series_graphics` | Link to series graphics |
| `sermon_title` | Previous sermon title |
| `sermon_description` | Previous sermon description |
| `speaker_rec_id` | Airtable record ID for speaker |
| `sermon_date` | Previous sermon date |
| `web_info_selection` | Web info selections |

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | ⚓ Websites | Website lookup by member |

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `strategy_sermon_data` | Previous sermon submission data |

---

## Usage

These endpoints are called by Fillout forms to:
1. **Get Sunday** - Pre-fill the sermon date field with the current/most recent Sunday
2. **Get Website** - Auto-select the member's website for web update options
3. **Previous Submission** - Pre-fill form fields with data from the last submission to save time




