# Squad Documentation Repository

This repository contains all documentation for The SQD workflows, processes, and systems. Documentation is automatically displayed on the Squad Wiki at https://wiki.thesqd.com.

## Supported Markdown Formatting

The Squad Wiki supports GitHub Flavored Markdown with additional features for enhanced documentation.

### Basic Formatting

#### Headings

```markdown
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

**Notes:**
- All headings automatically get anchor links for easy sharing
- Heading IDs are generated from the heading text (slugified)

#### Text Formatting

```markdown
**Bold text**
*Italic text*
***Bold and italic***
~~Strikethrough~~
`Inline code`
```

#### Lists

**Unordered lists:**
```markdown
- Item 1
- Item 2
  - Nested item 2.1
  - Nested item 2.2
- Item 3
```

**Ordered lists:**
```markdown
1. First item
2. Second item
3. Third item
```

**Task lists:**
```markdown
- [x] Completed task
- [ ] Incomplete task
- [ ] Another task
```

### Links

```markdown
[Link text](https://example.com)
[Internal link](./folder/document.md)
```

**Notes:**
- External links (starting with http/https) automatically open in new tabs
- Internal links should use relative paths to other markdown files

### Images

```markdown
![Alt text](image-url.png)
![Screenshot](./images/screenshot.png)
```

### Code Blocks

**Inline code:**
```markdown
Use `variable` for inline code
```

**Code blocks with syntax highlighting:**

````markdown
```javascript
function example() {
  console.log("Hello World");
}
```

```python
def example():
    print("Hello World")
```

```bash
npm install
git commit -m "Update docs"
```
````

**Supported languages include:**
- javascript, typescript, jsx, tsx
- python
- bash, sh, shell
- sql
- json, yaml, xml
- css, scss
- html
- And many more via highlight.js

### Tables

```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Row 1    | Data     | More     |
| Row 2    | Data     | More     |
```

**With alignment:**
```markdown
| Left aligned | Center aligned | Right aligned |
|:-------------|:--------------:|--------------:|
| Left         | Center         | Right         |
```

### Blockquotes

```markdown
> This is a blockquote
> It can span multiple lines
```

**Nested blockquotes:**
```markdown
> First level
> > Second level
> > > Third level
```

## Advanced Features

### Collapsible Sections

Create collapsible/expandable sections using HTML `<details>` and `<summary>` tags:

```markdown
<details>
<summary>Click to expand</summary>

Content goes here. You can use **any markdown** inside:

- Lists
- Code blocks
- Tables
- etc.

</details>
```

**Example:**
```markdown
<details>
<summary>Database Schema</summary>

## Users Table

| Column | Type | Description |
|--------|------|-------------|
| id     | uuid | Primary key |
| email  | text | User email  |

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT NOT NULL
);
```

</details>
```

**Best practices:**
- Use collapsible sections for detailed information that not everyone needs to see
- Put the summary text on the same line as the `<summary>` tag
- Leave blank lines around the content inside `<details>`
- Perfect for long code snippets, detailed configurations, or troubleshooting steps

### Raw HTML

The wiki supports raw HTML when needed:

```markdown
<div style="background: #f0f0f0; padding: 20px;">
  Custom HTML content
</div>
```

**Use cases:**
- Custom styling for special sections
- Embedding iframes or videos
- Complex layouts not possible with markdown

**Warning:** Use HTML sparingly to keep documents maintainable.

### Horizontal Rules

```markdown
---
```

Creates a horizontal line separator.

## File Organization

### Naming Conventions

- Use lowercase for all file and folder names
- Use hyphens (-) instead of spaces or underscores
- Use descriptive names: `database-schema.md` not `db.md`
- Group related docs in folders

### Folder Structure

```
docs/
├── workflows/
│   ├── clickup-automation.md
│   └── task-management.md
├── integrations/
│   ├── n8n-setup.md
│   └── supabase-functions.md
├── processes/
│   └── onboarding.md
└── README.md
```

## Writing Guidelines

### Document Structure

Every documentation file should follow this structure:

```markdown
# Document Title

Brief description of what this document covers.

## Table of Contents (optional for long docs)

- [Section 1](#section-1)
- [Section 2](#section-2)

## Section 1

Content here...

## Section 2

Content here...

## Related Documents

- [Related Doc 1](./path/to/doc.md)
- [Related Doc 2](./path/to/doc.md)
```

### Best Practices

1. **Start with context**: Begin each document with a brief explanation of what it covers
2. **Use descriptive headings**: Make headings searchable and descriptive
3. **Include examples**: Show code examples and screenshots where helpful
4. **Keep it updated**: Update docs when processes change
5. **Link related docs**: Help readers find related information
6. **Use collapsible sections**: For detailed or optional information
7. **Add code comments**: Explain complex code snippets
8. **Test your links**: Ensure all internal links work

### Voice and Tone

- Write in clear, simple language
- Use active voice: "Click the button" not "The button should be clicked"
- Be direct and concise
- Use "you" to address the reader
- Avoid jargon unless necessary (and explain it when used)

## Examples

### Complete Example Document

````markdown
# N8N Workflow Setup

This guide explains how to create and configure N8N workflows for automating ClickUp tasks.

## Prerequisites

Before starting, ensure you have:
- [x] Access to the N8N instance
- [x] ClickUp API credentials
- [ ] Supabase connection configured

## Creating a New Workflow

1. Navigate to N8N dashboard
2. Click **New Workflow**
3. Name your workflow descriptively

<details>
<summary>Detailed Steps with Screenshots</summary>

### Step 1: Access the Dashboard

Navigate to `https://n8n.thesqd.com` and log in.

![Dashboard Screenshot](./images/n8n-dashboard.png)

### Step 2: Configure Trigger

Select a trigger node and configure it:

```json
{
  "trigger": "webhook",
  "path": "/clickup-task-update"
}
```

</details>

## Common Configurations

### Webhook Trigger

```javascript
// Example webhook payload
{
  "event": "taskCreated",
  "task_id": "abc123",
  "list_id": "xyz789"
}
```

### Supabase Query

```sql
INSERT INTO tasks (task_id, status, created_at)
VALUES ($1, $2, NOW());
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Webhook not firing | Check webhook URL and authentication |
| Query failing | Verify Supabase credentials |

## Related Documentation

- [ClickUp Integration](./clickup-integration.md)
- [Supabase Functions](./supabase-functions.md)
````

## Need Help?

- Search existing docs using the wiki search
- Ask in the #documentation channel
- Contact the Systems Integration Squad

---

**Last Updated:** December 2025
