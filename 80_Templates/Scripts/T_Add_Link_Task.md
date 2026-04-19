<%*
// 1. APIs
const dv = app.plugins.plugins.dataview.api;
const currFile = tp.config.target_file;

// 2. Obtener datos de la nota actual
const currentData = dv.page(currFile.path);
const currentOrigin = currentData.origin;

if (!currentOrigin || !currentOrigin.path) {
    new Notice("⚠️ Esta nota no tiene un 'origin' válido.");
    return;
}

// 3. Buscar tareas candidatas
const tareasCandidatas = dv.pages("#type/task")
    .where(p => 
        p.origin && 
        p.origin.path === currentOrigin.path && 
        p.file.path !== currFile.path
    );

if (tareasCandidatas.length === 0) {
    new Notice("No se encontraron otras tareas con el mismo origen.");
    return;
}

// 4. Preparar el menú (Suggester)
const nombresTareas = tareasCandidatas.map(p => p.file.name).values;
const linksTareas = tareasCandidatas.map(p => p.file.link).values;

const seleccion = await tp.system.suggester(nombresTareas, linksTareas);

if (seleccion) {
    // 5. Procesar el nombre para que quede limpio [[nombre]]
    // seleccion.path es "Carpeta/Subcarpeta/Nombre.md"
    // Con esto obtenemos solo "Nombre"
    const nombreLimpio = seleccion.path.split('/').pop().replace('.md', '');
    const linkString = `[[${nombreLimpio}]]`;

    // 6. Actualizar el YAML
    await app.fileManager.processFrontMatter(app.vault.getAbstractFileByPath(currFile.path), (fm) => {
        if (!fm.related_tasks) fm.related_tasks = [];
        
        // Evitamos duplicados comparando el nombre limpio
        const existe = fm.related_tasks.some(t => t.includes(nombreLimpio));
        
        if (!existe) {
            fm.related_tasks.push(linkString);
            new Notice(`✅ Vinculado: ${nombreLimpio}`);
        } else {
            new Notice("Esta tarea ya está en la lista.");
        }
    });
}
%>