# MySquad Chrome Extension

The MySquad Chrome Extension is a comprehensive tool designed for managing tasks, team members, and various squad operations. It provides quick access to essential features and information directly from your browser.

**Download:** [Chrome Web Store](https://chromewebstore.google.com/detail/mysquad/accdcbpmoifgecjahfoabamdgjbnmnfb?authuser=0&hl=en)

## Access Permissions

- **Executives and SIS Team Members:** Full access to all pages
- **Other Users:** Access only to pages pertinent to their role

# Table of Contents

1. [Details](#details)
2. [Internal Comments](#internal-comments)
3. [Creative Tools](#creative-tools)
4. [Designer Scheduling](#designer-scheduling)
5. [SM Squad](#sm-squad)
6. [Admin](#admin)
7. [Squad Tools](#squad-tools)
8. [Settings](#settings)

---

## Details

The Details page provides comprehensive information about ClickUp tasks. **Note:** This page only displays data when viewing a ClickUp task in your browser.

### Task Details Section

Displays core task information:
- **Task Title:** Full name of the task
- **Task ID:** Unique identifier for the task
- **Status:** Current task status (e.g., CLOSED, In Progress)
- **Department:** Assigned department (e.g., Design Squad)

### Dropbox Folder

Shows the associated Dropbox folder for the task with:
- Folder name and path
- Quick copy and folder icon buttons for easy access

### Task Actions

Quick action buttons for common task operations:
- **Refresh Dropbox Folder Location:** Update the Dropbox folder link
- **Refresh Custom Fields:** Sync custom field data from ClickUp
- **Add Back to Queue:** Return task to the assignment queue
- **Block from Auto-Assignment:** Prevent automatic assignment

### Task Assignment Actions

Tools for managing task assignments:
- **Generate Time Estimate:** Calculate estimated completion time
- **Find Suitable Designer:** Locate available designers for the task
- **Show Me Everything:** Expand to see all available options

### Account & Plans

Displays account-level information:
- **Account Number:** Unique account identifier
- **Account Name:** Full account name
- **Plans:** Current subscription plan (e.g., Unlimited)
- **Add-ons:** Active add-ons with dates (e.g., +CD (2025))

### Active Tasks

Shows current task load:
- **Task Count:** Number of active tasks (e.g., 2/7)
- **Visual Progress Bar:** Graphical representation of capacity

### Auto-Assign Data

Provides insights into the assignment decision:
- **Status:** Assignment completion status
- **Assignment Explanation:** Detailed reasoning including:
  - Designer name and selection rationale
  - Due date analysis
  - Workday calculations
  - Available minutes breakdown
  - Execution timeline details

### Daily Schedule

Shows the designer's daily schedule with collapsible entries:
- **Schedule Date:** Current date being viewed
- **Task List:** All tasks scheduled for the day with:
  - Task ID
  - Task name
  - Department indicator
  - Estimated time

---

## Internal Comments

The Internal Comments page provides a dedicated space for team communication on tasks.

### Features

- **Comment Thread:** Chronological display of internal comments
- **Comment Input:** Text area for typing new internal comments with send button
- **User Information:** Each comment shows:
  - User name
  - Role (e.g., Graphic Designer)
  - Timestamp
  - Avatar
- **Rich Content:** Supports text and file attachments in comments
- **Request Feature:** Link at bottom to request new features

---

## Creative Tools

This page provides tools and actions specific to creative workflow management.

### Task Actions

Quick access buttons for common task operations:
- **Refresh Custom Fields:** Update task metadata
- **Add Back to Queue:** Return task to assignment pool
- **Block from Auto-Assignment:** Prevent automatic assignment (shown as disabled/inactive)

### Task Assignment Actions

Tools for managing creative assignments:
- **Generate Time Estimate:** Calculate task duration
- **Find Suitable Designer:** Locate available team members
- **Show Me Everything:** Expand to see all options

### Designer's Daily Schedule

View and manage the assigned designer's workload:
- **Schedule Header:** Shows designer name and workload statistics
  - Name (e.g., "Marcos Monti")
  - Time allocated vs. available (e.g., "500min / 420min (119%)")
- **Task List:** Detailed breakdown with columns:
  - **Task ID:** Unique identifier
  - **Name:** Task description
  - **Department:** Visual indicator (colored circle)
  - Sortable columns for organization

### Request Feature

Bottom link to submit feature requests for improvements.

---

## Designer Scheduling

The Designer Scheduling page provides tools for managing designer availability and assignments.

### Search and Filter

- **Search Bar:** Search designers by name or email
- **Refresh Button:** Update designer availability data
- **Add Button:** Add new designer to the system
- **Filter Tabs:** Filter by type:
  - All
  - Design
  - Video
  - Other

### Designer List

Scrollable list of all designers showing:
- **Avatar:** Profile picture or initials
- **Name:** Full designer name
- **Availability:** Number of days available (e.g., "5d")
- **Email:** Contact email address
- **Edit Button:** Modify designer information

### Features

- Real-time availability updates
- Quick access to edit designer schedules
- Department-specific filtering
- Availability indicators for easy capacity planning

### Request Feature

Link at bottom to request new scheduling features.

---

## SM Squad

The SM Squad (Social Media Squad) page manages account-specific information.

### Account Information Section

Key account details:
- **Coach:** Assigned coach name with clear/edit functionality
- **Bible Translation:** Dropdown to select Bible translation (with "Select bible translation..." placeholder)
- **Photos Link:** Link status (shows "Not set" if not configured)
- **SMS DB Link:** Database link for SMS tracking (shows full URL if set)

### Account Notes Section

- **Notes Text Area:** Large text field for adding account notes
  - Placeholder: "Add notes about this account..."
- **Status Indicator:** Shows "Up to date" when notes are saved

### Features

- Collapsible sections for better organization
- Direct links to external resources
- Quick edit functionality for all fields
- Request feature link at bottom

---

## Admin

The Admin page provides access to secure credentials and administrative functions.

### Vault Secrets

Secure credential management system:

#### Search and Add
- **Search Bar:** Search through stored secrets
- **Add Secret Button:** Create new secret entries

#### Secrets List

Displays all stored credentials with:
- **Secret Name:** Descriptive identifier
- **Info Button:** View secret details

**Example Secrets:**
- Airtable_api_key_id
- Full Access - Airtable
- ClickUp App Bearer Token
- supabase - squad-data password
- supabase - airtable-sync-data
- squad_crmn_key

### Security

- Credentials are stored securely
- View-only access with info buttons
- Organized alphabetically for easy reference

### Request Feature

Link at bottom to request new admin features.

---

## Squad Tools

The Squad Tools page provides access to utilities, branding resources, and reporting tools.

### Actions Section

Quick action buttons for common tasks:
- **Bug Report:** Submit bug reports with icon indicator
- **SIS Request:** Submit SIS-related requests

### Squad Colors

Comprehensive color palette for branding consistency:

#### Primary Colors
Grid of main brand colors with hex codes:
- #341756 (Deep Purple)
- #F9635D (Coral Red)
- #513DE5 (Royal Blue)
- #A960FF (Light Purple)
- #F2AF39 (Golden Yellow)
- #CC4291 (Magenta)

#### Secondary Colors
Supporting colors for design work:
- #BCB3F9 (Pale Purple)
- #DEC1FF (Lavender)
- #EBB8EC3 (Soft Pink)
- #FFDB9B (Pale Yellow)
- #FFA8A4 (Light Coral)
- #FFB48B (Peach)

#### Light Background Colors
Subtle colors for backgrounds:
- #FFFCF4 (Off White)
- #F3F1FF (Pale Lavender)
- #DCD8FA (Light Periwinkle)
- #DFDBFF (Soft Blue)
- #F5ECFF (Pale Purple)
- #FFE4F3 (Light Pink)
- Additional background options with hex codes

### Squad Assets

Brand asset management system:

#### File Search and Filter
- **Search Bar:** Search files by name
- **Type Dropdown:** Filter by asset type (logos, images, etc.)
- **Category Dropdown:** Filter by category

#### Asset Grid

Visual display of brand assets:
- **Asset Cards:** Thumbnail previews of files
- **File Names:** Descriptive names (e.g., "Badge Slanted - Blue-01.png")
- **Download Button:** Quick download icon on each card
- **Multiple Variants:** Different versions of logos and badges
  - Black versions
  - Blue versions
  - Slanted designs

### Features

- Collapsible sections for organized viewing
- Quick access to all brand materials
- Downloadable assets
- Searchable and filterable asset library

---

## Settings

The Settings page provides application configuration and user account management.

### App Settings Section

#### Theme
Dropdown to select application theme:
- Dark (default shown)
- Light (available)
- System (follows OS settings)

### Account Information Section

Displays current user details:
- **Username:** Full name and role (e.g., "Jacob Vendramin (R&D Engineer)")
- **Email:** User email address (e.g., "jacob@churchmediasquad.com")
- **Department:** User's department (e.g., "Systems Integration Squad")

### Changelog Section

Marked with "NEW" badge, provides update information:

#### New Features
- Toggle designer visibility by Design, Video Squad, and Other departments on Designer Scheduling page
- Cleaned up navigation menu for better visibility and easier navigation

#### Bugs Fixed
- Fixed Designer Scheduling page permissions
- Fixed Task Actions button layout on Creative Tools page
- Fixed Block from Auto-Assignment button positioning
- Fixed internal task viewing bug
- Cleaned up endpoints for SISX consistency
- Fixed Refresh Dropbox Folder endpoint bug

#### Version Information
- Current version displayed (e.g., "Version 1.6.3")

### User Actions

At bottom of settings:
- **Menu Navigation:** Access to all pages via hamburger menu
- **User Avatar:** Quick access to:
  - User profile
  - Refresh App Data
  - Sign Out (in red for visibility)

### Navigation Menu

Dropdown showing all available pages:
- Details
- Internal Comments
- Creative Tools
- Designer Scheduling
- SM Squad
- Admin
- Squad Tools
- Settings (with checkmark when active)

### Request Feature

Link at bottom to request new features.

---

## Navigation

The extension provides consistent navigation across all pages:

- **Hamburger Menu:** Access to all pages (bottom left)
- **User Avatar:** Quick access to account actions (bottom right)
- **Page Indicator:** Current page name displayed
- **Request Feature:** Available on all pages for user feedback

## Getting Started

1. Download the extension from the [Chrome Web Store](https://chromewebstore.google.com/detail/mysquad/accdcbpmoifgecjahfoabamdgjbnmnfb?authuser=0&hl=en)
2. Pin the extension to your Chrome toolbar
3. Click the MySquad icon to open
4. Navigate to any page using the menu at the bottom
5. Access page-specific features based on your role and permissions
