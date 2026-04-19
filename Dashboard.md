> [!todo] 🏛️ Centro de Mando `button-new-area`
> 
> ```dataview
> TABLE WITHOUT ID
> 	link(file.link, upper(area)) AS "Área",
> 	dateformat(file.mtime, "dd-MM-yyyy") AS "Actualizado"
> FROM "25_Areas" AND #type/area 
> WHERE origin = this.file.link
> SORT file.name ASC
> ```

> [!todo] 📥 Inbox `button-new-inbox-note`
> ```dataview
> LIST
> FROM "00_Inbox"
> SORT file.ctime DESC
> LIMIT 5
> ```

> [!todo] 🧠 Últimas Ideas Procesadas | [[MOC_Zettelkasten|🗃️ Explorar Conocimiento]]
> ```dataview
> LIST
> FROM #type/zettel 
> SORT file.ctime DESC
> LIMIT 5
> ```

> [!help]- 📘 Recursos de Ayuda
> - [[Guía del Vault|📖 Cómo usar este sistema]]
> - [[Configuracion_Botones|⚙️ Configuración de Botones]]