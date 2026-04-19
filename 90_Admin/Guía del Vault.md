# 📘 Manual Operativo del Vault
Este Vault separa la **naturaleza** de la nota de su **ubicación**, utilizando una arquitectura de tres capas: Logística, Inteligencia y Navegación.
## 📂 Estructura del Vault (PARA + Zettelkasten + MOC)

### 1. La Capa de Logística: [[Método PARA]] (Carpetas)
Las carpetas definen el **flujo de trabajo** y la urgencia de la información. No clasifican temas, sino niveles de compromiso:

| **Carpeta**      | **Propósito**                                                           |
| ---------------- | ----------------------------------------------------------------------- |
| `00_Inbox`       | Punto de entrada para capturas rápidas y notas sin procesar.            |
| `10_Library`     | El "cerebro". Contiene notas atómicas (`#type/zettel`) y MOCs de temas. |
| `20_Projects`    | Gestión de proyectos.                                                   |
| `25_Areas`       | Gestión de áreas (Trabajo, Facultad, etc.).                             |
| `30_Calendar`    | Notas diarias y planificación temporal.                                 |
| `80_Templates`   | La lógica del sistema. **Infraestructura**.                             |
| `90_Admin`       | Configuración técnica y manuales de uso.                                |
| `99_Attachments` | Assets. PDFs, Imágenes.                                                 |
La carpeta `80_Templates` es el **"Motor"** del sistema, ya que sin ella no podrías mantener los metadatos (YAML) que hacen que el resto funcione.
### 2. La Capa de Inteligencia: [[Método Zettelkasten|Zettelkasten]] (Notas Atómicas)
Esta capa vive principalmente en **`10_Library`**. Aquí es donde el sistema "piensa":
- **Notas Atómicas (`#type/zettel`)**: Son ideas individuales, breves y escritas con tus propias palabras.
- **Independencia**: A diferencia de las tareas o proyectos, estas notas son permanentes y no caducan; son los ladrillos de tu conocimiento personal.
### 3. La Capa de Navegación: [[Estrategia de navegación MOC (Map of Content|MOCs]] (Mapas de Contenido)
Los MOCs son los "pegamentos" que evitan que te pierdas en las carpetas. Funcionan como dashboards dinámicos (DataView):
- **MOC Central (Dashboard)**: Tu punto de control principal para ver todo el sistema de un vistazo.
- **MOCs de Estructura**: Utilizas plantillas específicas (`T_MOC_Area`, `T_MOC_Project`, `T_MOC_Topic`) para crear índices automáticos con **Dataview**.
- **Conexión**: Un MOC de un tópico en la librería puede conectar varias notas `zettel`, mientras que un MOC de proyecto organiza tus tareas y tickets por estado (Assigned, Testing, Resolved).
### Resumen del Ecosistema
| **Componente**   | **Función**                                                      | **Ubicación Principal**                  |
| ---------------- | ---------------------------------------------------------------- | ---------------------------------------- |
| **PARA**         | **Logística**: Organiza por "cuándo" se necesita la información. | `20_Projects`, `25_Areas`, `00_Inbox`    |
| **Zettelkasten** | **Cerebro**: Almacena "qué" sabes de forma atómica.              | `10_Library` (`#type/zettel`)            |
| **MOC**          | **Brújula**: Crea mapas para "encontrar" y conectar todo.        | Notas `MOC_` en sus respectivas carpetas |
| **Templates**    | **Motor**: Automatiza la creación de metadatos y lógica.         | `80_Templates`                           |

## Diccionario de Tags y Propiedades (La "Gramática" del Vault)
El **Diccionario de Tags y Propiedades** es el ADN de tu Vault. Sin estos metadatos, las notas serían texto plano; con ellos, se convierten en una base de datos dinámica que **Dataview** puede leer para generar tus tableros y MOCs automáticamente.

Aquí tienes el desglose de la "gramática" que has programado en tus plantillas:
### 1. Los Tags de Tipo (`#type/`)
Estos tags definen la **naturaleza** de la nota, no su contenido. Es la respuesta a la pregunta: _"¿Qué es este archivo?"_.

