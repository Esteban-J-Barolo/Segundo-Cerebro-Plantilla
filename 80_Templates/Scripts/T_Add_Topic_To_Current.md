<%*
const FOLDER = "10_Library";

// 1️⃣ Validar archivo activo
const file = app.workspace.getActiveFile();
if (!file) {
  new Notice("No hay una nota activa");
  return;
}

// 2️⃣ Recolectar topics existentes
const files = app.vault.getMarkdownFiles()
  .filter(f => f.path.startsWith(FOLDER));

let allTopics = new Set();

for (const f of files) {
  const cache = app.metadataCache.getFileCache(f);
  const fm = cache?.frontmatter;

  if (fm?.topic) {
    const arr = Array.isArray(fm.topic) ? fm.topic : [fm.topic];
    for (const t of arr) {
      if (typeof t === "string") allTopics.add(t);
    }
  }
}

let topicList = Array.from(allTopics).sort();
topicList.unshift("➕ Nuevo topic");

// 3️⃣ Selector
const selected = await tp.system.suggester(topicList, topicList);
if (!selected) return;

let newTopic =
  selected === "➕ Nuevo topic"
    ? await tp.system.prompt("Nombre del nuevo topic")
    : selected;

if (!newTopic) return;

// 4️⃣ Leer topics actuales (FORMA OFICIAL)
let currentTopics = [];

if (tp.frontmatter.topic) {
  currentTopics = Array.isArray(tp.frontmatter.topic)
    ? [...tp.frontmatter.topic]
    : [tp.frontmatter.topic];
}

// 5️⃣ Evitar duplicados
if (currentTopics.includes(newTopic)) {
  new Notice("Este topic ya está en la nota");
  return;
}
currentTopics.push(newTopic)

// 1. Leer todo el contenido actual del archivo
await app.fileManager.processFrontMatter(file, (fm) => {
  fm.topic = currentTopics;
});

new Notice(`Topic agregado: ${newTopic}`);
%>