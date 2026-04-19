---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
status: Active
priority: High
tags:
  - type/project
due_date:
project: <%* tR += tp.file.title.replace("MOC_", "").toLowerCase() %>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# <% tp.file.title.replace("MOC_","") %>
> [!ABSTRACT]- Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Active), option(OnHold), option(Completed), option(Archived)):status]`
> **Prioridad**: `INPUT[inlineSelect(option(High), option(Medium), option(Low)):priority]`
> **Deadline**: `INPUT[date:due_date]`
## Planificar tareas
```dataviewjs
const today = dv.date("today").toFormat("yyyy-MM-dd");
const prioOrder = { "High": 0, "Medium": 1, "Low": 2 };

const getTags = (p) => {
    if (!p.tags) return [];
    if (Array.isArray(p.tags)) return p.tags;
    if (p.tags.values) return p.tags.values;
    return [];
};

window.saveSecuencia = async (path, tareaId, valor, inputEl) => {
    const file = app.vault.getAbstractFileByPath(path);
    if (!file) return;
    let content = await app.vault.read(file);
    const lines = content.split("\n");
    for (let i = 0; i < lines.length; i++) {
        const partes = lines[i].split("|");
        if (partes.length < 7) continue;
        if (partes[1].trim() !== String(tareaId)) continue;
        partes[5] = ` ${valor} `;
        lines[i] = partes.join("|");
        break;
    }
    await app.vault.modify(file, lines.join("\n"));
    const flash = inputEl.parentElement.querySelector('.saved-flash');
    if (flash) { flash.style.opacity = '1'; setTimeout(() => flash.style.opacity = '0', 1000); }
};

window.saveFecha = async (path, tareaId, valor, inputEl) => {
    const file = app.vault.getAbstractFileByPath(path);
    if (!file) return;
    let content = await app.vault.read(file);
    const lines = content.split("\n");
    for (let i = 0; i < lines.length; i++) {
        const partes = lines[i].split("|");
        if (partes.length < 7) continue;
        if (partes[1].trim() !== String(tareaId)) continue;
        partes[4] = ` [fecha:: ${valor}] `;
        lines[i] = partes.join("|");
        break;
    }
    await app.vault.modify(file, lines.join("\n"));
    const flash = inputEl.parentElement.querySelector('.saved-flash');
    if (flash) { flash.style.opacity = '1'; setTimeout(() => flash.style.opacity = '0', 1000); }
};

