<%*
// 0. Leer tabla Plan de Desarrollo para ofrecer opciones
const view = app.workspace.activeLeaf?.view;
const editor = view.editor;
const lineCount = editor.lineCount();

let planHeaderLine = -1;
const tareasPlan = [];

for (let i = 0; i < lineCount; i++) {
    const lineText = editor.getLine(i);
    if (lineText.includes("| ID") && lineText.includes("| Tarea")) {
        planHeaderLine = i;
        continue;
    }
    if (planHeaderLine !== -1 && lineText.trim().startsWith("|")) {
        if (lineText.includes("---")) continue;
        const partes = lineText.split("|");
        if (partes.length < 3) continue;
        const id = partes[1].trim();
        const nombre = partes[2].trim();
        if (id && !isNaN(parseInt(id))) {
            tareasPlan.push({ id, nombre });
        }
    } else if (planHeaderLine !== -1 && !lineText.trim().startsWith("|")) {
        break;
    }
}

// 1. Inputs
const fecha = await tp.system.prompt("Fecha", tp.date.now("YYYY-MM-DD"));
const inicio = await tp.system.prompt("Hora Inicio (HH:MM)", tp.date.now("HH:mm"));
const fin = await tp.system.prompt("Hora Fin (HH:MM)", tp.date.now("HH:mm"));
const nota = await tp.system.prompt("¿Qué se hizo?");

// Elegir tarea del plan
let idTarea = "-";
if (tareasPlan.length) {
    const opciones = tareasPlan.map(t => `${t.id} — ${t.nombre}`);
    const seleccion = await tp.system.suggester(opciones, opciones);
    if (seleccion) idTarea = seleccion.split(" — ")[0];
}

if (fecha && inicio && fin) {
    const toMins = (t) => {
        const [h, m] = t.split(':').map(Number);
        return h * 60 + m;
    };
    const diffMins = toMins(fin) - toMins(inicio);

    if (diffMins < 0) {
        new Notice("❌ Error: Fin antes que Inicio.");
    } else {
        const h = Math.floor(diffMins / 60).toString().padStart(2, '0');
        const m = (diffMins % 60).toString().padStart(2, '0');
        const duracion = `${h}:${m}`;
        const nuevaFila = `| [fecha:: ${fecha}] | ${inicio} | ${fin} | [duracion:: ${duracion}] | ${idTarea} | ${nota || "-"} |`;

        if (!view || view.getViewType() !== "markdown") {
            new Notice("❌ No hay un editor de Markdown activo");
            return;
        }

        let headerLine = -1;
        for (let i = 0; i < lineCount; i++) {
            const lineText = editor.getLine(i);
            if (lineText.includes("| Inicio") && lineText.includes("| Fin")) {
                headerLine = i;
                break;
            }
        }

        if (headerLine !== -1) {
            let insertLine = headerLine + 2;
            while (insertLine < lineCount) {
                const lineText = editor.getLine(insertLine).trim();
                if (!lineText.startsWith("|")) break;
                insertLine++;
            }
            editor.setCursor({ line: insertLine, ch: 0 });
            editor.replaceRange(nuevaFila + "\n", { line: insertLine, ch: 0 });
            new Notice(`✅ Registrado: ${duracion} hs — Tarea ${idTarea}`);
        } else {
            new Notice("❌ No encontré la tabla de bitácora.");
        }
    }
}
%>