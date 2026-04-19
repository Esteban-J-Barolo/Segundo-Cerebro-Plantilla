## 🏗️ Tareas y Tickets `button-create-task`
```dataview
TABLE status, priority, due_date
FROM #type/task
WHERE contains(file.outlinks, this.file.link) AND !contains(file.path, this.file.path)
SORT priority DESC, status ASC
```
---