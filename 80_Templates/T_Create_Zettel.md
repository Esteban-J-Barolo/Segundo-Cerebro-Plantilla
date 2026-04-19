---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
status: Evergreen
tags:
  - type/zettel
<%*
const FOLDER = "10_Library";

const files = app.vault.getMarkdownFiles()
  .filter(f => f.path.startsWith(FOLDER));

let topics = new Set();

for (const file of files) {
  const cache = app.metadataCache.getFileCache(file);
  const fm = cache?.frontmatter;

  if (fm?.topic) {
    if (Array.isArray(fm.topic)) {
      fm.topic.forEach(t => topics.add(String(t)));
    } else {
      topics.add(String(fm.topic));
    }
  }
}

let topicList = Array.from(topics).sort();

// opción para crear uno nuevo
topicList.unshift("➕ Nuevo topic");

const selected = await tp.system.suggester(
  topicList,
  topicList
);

let finalTopic;

if (selected === "➕ Nuevo topic") {
  finalTopic = await tp.system.prompt("Nombre del nuevo topic");
} else {
  finalTopic = selected;
}

// fallback de seguridad
finalTopic = finalTopic?.trim() || "General";

tR += `topic:\n  - ${finalTopic}\n`;
%> 
<% tp.file.include("[[script_detectar_origen]]") %>
---
# 🧠 Idea: <% tp.file.title %>
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Evergreen), option(Developing), option(Deprecated)):status]`  
> 
> **Topic**: `BUTTON[btn-add-topic]`  

---
## Concepto


---
## Notas relacionadas
`button-create-zettel`
```dataview
LIST
FROM "10_Library"
WHERE origin = this.file.link
```
---
`button-delete-note`