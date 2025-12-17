# PRF - Update Users

## Overview
**Workflow ID:** XlljGsmnUf6gBIXJ  
**Status:** Active  
**Created:** September 25, 2025  
**Last Updated:** December 16, 2025

## Purpose
Manages ClickUp guest user access by adding or removing users from church folders, syncing guest records in Airtable, and sending notification emails.

## Trigger
**Type:** Webhook + Execute Workflow  
**Method:** POST  
**Path:** `/webhook/XlljGsmnUf6gBIXJ`  
**URL:** https://prf.thesqd.com/webhook/XlljGsmnUf6gBIXJ  
**Response Mode:** Last node, no body

**Can also be executed by other workflows** via Execute Workflow Trigger with email parameter.

## Workflow Process

### Add User Flow

#### 1. Input Processing
- Receives user addition request with emails to add
- Validates member ID exists
- Updates uptime monitor heartbeat

#### 2. Email Splitting & Validation
- Splits multiple emails
- Validates email format with regex
- Filters valid emails only

#### 3. User Lookup
- Searches ClickUp team for user by email
- Gets account details from Airtable
- Merges user and account data

#### 4. Record Existence Check
- Checks if guest record exists in Airtable
- Routes based on:
  - **Record Exists:** Update existing
  - **No Record:** Create new

#### 5. Add to Airtable (New Users)
- Creates new Guest record with:
  - Name, Email
  - ClickUp User ID
  - Account link
  - Invite date
  - Hide from list = FALSE

#### 6. Update Airtable (Existing Users)
- Updates existing Guest record
- Adds new account to Account Linked array
- Preserves existing account links

#### 7. Grant ClickUp Folder Access
- Adds guest to church's ClickUp folder
- Permission level: "edit"
- Include shared: false

#### 8. Send Welcome Email
- Checks for duplicate emails (removes in current execution)
- If CSS Rep exists for account:
  - Sends email with MailerSend
  - Template ID: 3z0vklo710eg7qrx
  - Includes: CSS Rep name, church name

### Remove User Flow

#### 1. Split Users to Remove
- Processes array of users to remove
- Gets account details

#### 2. Guest Record Lookup
- Finds guest by ClickUp User ID
- Gets account information
- Checks number of linked accounts

#### 3. Account Count Handling
- **More than 1 account:** 
  - Removes specific account from array
  - Updates guest record
  - Sends removal notification email (disabled)
- **Only 1 account:**
  - Checks if record exists
  - Deletes entire guest record
  - Sends final removal email (disabled)

#### 4. ClickUp Access Removal
- Removes guest from ClickUp folder
- Handles errors gracefully if user already removed
- Continues execution on error

## Key Features
- **Duplicate Prevention:** Removes duplicates within execution
- **Email Validation:** Regex-based email format checking
- **Batch Processing:** Handles multiple users simultaneously
- **Account Association:** Supports users across multiple accounts
- **Graceful Degradation:** Continues on ClickUp API errors

## Database Tables

### Airtable
- Guests (tbl8BhTfaBl4F410W) - Read/Write
- Accounts (tblFX0L0bSHei7N8M) - Read

## Email Templates

### Welcome Email
**Template ID:** 3z0vklo710eg7qrx  
**Variables:**
- CSSrep: CSS Rep full name
- firstname: CSS Rep first name
- churchname: Church name

### Removal Email (Disabled)
**Template ID:** pxkjn4178dq4z781  
**Variables:**
- UserName: Guest first name
- ChurchName: Church name
- submissionID: Timestamp
- AccountNumber: Member ID
- PersonWhoSubmittedRemovalsEmail: Submitter email

## ClickUp Settings
- **Team ID:** 1235435
- **Permission Level:** edit
- **Can Edit Tags:** false
- **Can See Time:** false
- **Can Create Views:** false
- **Role:** 4 (guest)

## Error Handling
- **Continue on Error:** ClickUp API operations
- **Full Response:** Enabled on folder access operations
- **Retry Logic:** Enabled on Airtable operations

## Integration Points
- ClickUp Team API for guest management
- Airtable for guest/account records
- MailerSend for email notifications
- Uptime Robot for health monitoring

## Notes
- Removal email functionality currently disabled
- Supports recursive execution (can call itself)
- Handles edge cases like missing CSS Reps
- Account linking preserves multi-account access
- Robust error handling prevents workflow failures




