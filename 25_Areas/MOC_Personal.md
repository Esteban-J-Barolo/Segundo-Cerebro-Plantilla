---
created: 2026-03-29 16:42
tags:
  - type/area
area: personal
origin: "[[Dashboard]]"
---
# Personal
## Tareas
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
FROM "00_Inbox"
WHERE origin = this.file.link
```

`button-delete-note`