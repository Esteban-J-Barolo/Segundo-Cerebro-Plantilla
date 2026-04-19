<%*
// 1. Obtener la vista activa de Markdown
const view = app.workspace.activeLeaf?.view;
if (!view || view.getViewType() !== "markdown") {
    new Notice("❌ No hay un editor de Markdown activo");
    return;
}

const editor = view.editor;

// GUARDAR la posición del cursor ANTES de hacer cambios
const cursorInicial = editor.getCursor();

const data = editor.getValue().split("\n"); 
let tableUpdated = false;

// Helpers
const toMins = (t) => {
    if (!t) return 0;
    const [h, m] = t.split(':').map(Number);
    return h * 60 + m;
};

const toTimeStr = (mins) => {
    let sign = mins < 0 ? "-" : "";
    mins = Math.abs(mins);
    const h = Math.floor(mins / 60).toString().padStart(2, '0');
    const m = (mins % 60).toString().padStart(2, '0');
    return `${sign}${h}:${m}`;
};

// 2. Procesamos línea por línea
const newData = data.map((line) => {
    const cols = line.split("|");
    
    if (cols.length >= 6) {
        const colInicio = cols[2]?.trim();
        const colFin = cols[3]?.trim();
        
        const timeRegex = /^\d{2}:\d{2}$/;
        if (timeRegex.test(colInicio) && timeRegex.test(colFin)) {
            const mins = toMins(colFin) - toMins(colInicio);
            const nuevaDuracion = toTimeStr(mins);
            
            const duracionActual = cols[4];
            if (!duracionActual.includes(nuevaDuracion)) {
                cols[4] = ` [horas:: ${nuevaDuracion}] `;
                tableUpdated = true;
                return cols.join("|");
            }
        }
    }
    return line; 
});

// 3. Aplicar cambios
if (tableUpdated) {
    editor.setValue(newData.join("\n"));
    new Notice("✅ Sesiones recalculadas.");
} else {
    new Notice("👍 Todo estaba actualizado.");
}

%>