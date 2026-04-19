---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags:  
  - type/log
<% tp.file.include("[[script_detectar_origen]]") %>
---
# ☕ Registro de Tiempos del Día
## ⏱ Bitácora del Día
| Fecha                | Inicio | Fin   | Duración           | Tipo   | Tarea Realizada |
| :------------------- | :----- | :---- | :----------------- | :----- | :-------------- |
`BUTTON[btn-add-interruption]` `BUTTON[btn-recalcular-tiempos]`

## 📊 Gantt del Día
```dataviewjs
const logContent = await dv.io.load(dv.current().file.path);
const createdMatch = dv.current().created?.toString().split(" ")[0] || dv.date("today").toFormat("yyyy-MM-dd");
const today = createdMatch;

const toMins = (s) => { const [h, m] = (s||"").replace("hs","").trim().split(":").map(Number); return (h||0)*60+(m||0); };
const toStr  = (m) => `${Math.floor(m/60).toString().padStart(2,"0")}:${(m%60).toString().padStart(2,"0")}`;

const getTags = (p) => {
    if (!p.tags) return [];
    if (Array.isArray(p.tags)) return p.tags;
    if (p.tags.values) return p.tags.values;
    return [];
};

// 1. Leer bitácora del log
const logLimpio = logContent.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
const bitacoraRegex = /## ⏱ Bitácora del Día[\s\S]*?((?:\|[^\n]+\|\n?)+)/;
const bitacoraMatch = logLimpio.match(bitacoraRegex);

const entradasBitacora = [];
const interrupciones = [];
const rutina = [];

if (bitacoraMatch) {
    let lines = bitacoraMatch[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
    if (lines.length > 0) lines.shift();
    for (const line of lines) {
        const partes = line.split("|");
        if (partes.length < 7) continue;
        const inicio = partes[2].trim();
        const fin    = partes[3].trim();
        const tipo   = partes[5].trim().toLowerCase();
        const desc   = partes[6].trim();
        if (!inicio.match(/^\d{2}:\d{2}$/) || !fin.match(/^\d{2}:\d{2}$/)) continue;
        const entrada = { inicio, fin, tipo, desc, inicioM: toMins(inicio), finM: toMins(fin) };
        entradasBitacora.push(entrada);
        if (tipo === "rutina") rutina.push(entrada);
        else interrupciones.push(entrada);
    }
}
entradasBitacora.sort((a, b) => a.inicioM - b.inicioM);

// 2. Calcular huecos libres para el planificado
const horaInicio = entradasBitacora.length ? entradasBitacora[0].inicioM : 9 * 60;
const horaFin = 20 * 60;
const ocupados = entradasBitacora.map(e => ({ ini: e.inicioM, fin: e.finM }));
const huecos = [];
let cursor = horaInicio;
for (const bloque of ocupados) {
    if (bloque.ini > cursor) huecos.push({ ini: cursor, fin: bloque.ini });
    cursor = Math.max(cursor, bloque.fin);
}
if (cursor < horaFin) huecos.push({ ini: cursor, fin: horaFin });

// 3. Leer tareas planificadas
const tickets = dv.pages('"20_Projects"')
    .where(p => {
        const tags = getTags(p);
        return !tags.includes("type/task/sub") &&
               !tags.includes("type/detail") &&
               p.status !== "Closed" && p.status !== "Resolved";
    });

const planificadas = [];

for (const ticket of tickets) {
    const content = await dv.io.load(ticket.file.path);
    const limpio = content.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
    const match = limpio.match(/### Plan de Desarrollo[\s\S]*?((?:\|[^\n]+\|\n?)+)/);
    if (!match) continue;
    let lines = match[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
    if (lines.length > 0) lines.shift();
    for (const line of lines) {
        const partes = line.split("|");
        if (partes.length < 7) continue;
        const id       = partes[1].trim();
        const nombre   = partes[2].trim();
        const est      = partes[3].trim();
        const fechaCol = partes[4].trim();
        const check    = partes[6]?.trim();
        if (!id || isNaN(parseInt(id))) continue;
        const fechaMatch = fechaCol.match(/\[\s*fecha::\s*([\d-]+)\s*\]/);
        if (!fechaMatch || fechaMatch[1] !== today) continue;
        planificadas.push({
            nombre: `${ticket.file.name.split(" - ").pop()} · ${nombre}`,
            durM: toMins(est) || 30,
            completada: check === "[x]",
            secuencia: parseInt(partes[5]?.trim()) || 999
        });
    }
}

// Subtareas planificadas
const subtareas = dv.pages('"20_Projects"')
    .where(p => {
        const tags = getTags(p);
        return tags.includes("type/task/sub") &&
               p.status !== "Closed" && p.status !== "Resolved";
    });

for (const sub of subtareas) {
    const content = await dv.io.load(sub.file.path);
    const limpio = content.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
    const match = limpio.match(/### Plan de Desarrollo[\s\S]*?((?:\|[^\n]+\|\n?)+)/);
    if (!match) continue;
    let lines = match[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
    if (lines.length > 0) lines.shift();
    for (const line of lines) {
        const partes = line.split("|");
        if (partes.length < 7) continue;
        const id       = partes[1].trim();
        const nombre   = partes[2].trim();
        const est      = partes[3].trim();
        const fechaCol = partes[4].trim();
        const check    = partes[6]?.trim();
        if (!id || isNaN(parseInt(id))) continue;
        const fechaMatch = fechaCol.match(/\[\s*fecha::\s*([\d-]+)\s*\]/);
        if (!fechaMatch || fechaMatch[1] !== today) continue;
        const subNombre = sub.detail || sub.file.name.split(" - ").pop();
        planificadas.push({
            nombre: `${subNombre} · ${nombre}`,
            durM: toMins(est) || 30,
            completada: check === "[x]",
            secuencia: parseInt(partes[5]?.trim()) || 999
        });
    }
}

planificadas.sort((a, b) => a.secuencia - b.secuencia);

// 4. Distribuir planificadas en huecos
const bloquesPlan = [];
let huecoIdx = 0;
let posEnHueco = huecos.length ? huecos[0].ini : horaInicio;

for (const tarea of planificadas) {
    let restante = tarea.durM;
    let parteIdx = 1;
    while (restante > 0 && huecoIdx < huecos.length) {
        const hueco = huecos[huecoIdx];
        if (posEnHueco >= hueco.fin) { huecoIdx++; if (huecoIdx < huecos.length) posEnHueco = huecos[huecoIdx].ini; continue; }
        const disponible = hueco.fin - posEnHueco;
        const usar = Math.min(restante, disponible);
        const nombre = planificadas.length > 1 || tarea.durM > disponible
            ? `${tarea.nombre} (${parteIdx})`.replace(/:/g," ").substring(0,40)
            : tarea.nombre.replace(/:/g," ").substring(0,40);
        bloquesPlan.push({
            nombre,
            ini: posEnHueco,
            fin: posEnHueco + usar,
            completada: tarea.completada
        });
        posEnHueco += usar;
        restante -= usar;
        parteIdx++;
        if (posEnHueco >= hueco.fin) { huecoIdx++; if (huecoIdx < huecos.length) posEnHueco = huecos[huecoIdx].ini; }
    }
}

// 5. Leer bitácoras reales de tickets y subtareas
const bloquesReales = [];

const todasLasNotas = [...tickets, ...subtareas];
for (const nota of todasLasNotas) {
    const content = await dv.io.load(nota.file.path);
    const limpio = content.replace(/```[\s\S]*?```/g, "").replace(/`[^`]*`/g, "");
    const bitMatch = limpio.match(/## 3\. Bitácora de Ejecución[\s\S]*?((?:\|[^\n]+\|\n?)+)/);
    if (!bitMatch) continue;
    let lines = bitMatch[1].split("\n").map(l => l.trim()).filter(l => l.includes("|") && !l.includes("---"));
    if (lines.length > 0) lines.shift();
    for (const line of lines) {
        const partes = line.split("|");
        if (partes.length < 7) continue;
        const fechaCol = partes[1].trim();
        const inicio   = partes[2].trim();
        const fin      = partes[3].trim();
        const desc     = partes[6].trim();
        const fechaMatch = fechaCol.match(/\[\s*fecha::\s*([\d-]+)\s*\]/);
        if (!fechaMatch || fechaMatch[1] !== today) continue;
        if (!inicio.match(/^\d{2}:\d{2}$/) || !fin.match(/^\d{2}:\d{2}$/)) continue;
        const nombre = `${nota.file.name.split(" - ").pop()} · ${desc}`.replace(/:/g," ").substring(0, 40);
        bloquesReales.push({ nombre, inicio, fin, inicioM: toMins(inicio), finM: toMins(fin) });
    }
}
bloquesReales.sort((a, b) => a.inicioM - b.inicioM);

