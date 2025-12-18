---
title: "Postgres Fuzzy Matching"
author: "Jacob Vendramin"
date: "2025-12-18"
last_updated: "2025-12-18"
tags: ["supabase", "postgres", "sql"]
version: "1.0.0"
emoji: "📊"
---

# 🔍 Postgres Fuzzy Matching

## 🎯 Beginner's Guide: Quick Reference

### What is Fuzzy Matching?

Standard SQL `LIKE` requires an exact match. Fuzzy matching allows you to find strings that are **similar but not identical**. It is the industry standard for matching text that contains dynamic variables (placeholders), Markdown formatting, typos, or emojis.

### Comparison Operators

The `pg_trgm` extension provides two primary operators:

| Operator | Type | Logic | Best For |
| --- | --- | --- | --- |
| **`%`** | Boolean | Returns `TRUE` if similarity is above threshold | Filtering rows in a `WHERE` clause |
| **`<->`** | Numeric | Calculates "distance" (0 = identical, 1 = unique) | Sorting by "Best Match" in `ORDER BY` |

---

## 🛠️ Data Cleaning & Regex (Ignoring Emojis)

To get a high-quality match, you must "clean" both the database column and the input string. This ensures that formatting or emojis don't break your search.

### Handling Emojis and Symbols

Emojis and special characters often differ between platforms (e.g., Slack vs. ClickUp). To ignore them, we target **Non-ASCII** characters or **Non-Alphanumeric** characters.

| Goal | Regex Pattern | Description |
| --- | --- | --- |
| **Aggressive Clean** | `[^a-zA-Z0-9]+` | **Best for Emojis.** Removes everything except letters and numbers. |
| **Ignore Emojis Only** | `[^\x00-\x7F]+` | Strips all non-ASCII characters (emojis/symbols) but keeps standard punctuation. |
| **Normalize Space** | `\s+` | Replaces newlines and tabs with a single space to prevent spacing mismatches. |

---

## 📚 Technical Implementation

### The "Universal Matcher" Query

This query is designed to find a match even if your input is cluttered with emojis, Markdown, or unique user names.

```sql
SELECT 
  *, 
  -- Calculate distance for sorting (0.0 is a perfect match)
  (REGEXP_REPLACE(target_column, E'[^a-zA-Z0-9]+', '', 'g') <-> 
   REGEXP_REPLACE('{{ $json.input_text }}', E'[^a-zA-Z0-9]+', '', 'g')) AS distance
FROM your_table_name
WHERE 
  -- Use similarity operator to filter candidates
  REGEXP_REPLACE(target_column, E'[^a-zA-Z0-9]+', '', 'g') % 
  REGEXP_REPLACE('{{ $json.input_text }}', E'[^a-zA-Z0-9]+', '', 'g')
ORDER BY distance ASC
LIMIT 1;

```

---

## ⚙️ Configuration & Performance

### Adjusting Strictness

The "forgiveness" of the match is controlled by the `similarity_threshold`. You can adjust this for your current session:

* **Lower (0.1 - 0.2):** Very forgiving. Matches strings with massive differences or many emojis.
* **Higher (0.6 - 0.9):** Very strict. Requires strings to be almost identical.

```sql
-- View current threshold
SHOW pg_trgm.similarity_threshold;

-- Set new threshold (Session-based)
SET pg_trgm.similarity_threshold = 0.3;

```

### Speeding Up Large Tables

For tables with more than 5,000 rows, use a **GIST** index. This prevents Postgres from having to scan every single row (Sequential Scan) to find a match.

```sql
-- Create an index to make fuzzy searches lightning fast
CREATE INDEX idx_fuzzy_search ON your_table_name USING gist (target_column gist_trgm_ops);

```

---

## 🎓 Common Use Cases

1. **Template Identification**: Finding a specific canned response when the input has unique user names or emojis (`👋`) inserted.
2. **Formatting Agnostic Search**: Matching plain text against data stored in Markdown (`**bold**`) or HTML (`<b>bold</b>`).
3. **Cross-Platform Matching**: Linking data between platforms where formatting and emoji sets differ but the core text is the same.
4. **Data Deduplication**: Identifying "John Doe 👋" and "John Doe" as the same record.
