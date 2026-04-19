---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags:
  - type/detail
detail: <% tp.file.title.split(" - ").slice(1).join(" - ") %>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# <% tp.file.title.split(" - ").slice(1).join(" - ") %>

## 📋 Descripción


---
## 🔗 Referencias
- 
---
`button-create-zettel` `button-delete-note`