| **Tag**          | **Propósito**                                                | **Plantilla Asociada** |
| ---------------- | ------------------------------------------------------------ | ---------------------- |
| `#type/inbox`    | Notas rápidas que aún no han sido procesadas.                | `T_Create_Inbox_Note`  |
| `#type/zettel`   | Ideas atómicas y conocimiento permanente en la librería.     | `T_Create_Zettel`      |
| `#type/task`     | Acciones concretas con seguimiento de tiempo y prioridad.    | `T_Create_Task`        |
| `#type/project`  | MOCs que coordinan esfuerzos con una meta clara.             | `T_MOC_Project`        |
| `#type/area`     | MOCs de responsabilidades a largo plazo (Trabajo, Facultad). | `T_MOC_Area`           |
| `#type/topic`    | Mapas de contenido que agrupan temas de conocimiento.        | `T_MOC_Topic`          |
| `#type/context`  | Información de soporte que da marco a un proyecto o área.    | `T_Create_Context`     |
| `#type/detail`   | Desgloses específicos de tareas o clases de capacitación.    | `T_Create_Detail`      |
| `#type/training` | Registro de cursos y capacitaciones activas.                 | `T_Create_Training`    |
| `#type/log`      | Bitácoras de tiempos muertos o registros de rutina.          | `T_Create_Routine_Log` |
### 2. Propiedades Universales (Frontmatter YAML)
Son los campos que aparecen al inicio de cada nota y permiten que el sistema "sepa" cómo conectarse entre sí.
- **`created`**: Fecha y hora de creación automática.
- **`origin`**: Es la propiedad más crítica. Almacena el enlace a la nota "padre" (MOC), permitiendo que la nota aparezca automáticamente en los listados del proyecto o área correspondiente.
- **`status`**: Define en qué etapa está la nota. Varía según el tipo:
    - **Inbox**: `Process`, `Review`, `OnHold`, `Dismissed`.
    - **Task**: `New`, `Assigned`, `Feedback`, `Testing`, `Resolved`, `Closed`.
    - **Zettel**: `Evergreen`, `Developing`, `Deprecated`.
