---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags:
  - type/area
area: <% tp.file.title.replace("MOC_", "").toLowerCase() %>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# <% tp.file.title.replace("MOC_","") %>
```dataviewjs
const area = dv.current().area; // Lee el atributo 'area' del YAML de esta nota

dv.paragraph("```todoist\n" +
`name: "Mis tareas del área ${area}"\n` +
`filter: "##Obsidian & @${area}"\n` +
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
<%*
try {
  // Leer origen que ya detectó script_detectar_origen
  const origenNombre = window._origenNota || 
    tp.frontmatter?.origin?.replace(/[\[\]"]/g, "").trim() || 
    "Desconocido";

  const token = (await app.vault.adapter.read(".obsidian/todoist-token")).trim();
  const raw = tp.file.title.replace("MOC_", "").toLowerCase();
  const labelName = raw.charAt(0).toUpperCase() + raw.slice(1);

  // Obtener proyecto Obsidian
  const syncRes = await requestUrl({
    url: "https://api.todoist.com/api/v1/sync",
    method: "POST",
    headers: { "Authorization": `Bearer ${token}` },
    contentType: "application/x-www-form-urlencoded",
    body: "sync_token=*&resource_types=%5B%22projects%22%5D"
  });

  const project = syncRes.json.projects.find(p => p.name === "Obsidian");
  if (!project) {
    new Notice("Error: No se encontró el proyecto Obsidian");
    return;
  }

  // Siempre crear etiqueta
  const commands = [{
    type: "label_add",
    temp_id: crypto.randomUUID(),
    uuid: crypto.randomUUID(),
    args: { name: labelName }
  }];

  // Solo crear sección si el origen es Dashboard
  if (origenNombre === "Dashboard") {
    commands.push({
      type: "section_add",
      temp_id: crypto.randomUUID(),
      uuid: crypto.randomUUID(),
      args: { name: labelName, project_id: project.id }
    });
  }

  await requestUrl({
    url: "https://api.todoist.com/api/v1/sync",
    method: "POST",
    headers: { "Authorization": `Bearer ${token}` },
    contentType: "application/x-www-form-urlencoded",
    body: `commands=${encodeURIComponent(JSON.stringify(commands))}`
  });

  const msg = origenNombre === "Dashboard"
    ? `✓ Etiqueta y sección "${labelName}" creadas en Todoist`
    : `✓ Etiqueta "${labelName}" creada en Todoist (sin sección, origen: ${origenNombre})`;

  new Notice(msg);

} catch(e) {
  new Notice("Error: " + e.message.substring(0, 100));
  console.error(e);
}
%>