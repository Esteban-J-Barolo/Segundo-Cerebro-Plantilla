---
created: 2026-01-25 18:58
tags:
  - type/area
area: serfe
ultimo_total_reportado: 57:59
ultimo_serfe_reportado: 57:59
reporte_fin: 2026-02-28
reporte_inicio: 2025-12-18
reporte_fin_registro: 2026-04-18
reporte_inicio_registro: 2026-04-11
origin: "[[MOC_Trabajo]]"
---
# Serfe
```dataviewjs
const area = dv.current().area; // Lee el atributo 'area' del YAML de esta nota

dv.paragraph("```todoist\n" +
`name: "Mis tareas del área ${area}"\n` +
`filter: "##Obsidian & /${area} & @${area}"\n` +
"```");
```
`BUTTON[add-task-todoist]`

---
## Horas para cargar
> [!ABSTRACT] Panel de Control
> **Rango de Fechas:** `INPUT[date:reporte_inicio_registro]` al `INPUT[date:reporte_fin_registro]`

```dataviewjs
// =============================================================================
// SCRIPT OPTIMIZADO DE REPORTE DE TIEMPO - DATAVIEW (OBSIDIAN)
// =============================================================================

// --- CONFIGURACIÓN DE FECHAS ---
let rawInicio = dv.current().reporte_inicio_registro;
let rawFin = dv.current().reporte_fin_registro;

function parseDate(input) {
    if (!input) return moment();
    if (input.ts) return moment(input.toJSDate());
    return moment(input);
}

const fInicio = parseDate(rawInicio);
const fFin = parseDate(rawFin);

// --- FUNCIONES DE TIEMPO ---
function timeToMins(timeStr) {
    if (!timeStr) return 0;
    const parts = timeStr.toString().split(":");
    return parts.length < 2 ? 0 : (parseInt(parts[0]) * 60) + parseInt(parts[1]);
}

function minsToTime(totalMins) {
    const sign = totalMins < 0 ? "-" : "";
    totalMins = Math.abs(Math.round(totalMins));
    const h = Math.floor(totalMins / 60).toString().padStart(2, '0');
    const m = (totalMins % 60).toString().padStart(2, '0');
    return `${sign}${h}:${m}`;
}

// --- NORMALIZACIÓN DE ORIGIN (OPTIMIZADA) ---
function normalizarOrigin(origin) {
    if (!origin) return "";
    const str = origin.path || origin.toString();
    return str.replace(/\[\[|\]\]|\.md/g, '').split('/').pop();
}

// --- EXTRACCIÓN DE INLINE FIELDS (CON CACHE DE REGEX) ---
const regexCache = {};
function getRegex(campo) {
    if (!regexCache[campo]) {
        regexCache[campo] = new RegExp(`\\[${campo}::\\s*([^\\]]+)\\]`, 'i');
    }
    return regexCache[campo];
}

function extraerInlineField(texto, campo) {
    const match = texto.match(getRegex(campo));
    return match?.[1]?.trim() || null;
}

// --- EXTRACCIÓN DE DATOS DE TABLAS MARKDOWN (OPTIMIZADA) ---
function extraerDatosTabla(contenido, campoTiempo = 'duracion') {
    if (!contenido) return [];
    
    const lineas = contenido.split('\n');
    const datos = [];
    let enTabla = false;
    let headerEncontrado = false;
    
    for (const linea of lineas) {
        // Detectar inicio de tabla
        if (!headerEncontrado && linea.includes('| Fecha') && 
            (linea.includes('| Duración') || linea.includes('| Tiempo'))) {
            enTabla = true;
            headerEncontrado = true;
            continue;
        }
        
        // Saltar separador
        if (enTabla && linea.includes(':---')) continue;
        
        // Procesar filas de datos
        if (enTabla && linea.trim().startsWith('|') && !linea.includes('BUTTON')) {
            const fechaValue = extraerInlineField(linea, 'fecha');
            const tiempoValue = extraerInlineField(linea, campoTiempo) || 
                               extraerInlineField(linea, 'horas');
            
            if (fechaValue && tiempoValue) {
                const columnas = linea.split('|');
                datos.push({ 
                    fecha: fechaValue, 
                    duracion: tiempoValue,
                    descripcion: columnas[columnas.length - 2]?.trim() || "Sin descripción"
                });
            }
        }
        
        // Salir de la tabla
        if (enTabla && (!linea.trim() || linea.includes('BUTTON') || linea.startsWith('#'))) {
            break;
        }
    }
    
    return datos;
}

