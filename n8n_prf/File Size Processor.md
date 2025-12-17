# File Size Processor

## Overview
**Workflow ID:** AsfOk8sSDZhGhetk  
**Status:** Active  
**Created:** July 31, 2025  
**Last Updated:** December 16, 2025

## Purpose
Processes file size template uploads from form submissions, uploads templates to Dropbox, and updates Airtable records with folder links.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/file-size-processor`  
**URL:** https://prf.thesqd.com/webhook/file-size-processor

## Workflow Process

### 1. Receive Submission
- Captures webhook payload with form submission
- Extracts default fields:
  - Submission ID
  - Form ID
  - Account ID (accid)
  - Project Type ID (ptid)
  - Upload path: `/Church Media Company Team Folder/Submission File Management (do not delete)/File Size Templates (do not delete)`

### 2. Fetch Submission Details
- Retrieves full submission from Fillout API
- Waits 3 seconds for data propagation

### 3. Query File Size Record
- Searches Airtable for File Size record by Submission ID
- Waits for record to be created by other workflows

### 4. Get Related Data
- Fetches Account details from Airtable
- Retrieves Project Type information

### 5. Check for Templates
Evaluates if Template Uploader field has files:
- **If NO templates:** Updates record with `Template Checked? = Yes`, exits
- **If templates exist:** Continues to upload process

### 6. Upload Process

#### a. Get Dropbox Keys
- Fetches Dropbox access credentials from Airtable
- Record ID: recvQPFvL4OHSEbVt

#### b. Prepare Templates
- Extracts files array from submission
- Separates each template file

#### c. Upload to Dropbox
- Uploads each file via Lambda function
- Lambda URL: `https://isccrtvuk2ca56ghkehtbdls2m0acpoh.lambda-url.us-east-2.on.aws/`
- Sends: Dropbox key, file URL, target path

#### d. Extract File Path
- Parses upload response to get Dropbox file path

#### e. Rename File
- Renames uploaded file to standardized format:
  - `{MemberNumber}_{ProjectTypeName}_{Filename}`
- Removes special characters: `"`:,'`
- Uses Dropbox API `/files/move_v2`
- Autorename enabled if conflicts

#### f. Generate Shared Link
- Creates shared link for uploaded template
- Uses Dropbox API `/sharing/create_shared_link_with_settings`

### 7. Update Airtable
- Updates File Size record with:
  - File Size Folder URL (shared link)
  - Template Checked? = Yes

## Key Features
- **Template Management:** Handles multiple template files per submission
- **Automatic Renaming:** Standardizes file naming convention
- **Shared Link Generation:** Creates accessible links for templates
- **Dropbox Integration:** Full lifecycle management (upload, move, share)

## Dropbox Configuration
- **Root ID:** 2562687859
- **User ID:** dbmid:AAAHEu7mbtnr5hPFM5Vf1sf_WXFqCw3J33g
- **Settings:**
  - Allow shared folder: true
  - Autorename: true
  - Allow ownership transfer: false

## Database Tables

### Airtable
- File Sizes (tblVcQIWUOEo50N9k) - Read/Write
- Accounts (tblJEtYMR7ZPSOkc6) - Read
- Project Types (tblY94xLE8XmHqqq7) - Read
- Keys (tblQ8o83lobk6PPzz) - Read (Dropbox credentials)

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Retry Logic:** Enabled on critical operations
- **Wait Between Retries:** 3 seconds
- **Timeout:** 300 seconds

## Lambda Function
Handles actual file upload to Dropbox:
- Accepts: API key, file URL, destination path
- Returns: File path in Dropbox

## File Naming Convention
```
{Member #}_{Project Type Name}_{Original Filename (cleaned)}
```
Example: `12345_Sermon Series Graphics_template.psd`

## Notes
- Requires Dropbox keys stored in Airtable
- Handles multiple template files per submission
- Uses Lambda for server-side Dropbox operations
- Creates publicly accessible shared links
- Removes special characters that could cause issues
- Waits for File Size record creation by other workflows




