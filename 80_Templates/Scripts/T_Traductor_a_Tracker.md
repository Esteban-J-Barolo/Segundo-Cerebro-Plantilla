<%*
// 1. Obtener el texto del portapapeles
let text = await tp.system.clipboard();

if (!text) {
    return "Error: El portapapeles está vacío.";
}

// 2. Aplicar reglas de transformación (Regex)
let translated = text
    // Cabeceras (H1 a H4)
    .replace(/^# (.*$)/gm, "====== $1 ======")
    .replace(/^## (.*$)/gm, "===== $1 =====")
    .replace(/^### (.*$)/gm, "==== $1 ====")
    .replace(/^#### (.*$)/gm, "=== $1 ===")
    
    // Imágenes de Obsidian ![[img.png]] a {{:tracker:holasim:img.png|}}
    .replace(/!\[\[(.*?)\]\]/g, "{{:tracker:holasim:$1|}}")
    
    // Enlaces internos [[link]] a [[tracker:holasim:link]]
    // Evita duplicar si ya tiene el namespace
    .replace(/\[\[(?!tracker:)(.*?)\]\]/g, "[[tracker:holasim:$1]]")
    
    // Bloques de código ```lang ... ``` a <code lang> ... </code>
    .replace(/```(\w+)?\n([\s\S]*?)\n```/g, (match, lang, code) => {
        return `<code ${lang || ''}>\n${code}\n</code>`;
    })
    
    // Protección de expresiones n8n {{ $json... }} y //
    // Las envolvemos en %% para que DokuWiki no las toque
    .replace(/\{\{\s*(\$json.*?)\s*\}\}/g, "%%{{ $1 }}%%")
    .replace(/(\/\/)/g, "%%//%%")
    .replace(/^\|[:\s-]*\|[:\s-|]*\r?\n/gm, "")
    .replace(/`/g,"''")
    .replace(/- /g,"  * ");

// 3. Envolver el resultado final en un bloque de código DokuWiki para Obsidian
let finalResult = "```DokuWiki\n" + translated + "\n```";

// 4. Insertar el texto transformado
return finalResult;
%>