if (!bloquesPlan.length && !bloquesReales.length && !interrupciones.length && !rutina.length) {
    dv.span("_Sin datos para mostrar en el Gantt._");
    return;
}

// 6. Construir mermaid
let code = "gantt\n";
code += "  dateFormat HH:mm\n";
code += "  axisFormat %H:%M\n";

if (bloquesPlan.length) {
    code += "  section Planificado\n";
    for (const b of bloquesPlan) {
        const estado = b.completada ? "done, " : "";
        code += `    ${b.nombre} : ${estado}${toStr(b.ini)}, ${toStr(b.fin)}\n`;
    }
}

if (bloquesReales.length) {
    code += "  section Real\n";
    for (const b of bloquesReales) {
        code += `    ${b.nombre} : crit, ${b.inicio}, ${b.fin}\n`;
    }
}

if (rutina.length) {
    code += "  section Rutina\n";
    for (const r of rutina) {
        code += `    ${r.desc.replace(/:/g," ").substring(0,40)} : ${r.inicio}, ${r.fin}\n`;
    }
}

if (interrupciones.length) {
    code += "  section Interrupciones\n";
    for (const i of interrupciones) {
        code += `    ${`${i.tipo} - ${i.desc}`.replace(/:/g," ").substring(0,40)} : ${i.inicio}, ${i.fin}\n`;
    }
}

// 7. Resumen de tiempos
const minsRutina = rutina.reduce((acc, r) => acc + (r.finM - r.inicioM), 0);
const minsInterrupciones = interrupciones.reduce((acc, i) => acc + (i.finM - i.inicioM), 0);
const minsPlanificado = bloquesPlan.reduce((acc, b) => acc + (b.fin - b.ini), 0);
const minsReal = bloquesReales.reduce((acc, b) => acc + (b.finM - b.inicioM), 0);

dv.paragraph("```mermaid\n" + code + "\n```");
dv.paragraph(
    `⏱ **Planificado:** ${toStr(minsPlanificado)} hs` +
    `　　✅ **Real:** ${toStr(minsReal)} hs` +
    (minsRutina ? `　　🔄 **Rutina:** ${toStr(minsRutina)} hs` : "") +
    (minsInterrupciones ? `　　⚡ **Interrupciones:** ${toStr(minsInterrupciones)} hs` : "")
);
```
`button-delete-note`