### 3. Propiedades Específicas (Funcionalidad Avanzada)
Tu sistema incluye campos técnicos especializados, especialmente para la gestión de trabajo y aprendizaje.
#### Para el Desarrollo y Estimación (`#type/task`)
- **`ticket_id`**: El número identificador del tracker de trabajo.
- **`est_E`, `est_EE`, `est_P`**: Tríada de estimación (Optimista, Esperada, Pesimista) para calcular riesgos.
- **`confidence_level`**: Tu grado de seguridad sobre la estimación dada.
- **`tech_stack`**: Las herramientas técnicas (lenguajes, frameworks) involucradas.
- **`worked_similar_before`**: Booleano (Sí/No) para que la IA ajuste la validación de tiempo.
#### Para el Conocimiento (`#type/zettel` y `#type/training`)
- **`topic`**: Define el área temática a la que pertenece la idea.
- **`category`**: Clasificación para capacitaciones (ej. Habilidades Blandas, IT).
- **`serfe_coverage_pct`**: Porcentaje de cobertura de la capacitación.
### 4. Por qué es importante respetar esta gramática
Si creas una nota de tarea y olvidas poner el tag `#type/task`, no aparecerá en el MOC de tu proyecto. Si el campo `origin` está vacío, la nota quedará "huérfana" y será difícil de encontrar mediante los sistemas automáticos que has diseñado.
> **Regla de oro:** Siempre usa las plantillas de la carpeta `80_Templates`. Ellas se encargan de escribir esta gramática por ti. Los botones usan estas plantillas.
## Flujo de Creación de Notas (El "Ciclo de Vida")
El **Ciclo de Vida** de una nota en tu Vault es el proceso por el cual una idea bruta se transforma en una tarea ejecutable o en conocimiento permanente. Gracias a tus plantillas y botones, este flujo es semi-automático y asegura que ninguna nota quede "huérfana".
Para el área "Trabajo" -> "Serfe" se creo un MOC personalizado.
![[Diagrama de creación de archivos - 20260216130407.png]]
![[Diagrama de creación de archivos v2 - 20260322222627.png]]
Aquí tienes el flujo desglosado en cuatro etapas principales:
### 1. Captura (Entrada al Sistema)
Todo comienza con una chispa de información que debe ser guardada rápidamente para no perder el foco.
- **Ubicación**: Carpeta `00_Inbox`.
- **Herramienta**: Plantilla `T_Create_Inbox_Note`.
- **Estado inicial**: La nota nace con el estado `Process`.
- **Acción**: Capturas el concepto de forma rápida en la sección de "Notas". El sistema registra automáticamente la fecha de creación y el origen si fue disparada desde otra nota.
### 2. Clasificación (El Triaje)
Una vez que tienes tiempo, revisas tu `00_Inbox`. Aquí decides el destino de la nota según su naturaleza.
- **Si es conocimiento puro**: Se destila hacia la `10_Library` como una nota atómica con `button-create-zettel`.
- **Si es descartable o ya fue procesada**: Cambias el estado a `Dismissed` en el Panel de Control.
### 3. Ejecución o Maduración (Refinamiento)
Dependiendo de la carpeta de destino, la nota sigue un camino de evolución distinto:
#### A. Camino de la Tarea (`#type/task`)
- **Análisis**: Utilizas la plantilla `T_Create_Task` para desglosar el ticket, definir el _stack_ tecnológico y realizar la estimación optimista/pesimista.
- **Estados**: La nota evoluciona de `New` → `Assigned` → `Testing` → `Resolved`.
- **Registro**: Usas la bitácora de ejecución para trackear el tiempo real invertido mediante botones de tiempo.
#### B. Camino del Conocimiento (`#type/zettel`)
- **Atomicidad**: Usas `T_Create_Zettel` para redactar la idea con tus propias palabras.
- **Clasificación**: Seleccionas un `topic` existente o creas uno nuevo mediante el _suggester_ de Templater.
- **Maduración**: La nota comienza como `Developing` y, una vez que la idea está completamente clara y conectada, pasa a ser `Evergreen`.
### 4. Conexión y Navegación ([[Estrategia de navegación MOC (Map of Content|MOCs]])
Una nota nunca vive sola. El último paso de su ciclo de vida es integrarse en el mapa mental del Vault.
- **Vinculación automática**: Gracias a la propiedad `origin`, la nota aparece instantáneamente en los listados de Dataview de su MOC correspondiente (ya sea un Proyecto, Área o Tópico).
- **Interconexión**: Desde una nota de conocimiento (`zettel`), puedes usar botones para crear notas relacionadas o contextos adicionales, manteniendo la trazabilidad del origen.
## Tipos de Notas y Plantillas (El "Catálogo")
El **Catálogo de Notas y Plantillas** es el inventario de la "fábrica" de tu Vault. Cada plantilla en la carpeta `80_Templates` tiene un propósito específico y una lógica de automatización diseñada para que no tengas que escribir metadatos manualmente.

Aquí tienes el desglose de tus herramientas según su función:
### 1. Gestión y Navegación (MOCs)
Estas plantillas actúan como "estaciones centrales" que agrupan otras notas mediante Dataview.
- **Project MOC (`T_MOC_Project`)**: Gestiona el ciclo de vida de un proyecto. Separa automáticamente las tareas por estado (**Assigned**, **Feedback**, **Testing**, **Resolved**, **Closed**) y visualiza el _deadline_ y la prioridad.
- **Area MOC (`T_MOC_Area`)**: Sirve de índice para una responsabilidad a largo plazo. Lista tanto los proyectos activos vinculados como otras sub-áreas relacionadas.
- **Topic MOC (`T_MOC_Topic`)**: La brújula de tu librería. Agrupa todas las notas `#type/zettel` que comparten un mismo tópico, permitiendo que el conocimiento emerja sin necesidad de subcarpetas.
### 2. Ejecución y Trabajo Técnico
Estas son las plantillas con mayor carga lógica, diseñadas para tu perfil de desarrollador.
- **Task/Ticket (`T_Create_Task`)**: Tu herramienta más compleja. Incluye:
    - Vinculación directa al tracker externo mediante `ticket_id`.
    - Triángulo de estimación de tiempos (**Optimista**, **Esperado**, **Pesimista**).
    - Un **Generador de Prompt para IA** que toma todo el contexto técnico (stack, complejidad, bloqueadores) para validar tus horas estimadas.
