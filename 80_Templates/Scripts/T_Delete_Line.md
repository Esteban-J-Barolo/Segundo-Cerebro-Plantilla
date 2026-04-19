<%*
const editor = app.workspace.activeLeaf.view.editor;
const cursor = editor.getCursor();
const lineNum = cursor.line;
const lineText = editor.getLine(lineNum);

// Extraer nombre de la tarea para mostrar en confirmación
const cols = lineText.split("|").map(c => c.trim()).filter(c => c !== "");
const nombre = cols[1] || "esta fila";

const confirm = await tp.system.suggester(["Sí, borrar", "No"], [true, false]);

if (!confirm) return;

editor.replaceRange("",
    { line: lineNum, ch: 0 },
    { line: lineNum + 1, ch: 0 }
);

new Notice(`✅ "${nombre}" eliminada.`);
%>