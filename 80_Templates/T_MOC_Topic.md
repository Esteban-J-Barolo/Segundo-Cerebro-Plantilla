---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tag: 
  - type/topic
<%*
let topic = String(tp.file.title).replace("MOC_","") || "";
  
if (!topic) return;

// escribir contenido del archivo
tR += `topic: ${topic}`;
%>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# 🧠 <% topic %>
```dataview
LIST
FROM "10_Library" AND #type/zettel
WHERE contains(topic, this.topic)
```

`button-delete-note`