
```button
name ➕ Nueva Área
type note(25_Areas/MOC_<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.system.prompt("¿Qué área es esta?")) %>) template
action T_MOC_Area
templater true
```
^button-new-area

```button
name 📝 Nota Rápida
type note(00_Inbox/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.date.now("YYYY-MM-DD_HH-mm_") ) %><% tp.system.prompt("¿De qué trata esta nota?") %>) template
action T_Create_Inbox_Note
templater true
```
^button-new-inbox-note

```button
name ➕ Nueva Proyecto
type note(20_Projects/MOC_<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.system.prompt("¿Qué proyecto es este?") ) %>) template
action T_MOC_Project
templater true
```
^button-new-project

```button
name ➕ Nueva Contexto
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.project ) %> - <% tp.system.prompt("¿Qué contexto es este?") %>) template
action T_Create_Context
templater true
```
^button-new-context

```button
name 🧠 Crear Nota Zettel
type note(10_Library/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.system.prompt("Atomic Idea Title") ) %>) template
action T_Create_Zettel
templater true
```
^button-create-zettel

```button
name 🗑️ Eliminar la nota actual
type command
action Eliminar archivo actual
class obsidian-button
color red
```
^button-delete-note

```button
name 💡 Create MOC
type note(10_Library/MOC_<%* window._origenNota = app.workspace.getActiveFile()?.basename; const ZF="10_Library"; const flat=(v)=>[].concat(v||[]); const files=app.metadataCache.getCachedFiles().filter(f=>f.startsWith(ZF)); const mocs=new Set(); const topics=new Set(); files.forEach(f=>{const m=app.metadataCache.getCache(f)?.frontmatter; if(m){const ts=flat(m.tag).concat(flat(m.tags)); const tps=flat(m.topic); if(ts.some(t=>t&&t.includes("type/topic"))) tps.forEach(t=>mocs.add(t)); tps.forEach(t=>topics.add(t));}}); const list=Array.from(topics).filter(t=>!mocs.has(t)).sort(); let res=app.workspace.activeEditor?.editor.getSelection()||(await tp.system.suggester(t=>t,list)); if(res) tR+=res; %>) template
action T_MOC_Topic
templater true
```
^button-create-moc-topic

```button
name 🧠 Crear Capacitación
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.area ? tp.frontmatter.area + " - " : "" ) %>Training - <% tp.system.prompt("Nombre de la Capacitación (Ej: Curso Docker)") %>) template
action T_Create_Training
templater true
```
^button-create-training

```button
name 🛠️ Crear Tarea
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.project ) %> - <% (await tp.system.prompt("Título de la Tarea")).replace(/[\\/:*?"<>|#^\[\]]/g, "").replace(/\(/g, "{").replace(/\)/g, "}").replace(/\s+/g, " ").trim() %> ) template
action T_Create_Task
templater true
```
^button-new-task

```button
name 🛠️ Crear Tarea
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.project ) %> - <% (await tp.system.prompt("Título de la Tarea")).replace(/[\\/:*?"<>|#^\[\]]/g, "").replace(/\(/g, "{").replace(/\)/g, "}").replace(/\s+/g, " ").trim() %> ) template
action T_Create_Task_Serfe
templater true
```
^button-new-task-serfe

```button
name 🛠️ Crear Sub Tarea
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.project) %> - <% (await tp.system.prompt("Título de la Tarea")).replace(/[\\/:*?"<>|#^\[\]]/g, "").replace(/\(/g, "{").replace(/\)/g, "}").replace(/\s+/g, " ").trim() %> ) template
action T_Create_Sub_Task
templater true
```
^button-new-sub-task

```button
name ➕ Agregar Detalle
type note(20_Projects/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.frontmatter.project ) %><% tp.frontmatter.ticket_id ? (" - " + (tp.frontmatter.ticket_id.includes("/") ? tp.file.title.split(" - ")[1] : tp.frontmatter.ticket_id)) : "" %> - <% ( await tp.system.prompt("¿De qué trata esta nota?")).replace(/[\\/:*?"<>|#^\[\]]/g, "").replace(/\(/g, "{").replace(/\)/g, "}").replace(/\s+/g, " ").trim() %>) template
action T_Create_Detail
templater true
```
^button-add-detail

```button
name ⏳ Nuevo Registro
type note(00_Inbox/<% (window._origenNota = app.workspace.getActiveFile()?.basename, tp.date.now("YYYY-MM-DD") ) %>_Routine) template
action T_Create_Routine_Log
templater true
```
^button-routine-log

```button
name ➕ Traducir Clipboard
type append template
action Scripts/T_Traductor_a_Tracker
templater true
```
^button-traducir-tracker