# 📑 Obsidian Markdown Quick Reference

This note summarizes the formatting elements, syntax structures, and metadata blocks used across your data engineering interview preparation vault.

---

## 1. Document Architecture & Layout

### YAML Frontmatter (Metadata)
* Always placed at the **very top** of the file, wrapped between two sets of triple dashes (`---`).
* Used to store variables that search plugins (like Dataview) can query.
```yaml
---
company: "Acme Corp"
status: "Technical Screening"
applied_date: 2026-07-22
---
```

### Horizontal Rules (Dividers)
* Created using three dashes `---` on a blank line. Useful for cleanly breaking up interview logs or sections.

---

## 2. Text Formatting & List Elements

### Visual Anchors (Emojis)
* Use headers paired with a functional emoji (e.g., `# 🗄️ Data Modeling`) to create a distinct visual scan path when scrolling through complex technical text.

### Interactive Task Lists
* Combine brackets with a space `[ ]` for an uncompleted checkbox, and an `[x]` for a completed task.
- [ ] Review Window Functions
- [x] Complete Python Generator Practice

### Informative Tables
* Formatted using pipes (`|`) to separate columns and dashes (`-`) to establish a header block. Use colons (`:---`) for left alignment or (`:---:`) for centering text blocks.

| Concept   | Option A | Option B |
| :-------- | :------- | :------- |
| **Speed** | Fast     | Slow     |

---

## 3. Code Blocks & Syntax Highlighting

To display clean, readable technical snippets, wrap code inside triple backticks (\`\`\`) and append the programming or structural language name to the opening block.

### SQL Syntax Block
```sql
SELECT user_id, COUNT(*) 
FROM logs 
GROUP BY 1;
```

### Python Syntax Block
```python
def stream_data():
    yield "Data Point"
```

### Text / Diagram Block
```text
[Ingestion Pipeline] ➔ [Kafka Event Broker] ➔ [Target Storage]
```

---

## 4. Obsidian-Specific Connections

### Bi-directional Internal Links
* Created by wrapping a target note title in double square brackets `[[Note Name]]`. 
* **The Magic:** Clicking this text instantly navigates to that sheet. If the note does not exist yet, Obsidian creates a blank template for it automatically.
* *Example:* Make sure to study [[SQL Optimization Principles]] before tomorrow.

### Custom Link Display Names
* Use a pipe (`|`) inside the brackets to show a cleaner title in your text while still pointing to the exact same backend file name.
* *Example:* Read this [[STAR - Optimizing High Throughput Data Ingestion|Ingestion Outage Story]].

### Data View
- The `Dataview` plug-in allows you to query files in your vault to produce custom dashboards. An example relevant to this vault is building a job search dashboard that lists jobs that you're actively applying to and jobs you've been rejected from.
- To use `Dataview`, install the community plug in and open a new note. The SQL-like query is placed in a `dataview` code block, like so:
```text
TABLE role AS "Position", status AS "Interview Stage", applied_date AS "Date Applied"
FROM "01_Job_Hunting" OR #interview
WHERE status != "❌ Rejected" AND status != "🎉 Offer Accepted"
SORT applied_date DESC  
```
- Active Pipeline Table: Scans your entire vault, looks for any note with a `status` in the YAML, filters out rejected or completed paths, and displays them in a structured table sorted by application date.
	- **Note:** The `text` code block above would need to be configured as a `dataview` code block for the query to produce a table. The primary key of the table is `File`.
```text
LIST company
FROM "01_Job_Hunting"
WHERE status = "❌ Rejected"
SORT applied_date DESC
```
- Rejected / Archive List: Keeps a history of companies you have engaged with, without cluttering your active pipeline grid.
#### How It Works (Connecting Metadata)
- Dataview relies explicitly on the text inside the triple dashes (`---`) at the top of your files. For the dashboard query above to populate your table automatically, ensure your individual company notes use consistent keys matching your template:
```yaml
---
company: "Snowflake"
role: "Data Engineer"
status: "Technical Screening"
applied_date: 2026-07-22
---

```
- Pro-Tips for Dashboard Success
	- **Dynamic Updates:** Whenever you update the `status:` line inside an individual company's file (e.g., changing it from `"Applied"` to `"Technical Screening"`), your master dashboard table will instantly recalculate and change its display the next time you open it.
	- **Clickable Links:** The first column of the `Dataview` table automatically generates an active hyperlink using the file's name, letting you click directly from your dashboard grid straight into that company's specific study log.

---

## 5. Helpful Plug-ins

### Core Plug-ins
1. Daily Notes:
	1. Create a custom template for a daily note, such as a job search / preparation activity summary.
	2. Configure the date (file name) format, new file location, and template file location.
	3. Create a daily note based on the custom template by clicking the "Open today's daily note" button.
2. Outline:
	1. Configure a hot key to show the table of contents of the current note.
	2. Press the hot key to show the outline on the right side of the screen.

### Community Plug-ins
1. Dataview
	1. Create complex data views (using SQL-like syntax) based on note metadata.