// --- CACHE DE CONTENIDOS DE ARCHIVOS ---
const cacheContenidos = new Map();

async function leerContenido(path) {
    if (!cacheContenidos.has(path)) {
        try {
            cacheContenidos.set(path, await dv.io.load(path));
        } catch (e) {
            cacheContenidos.set(path, null);
        }
    }
    return cacheContenidos.get(path);
}

// --- RECOLECCIÓN DE DATOS ---
const currentNoteName = dv.current().file.name;
const todasLasNotas = dv.pages('"20_Projects" or "00_Inbox"');
const carpetasProyecto = [];
const pages = [];

// Función recursiva para recolectar subtareas de cualquier nota
const recolectarSubtareas = (notaNombre) => {
    todasLasNotas.forEach(p => {
        const originNormalizado = normalizarOrigin(p.origin);
        const tags = p.tags?.toString().toLowerCase() || "";
        
        if (tags.includes("type/task/sub") && originNormalizado === notaNombre) {
            pages.push(p);
            // Recursivo: buscar subtareas de esta subtarea
            recolectarSubtareas(p.file.name);
        }
    });
};

// Bucle principal: proyectos, training y log directos
todasLasNotas.forEach(p => {
    const originNormalizado = normalizarOrigin(p.origin);
    const tags = p.tags?.toString().toLowerCase() || "";

    if (p.tags.includes("type/project") && originNormalizado === currentNoteName) {
        carpetasProyecto.push(p.file.name);
    }

    if (tags.includes("type/training") && originNormalizado === currentNoteName) {
        pages.push(p);
    }
});

// Segundo bucle: tareas, logs y subtareas de los proyectos hijos
if (carpetasProyecto.length > 0) {
    todasLasNotas.forEach(p => {
        const originNormalizado = normalizarOrigin(p.origin);
        const tags = p.tags?.toString().toLowerCase() || "";

        if (tags.includes("type/task") && !tags.includes("type/task/sub") &&
            carpetasProyecto.includes(originNormalizado)) {
            pages.push(p);
            // Recolectar subtareas de esta tarea recursivamente
            recolectarSubtareas(p.file.name);
        }

        if (tags.includes("type/log") && carpetasProyecto.includes(originNormalizado)) {
            pages.push(p);
        }
    });
}

// --- PROCESAMIENTO ASÍNCRONO EN PARALELO ---
const promesas = pages.map(async (page) => {
    const tags = page.file.tags?.toString().toLowerCase() || "";
    
    // Determinar tipo y campo de tiempo
    let tipo = "otro";
    let campoTiempo = "duracion";
    
    if (tags.includes("type/task")) {
        tipo = "tarea";
        campoTiempo = "duracion";
    } else if (tags.includes("type/training")) {
        tipo = "capacitacion";
        campoTiempo = "horas";
    } else if (tags.includes("type/log")) {
        tipo = "rutina";
        campoTiempo = "duracion";
    }
    
    // Leer y procesar contenido
    const contenido = await leerContenido(page.file.path);
    const datosTabla = contenido ? extraerDatosTabla(contenido, campoTiempo) : [];
    
    return { page, datosTabla, tipo };
});

// Esperar a que todas las lecturas terminen
const resultados = await Promise.all(promesas);

// --- CONSTRUCCIÓN DEL DIARIO ---
const diario = {};
let totalPeriodoMins = 0;

for (const { page, datosTabla, tipo } of resultados) {
    for (const fila of datosTabla) {
        const fechaEntrada = parseDate(fila.fecha);
        
        // Verificar si está en el rango de fechas
        if (fechaEntrada.isValid() && 
            fechaEntrada.isSameOrAfter(fInicio, 'day') && 
            fechaEntrada.isSameOrBefore(fFin, 'day')) {
            
            const dateKey = fechaEntrada.format("YYYY-MM-DD");
            const mins = timeToMins(fila.duracion);

            if (mins > 0) {
                // Inicializar día si no existe
                if (!diario[dateKey]) {
                    diario[dateKey] = { entradas: [], totalMins: 0 };
                }
                
                // Preparar nombre y link
                const nombreDisplay = page.file.name.split(" - ").pop();
                const safeLink = `[[${page.file.path}\\|${nombreDisplay}]]`;
                const nombreTarea = fila.descripcion;
                
                // Agregar entrada
                diario[dateKey].entradas.push({
                    link: safeLink, 
                    nombre: nombreTarea,
                    tiempo: minsToTime(mins),
                    mins: mins,
                    tipo: tipo
                });
                
                diario[dateKey].totalMins += mins;
                totalPeriodoMins += mins;
            }
        }
    }
}

