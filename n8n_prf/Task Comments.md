# Task Comments

## Overview
**Workflow ID:** QId5gqp9mVgxSKhG  
**Status:** Active  
**Created:** July 15, 2025  
**Last Updated:** December 16, 2025

## Purpose
Posts pre-defined template comments to ClickUp tasks based on comment type, helping maintain consistent communication with clients.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/post-task-comment`  
**URL:** https://prf.thesqd.com/webhook/post-task-comment  
**Triggered by:** Airtable automation (wfloVmLRsRP534oeU / wac5gx1gzKeEj18cK)

## Workflow Process

### 1. Receive Comment Request
- Webhook receives payload with:
  - `comment_type`: Type of comment to post
  - `taskid`: ClickUp task ID (optional)
  - `gisid`: General Information Submission ID (optional)

### 2. Determine Task and Comment
Switch based on comment_type:
- **missing_gi:** Posts missing general info comment
- **internal:** Posts internal team comment
- **sermon_recap:** Posts sermon recap instructions
- **with_content_count:** Posts content count clarification

### 3. Get Project Details
Routes based on comment type:
- **If taskid provided:** Searches for PD by task ID
- **If gisid provided:** Searches by General Information ID

Uses specific Airtable views per comment type:
- missing_gi → viwb8dVOJ7TjFWNNa
- internal → viwHtD52ykP5FXGwU
- sermon_recap → viwjYVp0QDrb0X9au
- with_content_count → viw28Rd6rXtmurDMP

### 4. Prepare Comment Fields
- Extracts guest email from Project Details
- Determines task ID (from webhook or PD lookup)
- Maps comment_type to Airtable comment record ID:
  - `internal` → recySmzM8bGA9bTZc
  - `sermon_recap` → rec1HAVD7647Bdsxs
  - `missing_gi` → recoTT4cQ1AQyzaHx
  - `with_content_count` → recdXUYyZFbZtuZmW

### 5. Guest Lookup
- Searches Guests table by email
- Retrieves ClickUp User ID for mention capability

### 6. Post Comment
- Sends request to SiS1 comment webhook
- Includes:
  - Task ID
  - Comment template ID
  - User ID for mentions (if available)
- Continues on error (doesn't fail workflow if comment fails)

## Comment Types

### missing_gi
For tasks missing General Information link

### internal
Internal team communication templates

### sermon_recap
Instructions for sermon recap projects

### with_content_count
Clarification when content count involved

## Key Features
- **Template System:** Pre-defined comments in Airtable
- **User Mentions:** Includes user in comment when guest found
- **Flexible Lookup:** Works with task ID or GI submission ID
- **Error Resilience:** Continues even if comment posting fails
- **Wait Logic:** 20-second wait disabled (commented out)

## Database Tables

### Airtable
- Project Details (tblUbcbEnRP1pLvaD) - Read
- Guests (tbl8BhTfaBl4F410W) - Read
- Comment Templates - Referenced by record IDs

## Integration Points
- **Airtable:** Comment template storage and view filtering
- **SiS1:** Comment posting service
  - Endpoint: https://sis1.thesqd.com/webhook/clickup-comment

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Continue on Error:** Enabled for comment posting
- **Timeout:** 300 seconds

## Notes
- Comment templates stored in Airtable for easy editing
- Supports user mentions when guest record exists
- Multiple views optimize comment type routing
- Retry logic on guest lookup (5-second wait)
- Flexible input accepts either task ID or GI submission ID




