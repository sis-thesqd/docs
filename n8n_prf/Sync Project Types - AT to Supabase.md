# Sync Project Types - AT to Supabase

## Overview
**Workflow ID:** zfZlwQiR2h8DtOzP  
**Status:** Active  
**Created:** October 29, 2025  
**Last Updated:** December 16, 2025

## Purpose
Syncs File Size records from Airtable to Supabase, maintaining file size configurations and their associations with project types and accounts.

## Trigger
**Type:** Schedule  
**Frequency:** Every hour

## Workflow Process

### 1. Fetch File Sizes
- Searches all records from File Sizes table in Airtable
- Retrieves all fields including configurations and associations

### 2. Transform Data
- Edits fields to prepare for Supabase format
- Converts Member # (from Account) to accounts array
- Excludes unnecessary fields:
  - Account
  - Project Types
  - Record ID (from Account)
  - Member # (from Account)

### 3. Upsert to prf_file_sizes
Inserts or updates records with:
- `id`: Airtable record ID
- `prf_selection_type`: Project type ID (first from array)
- `file_size_name`: Name of file size
- `label`: Display label
- `label_bleed`: Label with bleed information
- `dimensions`: File dimensions
- `print_digital`: Print or Digital designation
- `bleed_size`: Bleed amount
- `bleed_info`: Additional bleed details
- `default_size`: Boolean for default selection
- `recommended`: Boolean for recommended status
- `active`: Based on Status field
- `task_content_template`: Template text for tasks
- `task_content_checklist_name`: Checklist name
- `clean_folder_name`: Cleaned folder name
- `file_size_folder`: Folder URL
- `row_updated`: Last modified timestamp
- `created_at`: Creation timestamp

### 4. Process Account Associations
- Splits accounts array from file sizes
- Creates records in `prf_account_file_sizes` for each account-filesize pairing

### 5. Batch Update Account File Sizes
Executes complex SQL operation:
- Creates temporary table for current batch
- Upserts account-filesize associations
- Deactivates removed associations (sets active = false)
- Maintains data integrity across updates

## Key Features
- **Hourly Sync:** Keeps Supabase in sync with Airtable source of truth
- **Upsert Logic:** Efficiently handles new and updated records
- **Association Management:** Tracks which file sizes apply to which accounts
- **Deactivation Handling:** Marks removed associations as inactive
- **Batch Processing:** Handles all records in optimized queries

## Database Tables

### Source (Airtable)
- File Sizes (tblVcQIWUOEo50N9k)

### Destination (Supabase)
- `prf_file_sizes` - Main file size records
- `prf_account_file_sizes` - Account-specific file size associations

## Status Mapping
- Airtable "Status" field (lowercase) → Supabase `active` boolean
- "active" = true
- Other values = false

## SQL Operation Details

### Temporary Table
```sql
CREATE TEMP TABLE temp_file_sizes_batch (
    id TEXT PRIMARY KEY,
    account INTEGER,
    file_size TEXT,
    active BOOLEAN
)
```

### Deactivation Logic
Sets `active = false` for records where:
- Account is in current batch
- Record ID not in temporary table
- Currently marked as active

## Output
- Returns only: id, prf_selection_type, label
- Single query batching enabled
- Skip on conflict enabled for account associations

## Notes
- Handles one-to-many relationship (file sizes to accounts)
- Recommended filter excludes records where Member # is 306 (disabled)
- Maintains historical record of file size changes
- Critical for dynamic file size options in forms
- Updates task content templates and checklist names


