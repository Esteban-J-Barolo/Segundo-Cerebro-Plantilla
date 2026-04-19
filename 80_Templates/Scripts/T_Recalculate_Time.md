<%*
// 1. Leer el archivo actual
const file = tp.config.active_file;
let content = await app.vault.read(file);
let lines = content.split("\n");
let tableUpdated = false;

// GUARDAR posición del cursor inicial
const view = app.workspace.activeLeaf?.view;
if (!view || view.getViewType() !== "markdown") {
    new Notice("❌ No hay un editor de Markdown activo");
    return;
}
const editor = view.editor;
const cursorInicial = editor.getCursor();

// 2. Función auxiliar para minutos
const toMins = (t) => {
    if (!t) return 0;
    const [h, m] = t.split(':').map(Number);
    return h * 60 + m;
};

// 3. Función auxiliar para formatear HH:MM
const toTimeStr = (mins) => {
    let sign = mins < 0 ? "-" : "";
    mins = Math.abs(mins);
    const h = Math.floor(mins / 60).toString().padStart(2, '0');
    const m = (mins % 60).toString().padStart(2, '0');
    return `${sign}${h}:${m}`;
};

// 4. Recorrer líneas buscando la tabla de tiempos
// Buscamos líneas que tengan estructura de tabla y contengan fechas/horas
// Asumimos estructura: | Fecha | Inicio | Fin | Duración | Nota |
for (let i = 0; i < lines.length; i++) {
    const line = lines[i].trim();
    
    // Solo procesamos líneas que empiezan con | y no son cabeceras (---)
    if (line.startsWith("|") && !line.includes("---") && !line.includes("**Inicio**")) {
        const cols = line.split("|");
        
        // Verificamos que tenga suficientes columnas (al menos 6 elementos porque split crea vacíos al inicio/fin)
        if (cols.length >= 7) {
            const colInicio = cols[2].trim(); // Columna 2: Inicio
            const colFin = cols[3].trim();    // Columna 3: Fin
            
            // Validamos que parezcan horas (HH:MM)
            if (colInicio.match(/^\d{2}:\d{2}$/) && colFin.match(/^\d{2}:\d{2}$/)) {
                
                // Calculamos nueva duración
                const mins = toMins(colFin) - toMins(colInicio);
                const nuevaDuracion = toTimeStr(mins);
                
                // Reconstruimos la columna de duración manteniendo el formato Dataview
                // cols[4] es la columna de duración
                const duracionActual = cols[4];
                
                // Solo actualizamos si el valor es diferente para no "ensuciar" el historial
                if (!duracionActual.includes(nuevaDuracion)) {
                    cols[4] = ` [duracion:: ${nuevaDuracion}] `;
                    lines[i] = cols.join("|");
                    tableUpdated = true;
                }
            }
        }
    }
}

// 5. Guardar cambios si hubo actualizaciones
if (tableUpdated) {
    await app.vault.modify(file, lines.join("\n"));
    new Notice("✅ Tabla recalculada correctamente.");
} else {
    new Notice("👍 La tabla ya estaba actualizada.");
}

%>