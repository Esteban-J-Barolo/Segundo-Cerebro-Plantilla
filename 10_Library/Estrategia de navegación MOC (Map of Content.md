---
created: 2026-02-16 10:51
status: Evergreen
tags:
  - type/zettel
topic:
  - organización
 
origin: "[[Guía del Vault]]"
---
# 🧠 Idea: Estrategia de navegación MOC (Map of Content
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Evergreen), option(Developing), option(Deprecated)):status]`  
> 
> **Topic**: `BUTTON[btn-add-topic]`  

---
## Concepto
Un **MOC (Map of Content)** es, en esencia, una **nota que sirve como centro de mando o tablero de control** para un tema específico.

Imagina que tienes decenas de notas individuales sobre un tema. El MOC es la nota "maestra" que las agrupa, las organiza y les da sentido, permitiéndote navegar entre ellas sin perderte.
## ¿Cómo funciona un MOC?
A diferencia de una carpeta (donde guardas archivos) o una etiqueta (que solo agrupa), un MOC es **activo**. Tú decides el orden, la jerarquía y el contexto de los enlaces que pones en él.
### Sus funciones principales:
- **Curaduría:** Tú eliges qué notas son importantes para ese tema y cuáles no.
- **Navegación:** Funciona como el "Índice" de un libro que tú mismo estás escribiendo.
- **Contexto:** No es solo una lista de enlaces; puedes escribir frases entre los enlaces para explicar cómo se relacionan.
- **Flexibilidad:** Una misma nota puede aparecer en varios MOCs si es relevante para distintos temas.
## ¿Cuándo crear un MOC?
No se crean al principio. Un MOC nace de forma natural cuando:
1. Sientes que tienes **demasiadas notas** sobre un tema y te cuesta encontrarlas.
2. Empiezas a notar que hay una **estructura lógica** entre notas sueltas.
3. Necesitas una "mesa de trabajo" donde ver todas las piezas de un rompecabezas a la vez.

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