- **Context (`T_Create_Context`)**: Define el marco de un proyecto. Se usa para descripciones generales y para listar "detalles" y "contextos" hijos vinculados.
- **Detail (`T_Create_Detail`)**: El nivel más bajo de desglose. Se usa para registrar especificaciones técnicas o referencias puntuales dentro de una tarea o proyecto.
### 3. Conocimiento y Aprendizaje
Orientadas a la metodología Zettelkasten y al crecimiento profesional.
- **Zettel Note (`T_Create_Zettel`)**: Para ideas atómicas. Al crearla, te pregunta (vía _suggester_) a qué tópico pertenece o si deseas crear uno nuevo.
- **Training (`T_Create_Training`)**: Especializada en cursos. Permite categorizar el aprendizaje (ej. Habilidades Blandas, IT) y lleva una bitácora de sesiones con cálculo automático del tiempo total invertido.
## ✅ Conclusión: Reglas de Oro
1. **Nunca crear notas manualmente**: Usar siempre los **Buttons** y **Templates** para garantizar la integridad de los metadatos.
2. **Principio de Atomicidad**: Una nota Zettel debe contener una sola idea clara escrita con palabras propias.
3. **Mantenimiento**: Archivar proyectos finalizados cambiando el estado a `finalizado` para limpiar el Dashboard.
## ⚙️ Sección: Mantenimiento Técnico y Seguridad
Esta sección es vital para evitar que rompas la automatización por accidente al editar archivos de infraestructura.
- **Gestión de Infraestructura**: Todas las plantillas residen en `80_Templates`. No se deben editar directamente sin verificar el código Javascript interno (`<% ... %>`), ya que un error de sintaxis inhabilitará la creación de nuevas notas.
- **Control de Interfaz**: La configuración lógica de los botones está centralizada en [[Configuracion_Botones]]. Cualquier cambio en el comportamiento de los disparadores (triggers) debe realizarse allí para mantener la consistencia en todo el Vault.
- **Índice de Inteligencia**: El archivo [[MOC_Zettelkasten]] actúa como el "mapa de carreteras" de la biblioteca. Es el punto de control para revisar qué temas (`topics`) están creciendo y requieren una estructura más profunda.
## 🛠️ Sección: Guía de Personalización de Proyectos
Dado que manejas proyectos técnicos diversos (como **n8n**, **Odoo** o **bots de trading**), esta sección te ayuda a decidir cuándo crear una herramienta nueva y cuándo adaptar la existente.
### Estrategia de Adaptación
Para evitar la saturación de la carpeta `80_Templates`, sigue esta jerarquía de decisión:
1. **Modificación Ad-hoc (Recomendado)**: Usa la plantilla genérica de proyecto y añade secciones manuales debajo de las consultas de Dataview. Al ser Markdown libre, puedes insertar tablas, listas de tareas de n8n o diagramas específicos sin afectar los scripts automáticos.
2. **Plantillas Especializadas**: Solo crea una nueva plantilla (ej: `T_Proyecto_Tecnico`) si repites la misma estructura manual más de 5 veces.
3. **Regla de Integridad**: Cualquier plantilla nueva **debe** conservar las propiedades `tags: [type/context]` y `origin` para asegurar que los MOCs de nivel superior sigan rastreando la información correctamente.
# Templates
```dataview
TABLE
FROM "80_Templates"
```