// --- RENDERIZADO ---
dv.header(3, `📅 Registro Diario (${fInicio.format("DD/MM")} - ${fFin.format("DD/MM")})`);
dv.paragraph(`**Tiempo Total Registrado:** ${minsToTime(totalPeriodoMins)} hs`);
dv.paragraph("---");

function crearTablaMD(items, prefix) {
    if (items.length === 0) return `${prefix} _Sin actividad registrada_`;
    
    let md = `${prefix} | Actividad | Tiempo | Tarea |\n${prefix} | :--- | :---: | :---: |`;
    
    for (const item of items) {
        md += `\n${prefix} | <code style="user-select:all; display:block; color:var(--text-accent);">${item.nombre}</code> | ${item.tiempo} | ${item.link} |`;
    }
    
    return md;
}

// Ordenar días de más reciente a más antiguo
const diasOrdenados = Object.keys(diario).sort((a, b) => new Date(b) - new Date(a));

if (diasOrdenados.length === 0) {
    dv.paragraph("⚠️ No hay registros de tiempo en este rango de fechas.");
} else {
    for (const dia of diasOrdenados) {
        const dataDia = diario[dia];
        const fechaFormateada = moment(dia).format("dddd DD [de] MMMM");
        
        // Filtrar por tipo de actividad
        const tareas = dataDia.entradas.filter(e => e.tipo === 'tarea');
        const capacis = dataDia.entradas.filter(e => e.tipo === 'capacitacion');
        const rutinas = dataDia.entradas.filter(e => e.tipo === 'rutina');
        
        // Calcular totales por categoría
        const minTareas = tareas.reduce((acc, e) => acc + e.mins, 0);
        const minCapas = capacis.reduce((acc, e) => acc + e.mins, 0);
        const minRutinas = rutinas.reduce((acc, e) => acc + e.mins, 0);

        // Renderizar día
        dv.header(3, `${fechaFormateada} — Total: ${minsToTime(dataDia.totalMins)} hs`);

        const bloque = `> [!QUOTE] Actividades
>
>> [!TODO] 🛠️ Proyectos (${minsToTime(minTareas)})
${crearTablaMD(tareas, ">>")}
>
>> [!NOTE] 🎓 Capacitaciones (${minsToTime(minCapas)})
${crearTablaMD(capacis, ">>")}
>
>> [!EXAMPLE] ☕ Rutina (${minsToTime(minRutinas)})
${crearTablaMD(rutinas, ">>")}`;

        dv.paragraph(bloque);
    }
}
```
---
## Capacitaciones
`button-create-training`
> [!ABSTRACT] Panel de Control
> **Rango de Fechas:** `INPUT[date:reporte_inicio]` al `INPUT[date:reporte_fin]`
> **Horas reportadas:** 
> - Total `INPUT[text:ultimo_total_reportado]`(HH:MM)
> - Serfe `INPUT[text:ultimo_serfe_reportado]`(HH:MM)

> [!Note] Generador de Reporte de Capacitaciones (Tracker)
> ```dataviewjs
> // --- CONFIGURACIÓN Y NORMALIZACIÓN ---
>
> let rawInicio = dv.current().reporte_inicio;
> let rawFin = dv.current().reporte_fin;
> let rawUltimoTotal = dv.current().ultimo_total_reportado || "00:00";
> let rawUltimoSerfe = dv.current().ultimo_serfe_reportado || "00:00";
>
> function parseDate(input) {
>     if (!input) return moment();
>     if (input.ts) return moment(input.toJSDate());
>     return moment(input);
> }
>
> const fInicio = parseDate(rawInicio);
> const fFin = parseDate(rawFin);
>
> // --- CÁLCULO DE CATEGORÍAS ---
> // Buscamos todas las capacitaciones usando el tag maestro
> const allTrainingPages = dv.pages("#type/training");
> 
> const groupedPages = {};
>
> for (let p of allTrainingPages) {
>     // Agrupamos por la nueva propiedad "category"
>     let cat = p.category || "General";
>     
>     if (!groupedPages[cat]) {
>         groupedPages[cat] = [];
>     }
>     groupedPages[cat].push(p);
> }
>
> const sortedCategories = Object.keys(groupedPages).sort();
>
> // --- FUNCIONES DE TIEMPO ---
> function timeToMins(timeStr) {
>     if (!timeStr) return 0;
>     const parts = timeStr.toString().split(":");
>     return parts.length < 2 ? 0 : (parseInt(parts[0]) * 60) + parseInt(parts[1]);
> }
>
> function minsToTime(totalMins) {
>     let sign = totalMins < 0 ? "-" : "";
>     totalMins = Math.abs(Math.round(totalMins));
>     const h = Math.floor(totalMins / 60).toString().padStart(2, '0');
>     const m = (totalMins % 60).toString().padStart(2, '0');
>     return `${sign}${h}:${m}`;
> }
>
> // --- CÁLCULO PRINCIPAL ---
>
> let outputText = "";
> let totalCalculadoMins = 0; 
> let totalSerfeMins = 0;
>
> for (let catName of sortedCategories) {
>     const pages = groupedPages[catName];
>     if (pages.length === 0) continue;
>
>     let sectionText = "";
>     const sortedPages = pages.sort((a, b) => { 
> 	    const dateA = parseDate(a.created); 
> 	    const dateB = parseDate(b.created); 
> 	    return dateA - dateB; // Orden ascendente (más antiguo primero) 
>	});
>
>     for (let page of sortedPages) {
>         let minsPage = 0;
>         
>         // Sumamos las horas de la bitácora según el rango de fechas
>         if (page.fecha && page.horas) {
>             const fechas = Array.isArray(page.fecha) ? page.fecha : [page.fecha];
>             const horas = Array.isArray(page.horas) ? page.horas : [page.horas];
>             
>             fechas.forEach((f, i) => {
>                 const fechaEntrada = parseDate(f);
>                 if (fechaEntrada.isSameOrAfter(fInicio, 'day') && fechaEntrada.isSameOrBefore(fFin, 'day')) {
>                     minsPage += timeToMins(horas[i]);
>                 }
>             });
>         }
>
>         if (minsPage > 0) {
>             const timeStr = minsToTime(minsPage);
>             let pct = page.serfe_coverage_pct ?? 100;
>             const minsSerfePage = minsPage * (pct / 100);
>             
>             totalCalculadoMins += minsPage;
>             totalSerfeMins += minsSerfePage;
>
>             let lineDetail = pct < 100 ? ` (${pct}%)` : "";
>             const line = `- ${page.file.name.split(" - ").pop()}${lineDetail} `.padEnd(60, ".") + ` ${timeStr} hs`;
>             sectionText += line + "\n";
>         }
>     }
>
>     if (sectionText !== "") {
>         outputText += `Capacitaciones ${catName}\n` + sectionText + "\n";
>     }
> }
>
> // --- TOTALES Y SALIDA ---
>
> const ultimoTotalMins = timeToMins(rawUltimoTotal);
> const ultimoSerfeMins = timeToMins(rawUltimoSerfe);
> const avanceTotal = totalCalculadoMins - ultimoTotalMins;
> const avanceSerfe = totalSerfeMins - ultimoSerfeMins;
>
> outputText += `\nAvance de la semana:  ${minsToTime(avanceTotal)} hrs total / ${minsToTime(avanceSerfe)} hrs cubre Serfe`;
> outputText += `\nTotales acumulados:   ${minsToTime(totalCalculadoMins)} hrs total / ${minsToTime(totalSerfeMins)} hrs cubre Serfe`;
>
> dv.paragraph("\`\`\`text\n" + outputText + "\n\`\`\`");
> ```
### 🎓 Seguimiento de Capacitaciones
```dataviewjs
const pages = dv.pages('"20_Projects" and #type/training')
    .where(p => p.origin && p.origin.path === dv.current().file.path)
    .sort(p => [p.status, p.category, p.created], "asc");

dv.table(
    ["Nombre", "Categoría", "Estado", "Cobertura"],
    pages.map(p => [
        "[["+p.file.name+"|"+p.file.name.split(" - ").pop()+"]]",
        p.category || "-",
        p.status || "-",
        (p.serfe_coverage_pct || 0) + "%"
    ])
);
```
---
##  Proyectos
`button-new-project`
```dataview
LIST WITHOUT ID
	link(file.link, upper(project))
FROM "20_Projects" AND #type/project
WHERE origin = this.file.link
```
---
##  🏔️ Áreas
`button-new-area`
```dataview
LIST WITHOUT ID
	link(file.link, upper(area))
FROM "25_Areas" AND #type/area 
WHERE origin = this.file.link
```
---
## ☕ Registro de Tiempos del Día
```dataview
LIST
FROM "00_Inbox" AND #type/log
WHERE origin = this.file.link
SORT created DESC
LIMIT 5
```
---
# Notas
`button-new-inbox-note`
```dataview
LIST
FROM "00_Inbox" AND #type/inbox 
WHERE origin = this.file.link
```