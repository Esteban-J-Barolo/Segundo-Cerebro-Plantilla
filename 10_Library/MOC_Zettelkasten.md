# 🧠 Zettelkasten – Topics
```dataview
LIST WITHOUT ID link(file.link, topic) FROM #type/topic
```

## Topic Sin MOC
`button-create-moc-topic`
```dataviewjs
const ZETTEL_FOLDER = "10_Library";

// 1️⃣ Juntar topics y contar
// CAMBIO: Agregamos "AND -#type/topic" para contar solo notas normales, no los MOCs
const topicCount = {};
for (const page of dv.pages(`"${ZETTEL_FOLDER}" AND -#type/topic`)) {
  if (!page.topic) continue;

  const topics = Array.isArray(page.topic) ? page.topic : [page.topic];
  for (const t of topics) {
    topicCount[t] = (topicCount[t] || 0) + 1;
  }
}

// 2️⃣ Detectar MOCs existentes
// CAMBIO: Ahora buscamos cualquier archivo que tenga el tag #type/topic
const existingMOCs = new Set(
  dv.pages("#type/topic")
    .where(p => p.topic) // Nos aseguramos que tenga la propiedad topic llena
    .map(p => p.topic)
);

// 3️⃣ Filtrar los que NO tienen MOC (Igual que antes)
// Comparamos la lista de topics encontrados vs. la lista de MOCs existentes
const rows = Object.entries(topicCount)
  .filter(([topic]) => !existingMOCs.has(topic))
  .sort((a, b) => b[1] - a[1]);

// 4️⃣ Render
dv.table(
  ["Topic", "Cant. Notas"],
  rows.map(([topic, count]) => [
    // Opcional: Crear un link falso para que al hacer clic te sugiera crear el archivo
    `${topic}`, 
    count
  ])
);
```
## Notas Zettelkasten Sueltas
```dataviewjs
// Obtener todos los valores de topic de las notas #type/topic
const topicsConNota = new Set(
    dv.pages('#type/topic')
        .map(p => p.topic)
        .filter(t => t)
);

// Filtrar zettels que tienen TODOS sus topics sin nota #type/topic
const zettelsHuerfanos = dv.pages('#type/zettel')
    .where(p => p.topic && p.topic.length > 0 && p.topic.every(t => !topicsConNota.has(t)))
    .map(p => [p.file.link, p.topic.join(", ")]);

// Listar los zettels huérfanos
dv.table(["Zettel", "Topics huérfanos"], zettelsHuerfanos);
```
