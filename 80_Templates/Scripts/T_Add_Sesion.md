<%*
// 1. Inputs
const fecha = await tp.system.prompt("Fecha", tp.date.now("YYYY-MM-DD"));
const inicio = await tp.system.prompt("Hora Inicio (HH:MM)", tp.date.now("HH:mm"));
const fin = await tp.system.prompt("Hora Fin (HH:MM)", tp.date.now("HH:mm"));
const tema = await tp.system.prompt("Tema o Módulo estudiado");

if (fecha && inicio && fin) {
    // 2. Cálculos
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
        
        // Fila a insertar
        const nuevaFila = `| [fecha:: ${fecha}] | ${inicio} | ${fin} | [horas:: ${duracion}] | ${tema || "-"} |`;
        
        // 3. Manipulación del Editor
        const view = app.workspace.activeLeaf?.view;
        if (!view || view.getViewType() !== "markdown") {
            new Notice("❌ No hay un editor de Markdown activo");
            return;
        }
        
        const editor = view.editor;
        
        // GUARDAR posición del cursor
        const cursorInicial = editor.getCursor();
        
        const lineCount = editor.lineCount();
        let headerLine = -1;
        
        // Buscar la tabla
        for (let i = 0; i < lineCount; i++) {
            const lineText = editor.getLine(i);
            if (lineText.includes("| Inicio") && lineText.includes("| Fin")) {
                headerLine = i;
                break;
            }
        }
        
        if (headerLine !== -1) {
            // Buscar dónde insertar (saltando el separador |---|)
            let insertLine = headerLine + 2;
            while (insertLine < lineCount) {
                const lineText = editor.getLine(insertLine).trim();
                // Si la línea no empieza con |, hemos llegado al fin de la tabla
                if (!lineText.startsWith("|")) break;
                insertLine++;
            }
            
            editor.replaceRange(nuevaFila + "\n", {line: insertLine, ch: 0});
            new Notice(`✅ Registrada: ${duracion} hs`);
            
        } else {
            new Notice("❌ No encontré la tabla de sesiones.");
        }
    }
}
%>