<%*
async function getAreaChain(app, startName) {
  const chain = [];
  let current = startName;

  while (current) {
    const file = app.vault.getMarkdownFiles().find(f => f.basename === current);
    if (!file) break;

    const cache = app.metadataCache.getFileCache(file);
    const fm = cache?.frontmatter;
    if (!fm) break;

    const area = fm.area;
    if (area) chain.unshift(area.toLowerCase());

    // Limpiar "[[NombreNota]]" o '"[[NombreNota]]"'
    const rawOrigin = fm.origin;
    const origin = rawOrigin
      ? rawOrigin.toString().replace(/[\[\]"]/g, "").trim()
      : null;

    if (!origin || origin === "Dashboard") break;
    current = origin;
  }

  return chain;
}

async function getRecurringDate(tp) {
  const freqChoice = await tp.system.suggester(
    ["🔁 Cada día", "🔁 Cada semana", "🔁 Cada mes", "🔁 Cada X días", "✏️ Personalizada (inglés)..."],
    ["day", "week", "month", "xdays", "custom"],
    true,
    "¿Con qué frecuencia?"
  );

  if (freqChoice === "day") return "every day";

  if (freqChoice === "week") {
    const dayChoice = await tp.system.suggester(
      ["Lunes", "Martes", "Miércoles", "Jueves", "Viernes", "Sábado", "Domingo"],
      ["monday", "tuesday", "wednesday", "thursday", "friday", "saturday", "sunday"],
      true,
      "¿Qué día de la semana?"
    );
    return `every ${dayChoice}`;
  }

  if (freqChoice === "month") {
    const dayNum = await tp.system.prompt("¿Qué día del mes? (1-31)", "1");
    return `every month on the ${dayNum}`;
  }

  if (freqChoice === "xdays") {
    const every = await tp.system.prompt("¿Cada cuántos días?", "2");
    return `every ${every} days`;
  }

  if (freqChoice === "custom") {
    return await tp.system.prompt("Ingresá la frecuencia en inglés", "every monday, wednesday");
  }
}

try {
  const token = (await app.vault.adapter.read(".obsidian/todoist-token")).trim();
  const fm = tp.frontmatter;
	const tags = fm?.tags ?? [];
	const isProject = tags.includes("type/project");
	const isArea = tags.includes("type/area");
	
	let sectionName, labels;
	
	if (isProject) {
	  // Subir por origin hasta encontrar el área raíz
	  const rawOrigin = fm.origin?.toString().replace(/[\[\]"]/g, "").trim();
	  const chain = await getAreaChain(app, rawOrigin);
	  if (!chain.length) {
	    new Notice("Error: No se pudo determinar el área del proyecto");
	    return;
	  }
	  sectionName = chain[0];
	  const projectName = fm.project?.toLowerCase();
	  // Labels: cadena de áreas + el proyecto
	  labels = [
	    ...chain.map(a => a.charAt(0).toUpperCase() + a.slice(1)),
	    projectName.charAt(0).toUpperCase() + projectName.slice(1)
	  ];
	} else if (isArea) {
	  const chain = await getAreaChain(app, tp.file.title);
	  if (!chain.length) {
	    new Notice("Error: No se pudo determinar la cadena de áreas");
	    return;
	  }
	  sectionName = chain[0];
	  labels = chain.map(a => a.charAt(0).toUpperCase() + a.slice(1));
	} else {
	  new Notice("Error: La nota no tiene tag 'type/project' ni 'type/area'");
	  return;
	}

  // Pedir datos de la tarea
  const taskName = await tp.system.prompt("Nombre de la tarea");
  if (!taskName) return;

  const dateChoice = await tp.system.suggester(
	  ["📅 Hoy", "📅 Mañana", "📅 Esta semana", "📅 Próxima semana", "🔁 Repetitiva...", "📅 Sin fecha", "✏️ Ingresar fecha..."],
	  ["today", "tomorrow", "this week", "next week", "recurring", null, "custom"],
	  true,
	  "Fecha de vencimiento"
	);

let dueDate = null;
let isRecurring = false;

if (dateChoice === "recurring") {
  dueDate = await getRecurringDate(tp);
  isRecurring = true;
} else if (dateChoice === "custom") {
  dueDate = await tp.system.prompt("Ingresá la fecha", "today");
} else {
  dueDate = dateChoice;
}

  const priority = await tp.system.suggester(
    ["Sin prioridad", "Prioridad 1 🔴", "Prioridad 2 🟠", "Prioridad 3 🔵"],
    ["1", "4", "3", "2"],
    true,
    "Prioridad"
  );

  // Obtener proyecto Obsidian y sección
  const syncRes = await requestUrl({
    url: "https://api.todoist.com/api/v1/sync",
    method: "POST",
    headers: { "Authorization": `Bearer ${token}` },
    contentType: "application/x-www-form-urlencoded",
    body: "sync_token=*&resource_types=%5B%22projects%22%2C%22sections%22%5D"
  });

  const project = syncRes.json.projects.find(p => p.name === "Obsidian");
  if (!project) {
    new Notice("Error: No se encontró el proyecto Obsidian");
    return;
  }

  const section = syncRes.json.sections.find(
    s => s.project_id === project.id && s.name.toLowerCase() === sectionName
  );
  if (!section) {
    new Notice(`Error: No se encontró la sección "${sectionName}"`);
    return;
  }

  const noteName = tp.file.title; 
  const obsidianLink = `obsidian://open?vault=${encodeURIComponent(app.vault.getName())}&file=${encodeURIComponent(tp.file.path(true))}`; 
  const description = `[${noteName}](${obsidianLink})`;
  
  // Construir tarea
  const taskArgs = {
    content: taskName,
    description: description,
    project_id: project.id,
    section_id: section.id,
    priority: parseInt(priority ?? "1"),
    labels,
    ...(dueDate ? { due: { string: dueDate, lang: "en", ...(isRecurring ? { is_recurring: true } : {}) } } : {})
  };

  await requestUrl({
    url: "https://api.todoist.com/api/v1/sync",
    method: "POST",
    headers: { "Authorization": `Bearer ${token}` },
    contentType: "application/x-www-form-urlencoded",
    body: `commands=${encodeURIComponent(JSON.stringify([{
      type: "item_add",
      temp_id: crypto.randomUUID(),
      uuid: crypto.randomUUID(),
      args: taskArgs
    }]))}`
  });

  new Notice(`✓ Tarea "${taskName}" creada en Obsidian/${sectionName}`);

} catch(e) {
  new Notice("Error: " + e.message.substring(0, 100));
  console.error(e);
}
%>