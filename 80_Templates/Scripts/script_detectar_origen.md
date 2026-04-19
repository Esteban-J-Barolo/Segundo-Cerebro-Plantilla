<%*
let origenNombre = window._origenNota || "Desconocido";
window._origenNota = null;

if (origenNombre !== "Desconocido") {
    tR += `origin: "[[${origenNombre}]]"`;
} else {
    const history = app.workspace.getLastOpenFiles();
    const candidatos = history
        .slice(1, 7)
        .map(path => app.vault.getAbstractFileByPath(path))
        .filter(f => f !== null)
        .slice(0, 5);

    if (candidatos.length > 0) {
        const seleccionada = await tp.system.suggester(
            candidatos.map(f => f.basename),
            candidatos,
            true,
            "Selecciona la nota de origen:"
        );
        if (seleccionada) origenNombre = seleccionada.basename;
    }

    tR += origenNombre !== "Desconocido" 
        ? `origin: "[[${origenNombre}]]"` 
        : `origin: ${origenNombre}`;
}
%>