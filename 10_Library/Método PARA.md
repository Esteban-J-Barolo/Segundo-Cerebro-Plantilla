---
created: 2026-02-16 10:03
status: Evergreen
tags:
  - type/zettel
topic:
  - organización
origin: "[[Guía del Vault]]"
---
# 🧠 Idea: Método PARA
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Evergreen), option(Developing), option(Deprecated)):status]`  
> 
> **Topic**: `BUTTON[btn-add-topic]`  

---
## Concepto
El método **PARA** es un sistema de organización de información digital creado por **Tiago Forte** (autor de _Building a Second Brain_). A diferencia de los sistemas tradicionales que clasifican la información por "temas", PARA organiza la información por **producibilidad** o nivel de acción.
La premisa es simple: no guardas las cosas según _qué son_, sino según _para qué las necesitas ahora_.
### Las 4 Categorías de PARA
#### 1. Projects (Proyectos) 🚀
Son esfuerzos a **corto plazo** que tienen una **fecha de finalización** clara y un resultado específico.
- **Ejemplos:** "Terminar el informe trimestral", "Planear el viaje a Japón", "Rediseñar la página web".
- **Clave:** Si no tiene una fecha de entrega o un "visto bueno" final, no es un proyecto.
#### 2. Areas (Áreas) ⚖️
Son responsabilidades con un **estándar que mantener** a lo largo del tiempo. No tienen una fecha de fin, son áreas de tu vida que requieren atención continua.
- **Ejemplos:** Salud, Finanzas, Marketing, Relaciones, Desarrollo Profesional.
- **Clave:** Son "roles" que desempeñas. Si dejas de prestarles atención, la calidad de tu vida o trabajo baja.
#### 3. Resources (Recursos) 📚
Son temas de **interés general** o bibliotecas de consulta. Es información que te gusta o te parece útil, pero que no está ligada a un proyecto o área activa en este momento.
- **Ejemplos:** "Interés en la cocina orgánica", "Notas sobre inteligencia artificial", "Recursos de diseño CSS".
- **Clave:** Es tu "enciclopedia" personal. Si un recurso se vuelve relevante para un proyecto, lo mueves a la carpeta de Proyectos.
#### 4. Archives (Archivo) 🗄️
Es el cementerio (activo) de las otras tres categorías. Aquí va todo lo que ya se completó o que ya no te interesa seguir.
- **Ejemplos:** Proyectos terminados, áreas que ya no son tu responsabilidad, recursos que ya no te sirven.
- **Clave:** No borres nada, solo archívalo. Así liberas tu espacio mental sin perder la información.
### Diferencias Clave: ¿Proyecto o Área?
| **Característica** | **Proyecto (Project)**       | **Área (Area)**         |
| ------------------ | ---------------------------- | ----------------------- |
| **Meta**           | Un resultado específico.     | Un estándar de calidad. |
| **Tiempo**         | Tiene fecha de fin.          | Es indefinido.          |
| **Ejemplo**        | "Correr un maratón en mayo". | "Salud y ejercicio".    |
| **Acción**         | Se completa.                 | Se mantiene.            |

---
## Notas relacionadas
`button-create-zettel`
```dataview
LIST
FROM "10_Library"
WHERE origin = this.file.link
```
---
`button-delete-note`