const cargarSubtareas = async (notaPath) => {
    const subs = dv.pages('"20_Projects"')
        .where(d => {
            const tags = getTags(d);
            return tags.includes("type/task/sub") && d.origin?.path === notaPath;
        });
    const resultado = [];
    for (const sub of subs) {
        const subContent = await dv.io.load(sub.file.path);
        const subLimpio = subContent.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
        const subMatch = subLimpio.match(/### Plan de Desarrollo[\s\S]*?((?:\|[^\n]+\|\n?)+)/);
        const subTareas = [];
        if (subMatch) {
            let subLines = subMatch[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
            if (subLines.length > 0) subLines.shift();
            for (const line of subLines) {
                const partes = line.split("|");
                if (partes.length < 7) continue;
                const id = partes[1].trim();
                const nombre = partes[2].trim();
                const est = partes[3].trim();
                const fechaCol = partes[4].trim();
                const secuencia = partes[5].trim();
                if (!id || isNaN(parseInt(id))) continue;
                const fechaMatch = fechaCol.match(/\[\s*fecha::\s*([\d-]+)\s*\]/);
                if (fechaMatch && fechaMatch[1] < today) continue;
                subTareas.push({ id, nombre, est, fecha: fechaMatch?.[1] || "", secuencia });
            }
        }
        const subSubtareas = await cargarSubtareas(sub.file.path);
        resultado.push({ sub, subTareas, subSubtareas });
    }
    return resultado;
};

const renderSubtareas = (subtareas, idPadre, nivel) => {
    let html = "";
    const paddingBase = 20 + (nivel * 16);
    for (const { sub, subTareas, subSubtareas } of subtareas) {
        const idSub = `${idPadre}s`;
        const subNombre = sub.detail || sub.file.name;
        html += `<tr class="subtarea-row"><td style="color:var(--text-muted);font-size:11px;padding-left:${paddingBase}px">${idSub}</td><td colspan="4" style="padding-left:${paddingBase}px">↳ <a data-href="${sub.file.path}" href="${sub.file.path}" class="internal-link">${subNombre}</a></td></tr>`;
        for (const subTarea of subTareas) {
            const idSubTarea = `${idSub}.${subTarea.id}`;
            html += `<tr class="tarea-row"><td style="color:var(--text-muted);font-size:12px;padding-left:${paddingBase + 16}px">${idSubTarea}</td><td style="padding-left:${paddingBase + 16}px">${subTarea.nombre}</td><td><div class="edit-cell"><input class="date-input" type="date" value="${subTarea.fecha}" onchange="window.saveFecha('${sub.file.path}', '${subTarea.id}', this.value, this)"><span class="saved-flash">✓</span></div></td><td style="color:var(--text-muted);font-size:12px">${subTarea.est}</td><td><div class="edit-cell"><input class="seq-input" type="number" value="${subTarea.secuencia || subTarea.id}" min="1" onchange="window.saveSecuencia('${sub.file.path}', '${subTarea.id}', this.value, this)"><span class="saved-flash">✓</span></div></td></tr>`;
            const subSubs = subSubtareas.filter(s => String(s.sub.parent_task_id) === String(subTarea.id));
            if (subSubs.length) html += renderSubtareas(subSubs, idSubTarea, nivel + 1);
        }
    }
    return html;
};

// 1. Leer tickets
const tickets = dv.pages('"20_Projects"')
    .where(p => {
        const tags = getTags(p);
        return !tags.includes("type/task/sub") &&
               !tags.includes("type/detail") &&
               p.status !== "Closed" &&
               p.status !== "Resolved" &&
               p.origin?.path === dv.current().file.path;
    });

const ticketData = [];
for (const ticket of tickets) {
    const content = await dv.io.load(ticket.file.path);
    const contentLimpio = content.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
    const tableRegex = /### Plan de Desarrollo[\s\S]*?((?:\|[^\n]+\|\n?)+)/;
    const match = contentLimpio.match(tableRegex);
    if (!match) continue;
    let lines = match[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
    if (lines.length > 0) lines.shift();
    const tareasHoy = [];
    for (const line of lines) {
        const partes = line.split("|");
        if (partes.length < 7) continue;
        const id        = partes[1].trim();
        const nombre    = partes[2].trim();
        const est       = partes[3].trim();
        const fechaCol  = partes[4].trim();
        const secuencia = partes[5].trim();
        if (!id || isNaN(parseInt(id))) continue;
        const fechaMatch = fechaCol.match(/\[\s*fecha::\s*([\d-]+)\s*\]/);
        if (fechaMatch && fechaMatch[1] < today) continue;
        tareasHoy.push({ id, nombre, est, fecha: fechaMatch?.[1] || "", secuencia });
    }
    if (!tareasHoy.length) continue;
    const subtareasConTareas = await cargarSubtareas(ticket.file.path);
    ticketData.push({ ticket, tareasHoy, subtareas: subtareasConTareas });
}

// 2. Ordenar por prioridad
ticketData.sort((a, b) => (prioOrder[a.ticket.priority] ?? 99) - (prioOrder[b.ticket.priority] ?? 99));

if (!ticketData.length) { dv.span("_No hay tareas planificadas para hoy._"); return; }

// 3. Render
let html = `<style>
    .daily-table { width:100%; border-collapse:collapse; font-size:13px; }
    .daily-table th { text-align:left; padding:4px 8px; border-bottom:1px solid var(--background-modifier-border); color:var(--text-muted); font-weight:500; }
    .daily-table td { padding:4px 8px; border-bottom:1px solid var(--background-modifier-border); color:var(--text-normal); vertical-align:middle; }
    .daily-table tr.ticket-row td { font-weight:500; background:var(--background-secondary); }
    .edit-cell { display:flex; align-items:center; gap:4px; }
    .seq-input { width:44px; background:transparent; border:1px solid var(--background-modifier-border); border-radius:4px; padding:2px 4px; color:var(--text-normal); font-size:11px; text-align:center; }
    .seq-input:focus { outline:none; border-color:var(--interactive-accent); }
    .date-input { width:110px; background:transparent; border:1px solid var(--background-modifier-border); border-radius:4px; padding:2px 4px; color:var(--text-normal); font-size:11px; }
    .date-input:focus { outline:none; border-color:var(--interactive-accent); }
    .badge-high   { color:var(--color-red);    font-size:11px; }
    .badge-medium { color:var(--color-yellow); font-size:11px; }
    .badge-low    { color:var(--color-green);  font-size:11px; }
    .saved-flash  { color:var(--color-green); font-size:10px; opacity:0; transition:opacity 0.3s; }
</style><table class="daily-table"><thead><tr><th style="width:80px">ID</th><th>Nombre</th><th style="width:120px">Fecha</th><th style="width:55px">Est.</th><th style="width:80px">Secuencia</th></tr></thead><tbody>`;

let ticketSeq = 0;
for (const { ticket, tareasHoy, subtareas } of ticketData) {
    ticketSeq++;
    const idFicticio = ticketSeq * 100;
    const prio       = ticket.priority || "Medium";
    const badgeClass = `badge-${prio.toLowerCase()}`;
    const nombre     = ticket.file.name.split(" - ").pop();

    html += `<tr class="ticket-row"><td style="color:var(--text-muted);font-size:12px">${idFicticio}</td><td><a data-href="${ticket.file.path}" href="${ticket.file.path}" class="internal-link">${nombre}</a></td><td></td><td></td><td class="${badgeClass}">${prio}</td></tr>`;

    for (const tarea of tareasHoy) {
        const idCompleto = `${idFicticio}.${tarea.id}`;
        html += `<tr class="tarea-row"><td style="color:var(--text-muted);font-size:12px;padding-left:20px">${idCompleto}</td><td style="padding-left:20px">${tarea.nombre}</td><td><div class="edit-cell"><input class="date-input" type="date" value="${tarea.fecha}" onchange="window.saveFecha('${ticket.file.path}', '${tarea.id}', this.value, this)"><span class="saved-flash">✓</span></div></td><td style="color:var(--text-muted);font-size:12px">${tarea.est}</td><td><div class="edit-cell"><input class="seq-input" type="number" value="${tarea.secuencia || tarea.id}" min="1" onchange="window.saveSecuencia('${ticket.file.path}', '${tarea.id}', this.value, this)"><span class="saved-flash">✓</span></div></td></tr>`;

        const subs = subtareas.filter(s => String(s.sub.parent_task_id) === String(tarea.id));
        if (subs.length) html += renderSubtareas(subs, idCompleto, 1);
    }
}

html += `</tbody></table>`;
const container = dv.el("div", "");
container.innerHTML = html.replace(/\n\s*/g, "");
```
---
```dataviewjs
const project = dv.current().project; // Lee el atributo 'area' del YAML de esta nota

dv.paragraph("```todoist\n" +
`name: "Mis tareas del proyecto ${project}"\n` +
`filter: "##Obsidian & @${project}"\n` +
"```");
```
`BUTTON[add-task-todoist]`
## ☕ Registro de Tiempos del Día
`button-routine-log`
```dataview
LIST
FROM "00_Inbox" AND #type/log
WHERE origin = this.file.link
SORT created DESC
LIMIT 5
```
---
## 🏗️ Tareas y Tickets
`button-new-task`
### New
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "New")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
### Assigned
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "Assigned")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
### Feedback
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "Feedback")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
### Testing
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "Testing")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
### Resolved
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "Resolved")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
### Closed
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/task')
    .where(p => p.origin && p.origin.path === dv.current().file.path && p.status === "Closed")
    .sort(p => [p.status, p.priority, p.created], "asc");

dv.table(
    ["Nombre", "Prioridad"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.priority || "-"
    ])
);
```
---
##  🏔️ Contextos
`button-new-context`
```dataview
LIST WITHOUT ID
	link(file.link, upper(context))
FROM "20_Projects" AND #type/context
WHERE origin = this.file.link
```
---
## 📝 Notas
`button-new-inbox-note`
```dataview
LIST
FROM "00_Inbox"
WHERE origin = this.file.link
```

`button-delete-note`<%*
try {
  const token = (await app.vault.adapter.read(".obsidian/todoist-token")).trim();
  const raw = tp.file.title.replace("MOC_", "").toLowerCase();
  const labelName = raw.charAt(0).toUpperCase() + raw.slice(1);

  await requestUrl({
    url: "https://api.todoist.com/api/v1/sync",
    method: "POST",
    headers: { "Authorization": `Bearer ${token}` },
    contentType: "application/x-www-form-urlencoded",
    body: `commands=${encodeURIComponent(JSON.stringify([{
      type: "label_add",
      temp_id: crypto.randomUUID(),
      uuid: crypto.randomUUID(),
      args: { name: labelName }
    }]))}`
  });

  new Notice(`✓ Etiqueta "${labelName}" creada en Todoist`);

} catch(e) {
  new Notice("Error: " + e.message.substring(0, 100));
  console.error(e);
}
%>