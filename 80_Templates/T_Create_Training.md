---
<%*
// 2. BUSCAR CATEGORÍAS EXISTENTES DINÁMICAMENTE
// Escaneamos todas las notas para ver qué categorías ya usaste
const files = app.vault.getMarkdownFiles();
const existingCategories = new Set();

for (const file of files) {
    const cache = app.metadataCache.getFileCache(file);
    const fm = cache?.frontmatter;
    
    // Verificamos si la nota es una capacitación (tiene el tag type/training)
    if (fm && fm.tags && (fm.tags.includes("type/training") || fm.tags === "type/training")) {
        // Si tiene categoría definida, la guardamos
        if (fm.category) {
            existingCategories.add(fm.category);
        }
    }
}

// Convertimos a lista y ordenamos alfabéticamente
let options = Array.from(existingCategories).sort();

// Agregamos la opción de crear nueva al principio
options.unshift("✨ Nueva Categoría...");

// 3. MOSTRAR EL MENÚ
let selectedCategory = await tp.system.suggester(options, options);

// Lógica para crear una nueva si se selecciona la opción o si la lista estaba vacía
if (selectedCategory === "✨ Nueva Categoría..." || !selectedCategory) {
    selectedCategory = await tp.system.prompt("Nombre de la nueva categoría (Ej: Habilidades Blandas)");
}

// Fallback por si cancela el prompt
if (!selectedCategory) selectedCategory = "General";
%>
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
status: Active
tags:  
  - type/training
serfe_coverage_pct: 100
category: <% selectedCategory %>
project: Training - <% tp.file.title.split(" - ").pop() %>
<% tp.file.include("[[script_detectar_origen]]") %>
---
# 🎓 Capacitación: <% tp.file.title.split(" - ").pop() %>
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Active), option(Completed), option(OnHold), option(Canceled)):status]` 
> **Tiempo Total**: `$= { const horas = dv.current().horas; let mins = 0; if(horas){ const arr = Array.isArray(horas) ? horas : [horas]; arr.forEach(h => { const p = h.toString().split(":"); if(p.length===2) mins += parseInt(p[0])*60 + parseInt(p[1]); }); } const h = Math.floor(mins/60).toString().padStart(2,'0'); const m = (mins%60).toString().padStart(2,'0'); dv.span("**" + h + ":" + m + "**"); }` hs 
> **Cobertura Serfe**: `INPUT[number:serfe_coverage_pct]` %

---
### 📅 Bitácora de Sesiones
| Fecha                                     | Inicio | Fin   | Tiempo          | Tema / Módulo    |
| :---------------------------------------- | :----- | :---- | :-------------- | :--------------- |
| [fecha:: <% tp.date.now("YYYY-MM-DD") %>] | 00:00  | 00:00 | [horas:: 00:00] | Inicio del curso |
`BUTTON[btn-agregar-sesion]` `BUTTON[btn-recalcular-sesion]`

---
## 📝 Clases
`button-add-detail`
```dataview
LIST WITHOUT ID
	link(file.link, upper(detail))
FROM "20_Projects" AND #type/detail 
WHERE origin = this.file.link
```
# Notas
`button-new-inbox-note`
```dataview
LIST
FROM "00_Inbox"
WHERE origin = this.file.link
```

`button-delete-note`