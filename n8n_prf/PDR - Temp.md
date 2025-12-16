# PDR - Temp

## Overview
**Workflow ID:** 0S5CWqup36RUPSsi  
**Status:** Active  
**Created:** October 3, 2025  
**Last Updated:** December 16, 2025

## Purpose
Temporary implementation of Project Details Processor workflow. Processes project submissions, creates ClickUp tasks with dynamic content, and handles multi-content projects with subtask creation.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/eb738c75-081b-4b4e-b742-5452c1d0d1f1-temp`  
**URL:** https://prf.thesqd.com/webhook/eb738c75-081b-4b4e-b742-5452c1d0d1f1-temp

## Workflow Process

### 1. Validate Submission
- Checks if submission exists in `prf_project_submissions`
- Stops if not found (must rerun)
- Fetches detailed submission from Fillout

### 2. Filter Demo Submissions
- Excludes submissions from demo user (demo-rec-301)
- Saves execution data for tracking

### 3. Retrieve Core Data
- **Guest:** User information
- **Project Type:** Form and configuration details
- **ClickUp List:** Workspace location
- **General Info:** Creative direction and submission details

### 4. Handle Missing GI
- If no GI record found, calls Fix Missing Info workflow
- Waits and retries if Project Details missing

### 5. Generate Folder Code
- Checks for sequential folder number in Supabase
- Creates or retrieves folder code for project
- Updates General Info with folder code and design style

### 6. Build Task Content

#### Content Variables
- Fetches variables from `prf_content_variables` table
- Replaces placeholders with actual values:
  - `pd-header`: Administrative info, folder code, go-live date
  - `pd-filesizeinfo`: File size processing message
  - `pd-audience`: Audience/department information
  - `pd-description`: Project description
  - `cd-designstyle`: Design style preference
  - `cd-imagery`: Specific imagery requests

#### Content Processing
- Checks if content contains `pd-multiplecontent`
- Routes to appropriate handler:
  - **None:** Single content item
  - **> 10:** Multiple content items (creates subtasks)

### 7. Create Main Task
- Creates ClickUp task with formatted title
- Assigns to user from Guest record
- Sets in appropriate list/folder/space

### 8. Handle Multiple Content

#### If Content Count ≤ 10:
- Creates single task with all content
- Updates task description
- Logs to content history

#### If Content Count > 10:
- Calculates subtask ranges (10 items per subtask)
- Creates subtasks titled: "1-10", "11-20", etc.
- First subtask gets full content
- Remaining subtasks get reference to main task
- All subtasks marked as DEPENDENT
- Creates dependencies to main task

### 9. Update Databases
- **Airtable:** Links task URL, content, title
- **Supabase:** Updates `prf_project_submissions` with:
  - ClickUp ID
  - Task content
  - Other metadata
- **Content History:** Records initial content load

### 10. File Size Processing
- If file sizes present, triggers file size content update
- Updates task with file size information

### 11. Task Properties
- Calls update task properties endpoint
- Sets:
  - Department
  - Time estimate
  - Tags
  - Custom fields

## Key Features
- **Smart Content Handling:** Automatically creates subtasks for large projects
- **Dynamic Variables:** Replaces content placeholders with actual data
- **Folder Code Generation:** Sequential numbering per account
- **Content History:** Tracks all content changes
- **Retry Logic:** Auto-retries if Project Details not ready

## Database Tables

### Supabase
- `prf_project_submissions` (read/write)
- `prf_general_submissions` (read/update)
- `prf_content_variables` (read)
- `prf_content_history` (write)
- `prf_account_project_counter` (read/write)

### Airtable
- Project Details (tblUbcbEnRP1pLvaD)
- General Information (tblbHuKhDjUCwebAK)
- Project Types (tblY94xLE8XmHqqq7)
- Guests (tblSpD6Sw8VVnr0Wh)

## Content Formatting
- Removes `==` markers around descriptions
- Strips empty headers (** followed by only hyphens)
- Cleans multiple newlines
- Removes special markdown syntax
- Encodes URLs properly

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Retry Logic:** Enabled on API calls
- **Timeout:** 300 seconds
- **Auto-retry:** 1 minute wait if Project Details missing

## Subtask Logic
For projects with > 10 content items:
- Range 1: Gets full task content with all details
- Ranges 2+: Get reference message pointing to main task
- All subtasks link to main task as dependency

## Special Handling
- Sermon Series with SMS: Calls SMS project handler
- Missing Project Details: Auto-retries after delay
- First-time submissions: Creates folder codes

## Notes
- "Temp" in name suggests this is a temporary version
- Likely testing ground for Project Details Processor improvements
- Complex content variable system for dynamic task generation
- Handles edge cases like missing GI gracefully
- Creates comprehensive task structure for large projects

