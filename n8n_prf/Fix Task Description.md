# Fix Task Description

## Overview
**Workflow ID:** tWOqDpZEb5TWN4Xs  
**Status:** Active  
**Created:** August 14, 2025  
**Last Updated:** December 16, 2025

## Purpose
Repairs task descriptions missing audience and project description information by inserting them before the Creative Direction section.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/fix-missing-description`  
**URL:** https://prf.thesqd.com/webhook/fix-missing-description

## Workflow Process

### 1. Input Mode - Direct Fix (Webhook with pdid)
When provided with specific Project Detail ID:

#### a. Get Project Details
- Fetches PD record from Airtable by ID

#### b. Get General Information
- Retrieves GI record linked to PD

#### c. Get ClickUp Task
- Fetches task details from ClickUp API

#### d. Validate Content
Checks if task needs fixing:
- Has text_content
- Not placeholder text ("We'll update this task once...")
- Doesn't have View/Get links already formatted

#### e. Fix Content
- Inserts audience before Creative Direction
- Inserts description (wrapped in ==) before Creative Direction
- Cleans up formatting

#### f. Update Task
- Posts updated content to ClickUp via SiS1
- Updates Airtable PD record
- Updates Supabase `prf_project_submissions`
- Logs to `prf_content_history`

### 2. Batch Mode - Process Missing Descriptions (No pdid)

#### a. Search Missing Descriptions
- Queries Airtable view "Missing Description" (viwaBvW5QdGov4rfy)
- Limited to 50 records
- Ordered by Project Type

#### b. Loop Through Records
- Processes each PD record individually
- Gets General Information
- Gets ClickUp task details from Supabase

#### c. Validate and Fix
- Same validation as direct mode
- Updates content with audience and description
- **Option 1:** If needs fixing → Updates task content
- **Option 2:** If TEMP Bypass set → Skips to next

#### d. Update All Systems
- Updates task in ClickUp
- Updates Airtable PD record
- Updates Supabase project submission
- Logs to content history

#### e. Continue Loop
- Processes all records in batch

## Key Features
- **Dual Mode:** Direct fix or batch processing
- **View-Based:** Uses Airtable view to find tasks needing fixes
- **Validation:** Checks for placeholder text and existing formats
- **Batch Processing:** Handles multiple tasks efficiently
- **Comprehensive Updates:** Syncs across all systems

## Content Insertion Logic
Finds `### 🎨 Creative Direction` marker and inserts before it:
```markdown
**Audience:** {audience}

=={description}==

### 🎨 Creative Direction
```

## Validation Checks
Content is considered "needs fixing" if:
- Has text_content OR is placeholder text OR TEMP Bypass = Yes
- AND doesn't contain formatted "View the ... here" or "Get the ... here" patterns

## Database Tables

### Supabase
- `prf_project_submissions` (read/write)
- `prf_content_history` (write)
- `task_update_log` (write)

### Airtable
- Project Details (tblUbcbEnRP1pLvaD) - Read/Write
- General Information (tblbHuKhDjUCwebAK) - Read

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Timeout:** 300 seconds
- **Save Progress:** Enabled
- **Continue on Error:** Enabled for ClickUp API

## Content History
Update type logged as: `content_fix`

## Task Update Log
Narratives:
- "task created"
- "content fix - update description"

## Notes
- TEMP Bypass field allows skipping specific records
- Cleans up formatting (multiple newlines, etc.)
- Preserves existing Creative Direction section
- Non-destructive updates (adds, doesn't replace)
- Handles both webhook and batch processing modes
- Test area section suggests ongoing development


