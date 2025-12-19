---
title: "MySquad Chrome Extension"
author: "Jacob Vendramin"
date: "2025-12-16"
last_updated: "2025-12-19"
tags: ["chrome extension"]
version: "1.7.0"
emoji: "💻"
---

# MySquad Chrome Extension

> &nbsp;&nbsp;&nbsp;&nbsp;📥 &nbsp;&nbsp;**Download the MySquad Chrome Extension:** **[Chrome Web Store](https://chromewebstore.google.com/detail/mysquad/accdcbpmoifgecjahfoabamdgjbnmnfb?authuser=0&hl=en)**

Your all-in-one tool for managing tasks, assignments, and team operations right from your browser. Access everything you need without switching between tabs or applications.

---

## Getting Started

1. **Install:** Download the extension from the Chrome Web Store (link above)
2. **Login:** Click the extension icon and sign in with your ClickUp account using OAuth 2.0 authentication
3. **Access Task Details:** Navigate to any ClickUp task in your browser to see detailed information

The extension works with both regular and internal ClickUp tasks. For internal tasks, you'll see the actual task ID - not just the internal task ID - making it easier to reference and track tasks accurately.

### Smart Content Display

The extension intelligently manages content visibility:
- **30-Second Delay:** When you navigate away from a ClickUp task, content remains visible for 30 seconds with a countdown timer
- **Instant Activation:** Content appears immediately when you navigate to a task
- **Visual Indicators:** Color-coded status indicators (red = active task, yellow = delayed content)
- **Seamless Experience:** No need to manually refresh or reload task information

---

## Details

Your command center for task information. Open any ClickUp task and see everything important at a glance.

### Quick Task Info

See the essentials without scrolling through ClickUp:
- Task title, ID, status, and assigned department
- Direct link to the task's Dropbox folder (with quick copy button)
- Click-to-copy functionality for task name, task ID, and Dropbox path
- **"Currently Viewing" Badge:** Instantly see if you're assigned to the task you're viewing

### Task Management

**Quick Actions:**
- **Update Dropbox Folder:** Refresh the folder location when files move to ensure accurate linking
- **Refresh Custom Fields:** Sync latest data from ClickUp to get current field values
- **Return to Queue:** Send a task back for reassignment when priorities change
- **Block from Auto-Assignment:** Prevent specific tasks from being automatically scheduled
- **Favorite Task:** Save tasks to your portfolio (designers only) for quick access later

**Assignment Tools:**
- Generate time estimates automatically based on task requirements
- Find available designers who can take the task based on capacity
- View all assignment options at once with detailed availability information

### Account Information

Check account details and capacity:
- Account name and member number
- Active subscription plan and add-ons (scrollable view for accounts with many products)
- Current task load (e.g., "3 active tasks / 7 capacity")
- Member since date to understand account longevity
- Queued tasks count for workload planning

### Assignment Intelligence

Understand why a task was assigned to someone:
- See the complete logic behind each assignment decision
- View workload calculations and available time
- Check how assignments fit into the designer's schedule
- Auto-assignment status and queue position
- Detailed explanation in markdown format

### Designer Schedule

See what's on the assigned designer's plate:
- All tasks scheduled for the day with accurate due dates
- Time estimates for each task (e.g., "60min")
- Department indicators to spot workload balance
- Total time commitments with capacity percentage

---

## Internal Comments

Keep team conversations organized and accessible. View and respond to internal task comments without leaving your current page.

**What You Can Do:**
- Read all internal comments on a task in chronological order
- Add new comments with text and file attachments
- See who said what and when, with names, roles, and timestamps
- Keep discussions in one place so nothing gets lost
- **Delete Automated Comments:** Users with Creative Tools access can remove system-generated comments

**Note:** This page is automatically hidden when viewing internal tasks to streamline the interface.

---

## Creative Tools

Your workflow management hub for creative tasks.

### Task Actions

Handle common scenarios quickly:
- Sync latest custom field data from ClickUp
- Send tasks back to the queue for reassignment
- Prevent specific tasks from being auto-assigned
- Delete automated system comments for cleaner task threads

### Assignment Management

- Calculate accurate time estimates based on task complexity
- Find the right designer for the job using availability data
- See all available assignment options with detailed schedules

### Workload Visibility

Check the assigned designer's full schedule:
- See their daily task list with all commitments
- View time commitments with capacity percentage (e.g., "500min / 420min (119%)")
- Identify overbooked days at a glance with color coding
- Sort and organize tasks by ID, name, or department
- Real-time updates when designer schedules change

---

## Designer Scheduling

Manage your team's capacity and availability in one place.

**Find Designers Fast:**
- Search by name or email with instant filtering
- Filter by department (Design Squad, Video Squad, Other)
- Toggle between departments for focused views
- See availability at a glance (e.g., "5d" = 5 days available this week)

**Manage Schedules:**
- **Edit Schedules:** Update individual designer availabilities
- **Add New Designers:** Onboard team members with initial availability
- **Set Allotments:** Configure standard and priority schedule buffers (for future auto-assignment features)
- **Copy Days:** Duplicate one day's availability across the week
- **Clear All:** Reset all availabilities at once
- **Task Type Indicators:** See what types of work each designer is eligible for (Design, Video, etc.)
- **Refresh Data:** Sync latest availability information
- **View Contact Info:** Access email addresses for quick communication

**Permissions:**
- View-only access for most team members
- Edit access for authorized schedulers
- Department-based visibility controls

**Why It's Useful:**
No more guessing who's available or checking multiple calendars. See your entire team's capacity in seconds and make informed assignment decisions.

---

## My Design Favorites

Build and showcase your design portfolio with ease.

**Portfolio Management:**
- **Favorite Tasks:** Save tasks you're assigned to by clicking the star icon on the Details page
- **Search & Filter:** Find favorites by task name, church name, or account number
- **Task Enrichment:** See full task details including church name and account number
- **Currently Viewing Badge:** Quickly identify which favorited task you're currently looking at
- **Remove Favorites:** Clean up your portfolio by removing completed or unwanted tasks

**How It Works:**
- Only shows favorites for your account (private to each designer)
- Automatically fetches and caches task details for faster loading
- Updates in real-time when you add or remove favorites
- Scroll indicators for long lists of favorited tasks

**Why It's Useful:**
Designers can build a portfolio of their best work, making it easy to reference completed projects during reviews or showcase their design capabilities to the team.

---

## SM Squad

Centralize account information for the Social Media Squad.

**Account Details:**
- Assign and update account coaches with dropdown selection
- Select the correct Bible translation from available options
- Link to photo libraries and SMS databases
- Add detailed account notes that save automatically
- View primary email, Facebook, Instagram, and website links
- Access Notion internal notes and dashboard links
- See latest sermon recap information

**Stay Organized:**
All account-specific information in one place means less searching through emails or shared docs. Everyone on the team can access what they need instantly.

**Note:** This page is automatically hidden when viewing internal tasks.

---

## Admin

Secure access to team credentials and API keys.

**Vault Secrets:**
Store and access sensitive information safely:
- Search through all stored credentials with instant filtering
- Add new secrets as needed with proper categorization
- View credentials with one click (password protected)
- Edit existing secrets to keep information current

**Common Secrets:**
- API keys (Airtable, ClickUp, Supabase, Notion)
- Database passwords and connection strings
- Integration tokens (OAuth, webhooks, etc.)
- Third-party service credentials

**Security Note:**
All credentials are encrypted and stored securely in Supabase. Only authorized Systems Integration Squad members can access the Admin vault.

---

## Squad Tools

Everything you need for branding, reporting, and quick actions.

### Report Issues

Submit feedback and get help:
- **Report Bug:** Send detailed bug reports directly to the development team
- **Request Help:** Contact SIS for technical issues or feature requests
- Links open directly to appropriate communication channels

### Brand Colors

Never guess hex codes again. Access our complete color palette:
- **Primary Colors:** Main brand colors for logos and headers
- **Secondary Colors:** Supporting colors for designs
- **Background Colors:** Subtle colors for layouts
- **One-Click Copy:** Click any color to copy its hex code to clipboard

All colors are organized by category with visual swatches and hex codes displayed.

### Brand Assets

Download logos, badges, and graphics instantly:
- Search by filename with real-time filtering
- Filter by type or category (logos, badges, icons, etc.)
- Preview thumbnails before downloading
- Access multiple color variants (black, blue, white, etc.)
- Direct download links for quick asset retrieval
- High-resolution files for professional use

**Why It's Useful:**
Designers and marketers can grab the exact asset they need in seconds, ensuring brand consistency across all projects.

---

## Settings

Customize your experience and stay updated.

### Appearance

Choose your preferred theme:
- **Dark:** Easy on the eyes for long work sessions with reduced blue light
- **Light:** Traditional bright interface for high-contrast visibility
- **System:** Automatically match your computer's theme preferences

Theme changes apply instantly across the entire extension.

### Your Account

View your profile information:
- Name and role within Church Media Squad
- Email address linked to your ClickUp account
- Department assignment (determines page visibility)
- ClickUp user ID for troubleshooting

### What's New

Stay informed about updates:
- See new features as they're released with detailed descriptions
- Read about bug fixes and improvements
- Check the current version number (displayed in header)
- Access full changelog with version history

### Account Actions

Manage your session and data:
- **Refresh App Data:** Clear cache and sync latest information from ClickUp
- **Sign Out:** Safely log out when switching accounts or ending your session

**Note:** Signing out clears all cached data and navigation preferences for security.

---

## Navigation Tips

**Quick Access:**
- Click the dropdown menu (bottom left) to switch between pages
- Click your avatar (bottom right) for account options and quick actions
- Use "Request a feature" links throughout the app to suggest improvements
- Look for the ClickUp indicator (top right) to see task detection status

**Finding What You Need:**
- **Task info?** → Details page (when viewing a ClickUp task)
- **Team capacity?** → Designer Scheduling
- **Brand assets?** → Squad Tools
- **Account details?** → SM Squad
- **Credentials?** → Admin (SIS only)
- **Portfolio?** → My Design Favorites (Designers)
- **Comments?** → Internal Comments

**Department-Based Access:**
Different departments see different pages based on their role:
- **All Departments:** Details, Squad Tools, Settings
- **Creative Departments:** Creative Tools, Designer Scheduling
- **Designers:** My Design Favorites
- **SM Squad:** SM Squad page
- **SIS:** Admin page (vault access)

---

## Technical Details

### Architecture

**Built With:**
- **WXT Framework:** Modern Chrome extension framework with Vite
- **React:** Component-based UI with TypeScript
- **Tailwind CSS:** Utility-first CSS with custom design tokens
- **Untitled UI:** Professional component library
- **Supabase:** PostgreSQL database with real-time capabilities

**Key Features:**
- OAuth 2.0 authentication with ClickUp
- Secure token storage in browser extension storage
- 5-minute data caching for improved performance
- Real-time URL monitoring and task detection
- Department-based role access control
- Automatic cache cleanup and data refreshing

### Authentication Flow

1. **Sign In:** User clicks "Sign in with ClickUp" button
2. **OAuth Window:** Opens ClickUp authorization in new tab
3. **Authorization:** User authorizes the app in ClickUp
4. **Callback:** ClickUp redirects with auth code
5. **Success Page:** Beautiful "Authentication Complete" page with 5-second countdown
6. **Auto-Close:** Window automatically closes
7. **Token Exchange:** Extension exchanges auth code for access token
8. **User Data:** Fetches user profile from ClickUp API
9. **Display:** Shows personalized greeting with username

**Security Features:**
- State parameter prevents CSRF attacks
- Tokens stored securely in browser extension storage
- Automatic cleanup of auth codes after use
- Comprehensive error handling and user feedback

### Data Management

**Caching Strategy:**
- 5-minute cache duration for all data types
- Automatic expired cache cleanup
- Cache invalidation on data updates
- Separate caches for projects, accounts, and designer schedules

**Data Sources:**
- ClickUp API v2 for task and user data
- Supabase RPCs for enriched task information
- Custom services for specialized data processing

### Configuration System

**Global Banners:**
The extension supports dynamic global banners configured via Supabase:
- **Live Status:** Show/hide banners instantly
- **End Date:** Automatically hide expired banners
- **Department Targeting:** Show banners to specific departments or all users
- **Link Types:** Internal page navigation (`ext_page`) or external URLs (`url`)
- **Dismissible:** Users can dismiss banners for their session

**Component Visibility:**
Each page component has configurable visibility based on:
- User department assignment
- Feature flags for gradual rollouts
- Role-based permissions

---

## Troubleshooting

### Common Issues

**Content Not Appearing:**
- Check if ClickUp task URL is properly detected (look for indicator)
- Verify you're signed in with correct ClickUp account
- Try refreshing the extension data from Settings

**Authentication Issues:**
- Clear browser cache and extension storage
- Disable popup blockers for ClickUp domain
- Verify OAuth credentials are current

**Performance Issues:**
- Clear cached data from Settings → Refresh App Data
- Check for multiple extension instances running
- Verify internet connection is stable

**Missing Pages:**
- Some pages are department-specific (check your role)
- Internal tasks hide certain pages automatically
- Contact SIS if you believe you should have access

### Debug Mode

For troubleshooting, enable debug logging:
- Open browser console (F12)
- Look for `[useBanner]`, `[Fetch Service]`, or `[BannerService]` prefixed logs
- Check ClickUp indicator for detailed state information

### Getting Help

**Report Issues:**
- Use Squad Tools → Report Bug for technical issues
- Include screenshots and steps to reproduce
- Check What's New in Settings for known issues

**Feature Requests:**
- Use "Request a feature" links throughout the app
- Provide detailed descriptions of desired functionality
- Include use cases and expected benefits

---

## Version History

<details>
<summary><strong>Version 1.7 (2025-12-19)</strong></summary>

## New Features
- Update for users with Creative Tools access to delete automated comments from tasks
- Update for designers to be able to favorite a task they are assigned to in the extension for portfolio building (accessible in the My Design Favorites page)
- There is now a badge on the details page that tells you if you are viewing a task in which you are assigned to at a glance
- Added ability to set standard and priority schedule allotments on the Designer Scheduling page (not in use yet but added for future needs)

## Bugs Fixed
- Fixed a bug where it was not updating the designer daily schedule section on the Creative Tools page in a timely manner

</details>

<details>
<summary><strong>Version 1.6 (2025-11-12)</strong></summary>

## New Features
- On Designer Scheduling page you can now toggle by Design and Video Squad and Other departments making it easier to see who you are viewing
- Cleaned up the navigation menu in the bottom left corner to increase visibility and make it more obvious to users how to navigate pages

## Bugs Fixed
- Fixed permissions on the Designer Scheduling page for who is allowed to edit schedules, who is view only and who does not see the page at all
- Fixed bug where Task Actions buttons on Creative Tools page were all side by side when they were supposed to be stacked causing the "Block from Auto-Assignment" button to be outside the parent container
- Fixed a bug where users could not view internal tasks
- Cleaned up all endpoints used to go through SISX only for consistency and reliability
- Fixed a bug where it was using the wrong endpoint for Refresh Dropbox Folder

</details>

<details>
<summary><strong>Version 1.5 (2025-10-23)</strong></summary>

## New Features
- CX Squad can now block tasks that are not yet assigned from being auto-scheduled using the Block button on the Creative Tools page

## Bugs Fixed
- Fixed bug where API calls were being made multiple times on load

</details>

<details>
<summary><strong>Version 1.4 (2025-10-10)</strong></summary>

## New Features
- CX Squad page renamed to "Creative Tools"
- On "Details Page" there is now a "Refresh Dropdown" button to ensure the currently viewing project's folder is in the correct place
- New "Designer Scheduling" Page now allows for viewing, adding and editing daily availabilities for designers (for use with the auto-assigner)
- When adding a new Designer Schedule, it now shows what types of tasks they are eligible for (eg. Design, Video, etc)
- When adding a new Designer Schedule, you can now copy the availability of a day to all other days and clear all availabilities in one click

## Bugs Fixed
- Renamed "Load Custom Fields" to "Refresh Custom Fields" for better clarity

</details>

<details>
<summary><strong>Version 1.3 (2025-09-30)</strong></summary>

## New Features
- Added a "Load Custom Fields" button to the Project Details page to allow users to load custom fields for a task
- Added a manual refresh option that can be found by opening up the user profile popup in the bottom right corner
- Squad tools page now has hex codes for all Squad brand colors for easy copy to clipboard
- Click to copy added to these fields: Task name, Task ID, and Dropbox path on the Project Details page
- Rearranged the Settings Page for a better user experience
- Added a "Changelog" section to the Settings Page so users can see bug fixes and new features added to the latest extension version
- Added the ability to show a badge in each section's header for quick at a glance data visibility
- Added Project Details (minimized by default) to the top of CX Squad Page to provide quick access to project details without needing to navigate back to the main Details Page

## Bugs Fixed
- Fixed bug where UI wouldn't reliably update when the user navigates to a new task in ClickUp
- Fixed bug in CX Squad page were Load Custom Field button was always clickable, however it should be disabled when task has no tags
- Fixed bug where the Dropbox path did not show in Project Details for a task when it should
- Fixed bug where Dropbox path was only fully visible when expanding the extension width an extreme amount when path was very long (element is now scrollable horizontally)
- Fixed bug where "Auto-Assign Data" section in the "Project Details" page was not open by default
- Fixed bug where status badge went to 2 lines when the extension width was shrunk down making it hard to read and an eye sore
- Fixed bug on CX page for the Find Suitable Designer functionality where the pagination component was too wide for the container
- Fixed bug on tool that shows designer's scheduling for a certain day where it was sending today's date instead of the correct due date
- Fixed bug causing admin secrets vault to be closed by default (SIS access only)
- Fixed an inconsistency where some pages had headings and others didn't
- Fixed bug when account had a lot of plans and add ons it became too wide and exceeded the width of the container - now area is scrollable so user can view all plans and add ons
- Fixed bug where loading indicator on "Generate Time Estimate" button was not visible
- Fixed bug where Exec Squad could not see CX Squad page
- Fixed bug where "Designer Schedule" on CX Page was showing the incorrect date despite the actual data shown being correct

</details>

---

## License

This project is proprietary to Church Media Squad.

---

*For technical support or feature requests, contact the Systems Integration Squad.*
