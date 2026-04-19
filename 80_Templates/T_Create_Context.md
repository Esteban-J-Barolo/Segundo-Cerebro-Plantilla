---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags:
  - type/context
project: <% tp.file.title.split(" - ")[0].toLowerCase() %>
context: <% tp.file.title.split(" - ").pop() %>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# <% tp.file.title.split(" - ").pop() %>
---
## Descripción


---
## Detalles 
`button-add-detail`
```dataview
LIST WITHOUT ID
	link(file.link, detail)
FROM "20_Projects" AND #type/detail 
WHERE origin = this.file.link
```
---
##  🏔️ Contextos
`button-new-context`
```dataview
LIST WITHOUT ID
	link(file.link, context)
FROM "20_Projects" AND #type/context
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