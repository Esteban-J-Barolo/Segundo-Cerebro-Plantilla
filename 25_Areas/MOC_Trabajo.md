---
created: 2026-01-25 18:57
tags:
  - type/area
area: trabajo
origin: "[[Dashboard]]"
---
# Trabajo
```dataviewjs
const area = dv.current().area; // Lee el atributo 'area' del YAML de esta nota

dv.paragraph("```todoist\n" +
`name: "Mis tareas del área ${area}"\n` +
`filter: "##Obsidian & /${area} & @${area}"\n` +
"```");
```
`BUTTON[add-task-todoist]`

---
## Proyectos
`button-new-project`
```dataview
LIST WITHOUT ID
	link(file.link, upper(project))
FROM "20_Projects" AND #type/project
WHERE origin = this.file.link
```
---
##  🏔️ Áreas
`button-new-area`
```dataview
LIST WITHOUT ID
	link(file.link, upper(area))
FROM "25_Areas" AND #type/area
WHERE origin = this.file.link
```
---
# Notas
`button-new-inbox-note`
```dataview
LIST
FROM "00_Inbox" AND #type/inbox
WHERE origin = this.file.link
```
