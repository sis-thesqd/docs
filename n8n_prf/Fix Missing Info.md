# Fix Missing Info

## Overview
**Workflow ID:** jjM1U3QtQA3guJQr  
**Status:** Active  
**Created:** July 8, 2025  
**Last Updated:** December 16, 2025

## Purpose
Repairs incomplete General Information records by creating missing database entries and establishing proper relational links when project details lack GI references.

## Triggers
**Type:** Webhook + Schedule

### Webhook Trigger
**Method:** POST  
**Path:** `/webhook/fix-missing-info`  
**URL:** https://prf.thesqd.com/webhook/fix-missing-info  
**Response Mode:** Uses Respond to Webhook node

### Schedule Trigger
**Frequency:** Every hour at :15 minutes

## Workflow Process

### 1. Check Existing GI Record
- Queries `prf_general_submissions` for submission ID
- Determines if record exists or needs creation

### 2. Path A: GI Exists (Scheduled)
- Finds Project Details missing GI in Airtable view
- Gets submission data from Supabase
- Links existing GI to orphaned Project Details
- Updates PD with GI reference

### 3. Path B: GI Missing (Webhook)
When called via webhook with missing GI:
- Retrieves account information
- Gets user information from ClickUp User ID
- Creates new GI record in Supabase with minimal data

### 4. Create Selection Data
- Stores selection submission in `prf_selection_submissions`
- Includes:
  - Trust the squad preference
  - Selected project IDs
  - Design style
  - Vision and inspiration images

### 5. Process Project Types
- Splits selected project types
- Fetches type details from `prf_selection_types`
- Formats fields with ministry and account info
- Aggregates all selections

### 6. User Information Lookup
- Retrieves user record using ClickUp User ID
- Uses Airtable record checker

### 7. Create Ministry Department (if needed)
- Checks if ministry department exists in Supabase
- Creates department record in Airtable if ministry name provided

### 8. Create User Selection Record
- Links submission to:
  - Account
  - Guest/User
  - Submission timestamp

### 9. Create Project Selection Record
- Links all selected project types
- Associates with submission ID

### 10. Create/Update General Information
- Creates comprehensive GI record with:
  - Title (or "[Missing Info]" if unavailable)
  - Account link
  - Department
  - Description
  - Creative direction
  - Design style
  - Links to User Selection and Project Selection

## Key Features
- **Dual Mode:** Works both scheduled and on-demand
- **Self-Healing:** Repairs missing relational data
- **Graceful Degradation:** Uses "[Missing Info]" when data unavailable
- **Comprehensive Logging:** Tracks all repair operations

## Database Tables

### Supabase
- `prf_general_submissions` (read/write)
- `prf_selection_submissions` (write)
- `prf_ministry_departments` (read)

### Airtable
- General Information (tblbHuKhDjUCwebAK)
- Account - Department (tbl93Rrk6L4jZCkhe)
- User Selection (tbliUuVlf8xQsPBul)
- Project Selection (tblik6QFXNagHRclV)
- Project Details (tblUbcbEnRP1pLvaD)

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Retry Logic:** Enabled on Airtable operations
- **Timeout:** 300 seconds

## Schedule View
**Airtable View:** "Missing GI" (viwb8dVOJ7TjFWNNa)
- Shows Project Details without GI links
- Limited to 30 records per run

## Notes
- Critical for maintaining data integrity
- Handles edge cases from incomplete submissions
- Can be triggered manually via webhook
- Scheduled run ensures orphaned records are eventually fixed
- Properly handles optional creative direction data

