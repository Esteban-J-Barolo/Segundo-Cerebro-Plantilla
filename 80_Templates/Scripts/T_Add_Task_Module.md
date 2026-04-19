<%*
// 1. Leer editor
const view = app.workspace.activeLeaf?.view;
if (!view || view.getViewType() !== "markdown") {
    new Notice("❌ No hay un editor de Markdown activo");
    return;
}
const editor = view.editor;
const lineCount = editor.lineCount();

// 2. Buscar tabla y calcular próximo ID
let headerLine = -1;
let maxId = 0;

for (let i = 0; i < lineCount; i++) {
    const lineText = editor.getLine(i);
    if (lineText.includes("| ID") && lineText.includes("| Tarea")) {
        headerLine = i;
        continue;
    }
    if (headerLine !== -1 && lineText.trim().startsWith("|")) {
        if (lineText.includes("---")) continue;
        const cols = lineText.split("|").map(c => c.trim()).filter(c => c !== "");
        if (cols.length >= 2 && !isNaN(parseInt(cols[0]))) {
            maxId = Math.max(maxId, parseInt(cols[0]));
        }
    } else if (headerLine !== -1 && !lineText.trim().startsWith("|")) {
        break;
    }
}

if (headerLine === -1) {
    new Notice("❌ No encontré la tabla Plan de Desarrollo.");
    return;
}

const nuevoId = maxId + 1;

// 3. Inputs
const tarea     = await tp.system.prompt("Tarea / Módulo");
const est       = await tp.system.prompt("Estimación (HH:MM)", "0:00");
const fecha     = await tp.system.prompt("Fecha", "");
// const fecha     = await tp.system.prompt("Fecha", tp.date.now("YYYY-MM-DD"));
const secuencia = await tp.system.prompt("Secuencia", String(nuevoId));

if (!tarea || !est) {
    new Notice("❌ Tarea y estimación son obligatorios.");
    return;
}

// 4. Construir fila con botón eliminar
const fechaFmt  = fecha ? `[fecha:: ${fecha}]` : "";
const nuevaFila = `| ${nuevoId} | ${tarea} | ${est} | ${fechaFmt} | ${secuencia || nuevoId} | [ ] | \`BUTTON[btn-toggle]\` | \`BUTTON[btn-delete-line]\` |`;

// 5. Buscar dónde insertar
let insertLine = headerLine + 2;
while (insertLine < lineCount) {
    const lineText = editor.getLine(insertLine).trim();
    if (!lineText.startsWith("|")) break;
    insertLine++;
}

// 6. Insertar
editor.setCursor({ line: insertLine, ch: 0 });
editor.replaceRange(nuevaFila + "\n", { line: insertLine, ch: 0 });

new Notice(`✅ Tarea ${nuevoId} agregada.`);
%>