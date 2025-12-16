# Task - Link SMS Projects

## Overview
**Workflow ID:** tLx4MnyLFuzfOwE1  
**Status:** Active  
**Created:** May 29, 2025  
**Last Updated:** December 16, 2025

## Purpose
Automatically links related SMS (Social Media Strategy) projects with their corresponding main design tasks, updating task descriptions and creating dependencies.

## Triggers
**Type:** Webhook + Schedule

### Webhook Trigger
**Method:** POST  
**Path:** `/webhook/link-sms-projects`  
**URL:** https://prf.thesqd.com/webhook/link-sms-projects

### Schedule Trigger
**Frequency:** Every hour at :12 minutes

## Workflow Process

### 1. Load Unprocessed Projects
- Queries Airtable view "SMS Projects to Process"
- Finds General Information records where SMS projects need linking
- Waits 10 seconds for data to settle

### 2. Validate Project Creation
- Filters for records where all project selection types have corresponding project details
- Checks: `PS Project Types` all exist in `PD Project Types`

### 3. Process Each Project Type
- Splits project types array
- For each type:
  - Searches for SMS Project Type configuration
  - If SMS Linked Type exists, continues processing

### 4. Find Linked Projects
- Searches for linked project details using GI Submission ID and SMS Linked Type
- Fetches linked submission details from Fillout
- Retrieves linked data from `prf_project_submissions` (Supabase)

### 5. Find SMS Project
- Searches for SMS project using original submission ID
- Gets SMS Data from Supabase

### 6. Update Main Task Content
- Replaces placeholder text with actual SMS task information
- Updates link from `pd-maindesignurl` to actual design task URL
- Removes `#linktodesign` placeholder
- Posts updated content to main task
- Logs update to `task_update_log`

### 7. Link Design Task
- Creates task link between SMS task and design task via ClickUp API
- Logs the linking action

### 8. Update Subtasks (if applicable)
- Fetches all subtasks of main task
- For subtasks containing `#linktodesign`:
  - Updates content with linked project information
  - Creates task dependency
  - Logs updates

### 9. Mark as Processed
- Updates General Information record
- Sets `SMS Projects Processed?` to TRUE

## Key Features
- **Automatic Linking:** Runs hourly to catch any missed linkages
- **Content Updates:** Dynamically updates task descriptions with linked project info
- **Dependency Management:** Creates proper task dependencies in ClickUp
- **Subtask Support:** Handles both main tasks and subtasks
- **Audit Trail:** Comprehensive logging to task_update_log

## Database Tables

### Supabase
- `prf_project_submissions` (read)
- `prf_content_history` (write)
- `task_update_log` (write)

### Airtable
- General Information (tblbHuKhDjUCwebAK)
- Project Details (tblUbcbEnRP1pLvaD)
- Project Types (tblY94xLE8XmHqqq7)

## Content Placeholders Replaced
- `` `The related project has not been processed yet...` `` → SMS Task Info
- `pd-maindesignurl` → Actual ClickUp task URL
- `#linktodesign` → (removed)

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Retry Logic:** Enabled on critical steps
- **Timeout:** 300 seconds

## Execution Settings
- **Save Progress:** Enabled
- **Caller Policy:** Workflows from same owner

## Notes
- Processes only unlinked SMS projects
- Scheduled execution ensures no projects are missed
- Updates both main tasks and all related subtasks
- Maintains content history for auditing

