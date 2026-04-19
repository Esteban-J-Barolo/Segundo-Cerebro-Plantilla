<%*
const editor = app.workspace.activeLeaf.view.editor;
const cursor = editor.getCursor();
const lineNum = cursor.line;
const lineText = editor.getLine(lineNum);

// Alternar entre [ ] y [x]
let newLine;
if (lineText.includes("[ ]")) {
  newLine = lineText.replace("[ ]", "[x]");
} else if (lineText.includes("[x]")) {
  newLine = lineText.replace("[x]", "[ ]");
} else {
  new Notice("No se encontró checkbox en esta línea");
  return;
}

editor.replaceRange(newLine, 
  { line: lineNum, ch: 0 }, 
  { line: lineNum, ch: lineText.length }
);

const cols = lineText.split("|").map(c => c.trim()).filter(c => c !== "");
const nombre = cols[1] || "tarea";
const estado = newLine.includes("[x]") ? "completada ✅" : "reabierta 🔄";
new Notice(`"${nombre}" ${estado}`);
%>