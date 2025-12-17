# Brand Guide Edit

## Overview
**Workflow ID:** L2RbFxyuGP9247Vf  
**Status:** Active  
**Created:** June 12, 2025  
**Last Updated:** December 16, 2025

## Purpose
Processes brand guide edit requests submitted through forms, creates ClickUp tasks for the design team, and notifies the PM team via Slack.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/brand-edit`  
**URL:** https://prf.thesqd.com/webhook/brand-edit

## Workflow Process

### 1. Input Validation
- Receives webhook payload with form submission data
- Fetches detailed submission from Fillout API
- Validates URL parameters (email and memberid)
- Stops with error if missing required parameters

### 2. Data Retrieval
- Checks for existing Project Details record in Airtable
- Fetches Project Type information
- Retrieves ClickUp List details for task creation
- Gets Guest information based on email

### 3. Task Creation
- Creates ClickUp task with:
  - Title: "Brand Guide Edit"
  - Location: Determined by ClickUp List checker
  - Assignee: Designer from Guest record
  - Status: "READY TO START"
  - Time Estimate: 60 minutes
- Logs task creation to `task_update_log`

### 4. Content Update
- Sends task content to SiS1 for description formatting
- Updates task with formatted content
- Logs content update to `task_update_log`

### 5. Task Properties & Airtable Sync
- Updates task properties via webhook
- Updates Project Details record in Airtable with:
  - ClickUp Task Link
  - Task Content (Raw)
  - Initial Task Title

### 6. Slack Notification
- Posts message to #pm-brand-web channel
- Includes task link and suggested welcome message
- Uses custom bot profile with emoji

## Key Features
- **Error Handling:** Error workflow configured (38tttU6Tcl3Y8tZy)
- **Retry Logic:** Automatic retries on API failures
- **Audit Trail:** Comprehensive logging to task_update_log

## Data Sources
- **Fillout API:** Form submission details
- **Airtable:** Project Details, Project Types, Guests
- **ClickUp API:** Task creation and management
- **Slack API:** Team notifications

## Integration Points
- Fillout form submissions
- SiS1 content formatting service
- PRF task properties updater
- Slack notifications

## Notes
- Fixed 60-minute time estimate for all brand guide edits
- Automatically assigns to designer based on guest email
- Requires valid email and member ID in URL parameters




