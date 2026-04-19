---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
status: Process
tags: 
  - type/inbox
<% tp.file.include("[[script_detectar_origen]]") %>
---
# 📥 Nueva Captura: <% tp.file.title.replace(tp.date.now("YYYY-MM-DD_HH-mm_"),"") %>
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Process), option(Review), option(OnHold), option(Dismissed)):status]`  

---
## 🖋️ Notas


---
## 🔗 Referencias
- 
---
`button-create-zettel` `button-delete-note`