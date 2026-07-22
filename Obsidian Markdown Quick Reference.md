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
