<%*
const view = app.workspace.activeLeaf?.view;
if (!view || view.getViewType() !== "markdown") { new Notice("❌ No hay editor activo"); return; }
const editor = view.editor;
const lineCount = editor.lineCount();

const fecha  = await tp.system.prompt("Fecha", tp.date.now("YYYY-MM-DD"));
const inicio = await tp.system.prompt("Hora Inicio (HH:MM)", tp.date.now("HH:mm"));
const fin    = await tp.system.prompt("Hora Fin (HH:MM)", tp.date.now("HH:mm"));
const tipo   = await tp.system.suggester(
    ["Rutina", "Reunión", "Consulta", "Bug urgente", "Admin", "Otro"],
    ["rutina", "reunión", "consulta", "bug urgente", "admin", "otro"]
);
const desc = await tp.system.prompt("¿Qué se hizo?");

if (!fecha || !inicio || !fin) return;

const toMins = (t) => { const [h, m] = t.split(":").map(Number); return h * 60 + m; };
const diff = toMins(fin) - toMins(inicio);
if (diff < 0) { new Notice("❌ Fin antes que Inicio"); return; }

const h = Math.floor(diff / 60).toString().padStart(2, "0");
const m = (diff % 60).toString().padStart(2, "0");
const duracion = `${h}:${m}`;

const nuevaFila = `| [fecha:: ${fecha}] | ${inicio} | ${fin} | [duracion:: ${duracion}] | ${tipo || "rutina"} | ${desc || "-"} |`;

// Buscar tabla por header
let headerLine = -1;
for (let i = 0; i < lineCount; i++) {
    const t = editor.getLine(i);
    if (t.includes("| Tipo") && t.includes("| Tarea Realizada")) { headerLine = i; break; }
}

if (headerLine === -1) { new Notice("❌ No encontré la tabla"); return; }

let insertLine = headerLine + 2;
while (insertLine < lineCount) {
    if (!editor.getLine(insertLine).trim().startsWith("|")) break;
    insertLine++;
}

editor.setCursor({ line: insertLine, ch: 0 });
editor.replaceRange(nuevaFila + "\n", { line: insertLine, ch: 0 });
new Notice(`✅ Registrado: ${duracion} hs — ${tipo}`);
%>