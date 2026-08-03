# CLAUDE.md — stxt-impl

## Qué es este repositorio

Descripción en **pseudocódigo neutro de lenguaje** de la implementación de STXT (Semantic Text).
Es el plano ("blueprint") para portar STXT a nuevas plataformas: un implementador debe poder
escribir un port conforme leyendo únicamente este repositorio y la especificación, sin leer
TypeScript ni Java.

La guía de estilo del pseudolenguaje está en [README.md](README.md) (keywords, tipos,
convenciones de nombres). Todo fichero `.txt` de este repo la sigue.

## Ecosistema STXT

Repos hermanos en `C:\desarrollo\workspace_java` (org GitHub `stxt-lang`, web https://stxt.dev):

| Repo | Rol |
|---|---|
| `../stxt-web` | **Especificación normativa**, escrita en el propio STXT. Canónico en `es/`: `stxt-core-ref.stxt`, `stxt-schema-ref.stxt`, `stxt-template-ref.stxt`, `stxt-discovery-ref.stxt`. Espejo en `en/`. Sin números de versión: se rastrea con `Last modif`. También contiene el corpus de ejemplos que usan las suites de conformidad de js y java. |
| `../stxt-js` | Implementación oficial. npm `@stxt-lang/core` (TypeScript). Su `CLAUDE.md` documenta la arquitectura. |
| `../stxt-java` | Port Java con paridad deliberada con js (mismos códigos de error, versiones alineadas). Maven Central `dev.stxt:stxt-core`. |
| `stxt-impl` (este) | Pseudocódigo agnóstico de la implementación. |
| Otros | `../stxt-vscode` (extensión), `stxt-cli`, `../stxt-python` (esqueleto, primer candidato a usar este blueprint), `../stxt-cms`. |

**Orden de autoridad ante ambigüedades (objetivo):**

```
spec (stxt-web)  →  stxt-impl (este repo)  →  stxt-js  →  stxt-java
```

Hoy el orden vigente es spec → js → java; este repo entrará en segunda posición **cuando esté
completo y correcto**. En ese momento se actualizarán los CLAUDE.md de los otros repos y la
spec para reflejarlo. Hasta entonces, no anunciar ese estatus fuera de aquí.

## Alcance normativo (decisión cerrada, 2026-08-03)

La frontera es **"semántica del lenguaje"**, no "lo que hace stxt-js".

**Dentro (este repo lo describe y es normativo):**

- `core/` — parser sintáctico, nodos, indentación, nombres y namespaces.
- `schema/` — validación semántica, tipos, providers de schema.
- `template/` — compilación de `@stxt.template` a schema.
- `discovery/` — resolución de definiciones en filesystem (STXT-DISCOVERY-SPEC). **Pendiente de escribir.**
- **NodeWriter** (serializador) — el round-trip (escribir y reparsear produce el mismo árbol) es
  una propiedad semántica del lenguaje. **Pendiente de escribir.**
- **Interfaces `Observer` y `Validator`** con sus puntos de enganche al parser — definen *cuándo*
  se procesa/valida un nodo (validación en streaming al cerrar cada nodo), que es comportamiento
  observable. **Pendiente de escribir como contrato explícito.**

**Fuera (libertad de diseño de cada port):**

- Fachadas de conveniencia (`STXT.parser()`...), `UnifiedSchemaProvider`, caches más allá de
  `SchemaProviderCache`, adaptadores concretos de filesystem/entorno.
- Como mucho, una nota de una línea: "cada implementación puede ofrecer una fachada idiomática".

## Decisiones semánticas resueltas (confirmadas por Joan, 2026-08-03)

1. **Nombre canónico: conserva acentos, pasa a minúsculas.**
   `Año == año`, y ambos son **distintos** de `ano`/`ANO` (que son iguales entre sí).
   Es el modelo IDN de la spec: NFC + minúsculas + compactación de separadores.
   ⚠️ El pseudocódigo actual está **mal**: `normalizeChars` en
   [core/string_utils.txt](core/string_utils.txt) describe NFKD + eliminación de diacríticos +
   `[^a-z0-9] -> '-'`, que destruiría los acentos. Hay que corregirlo (y revisar el comentario
   equivalente en [core/node.txt](core/node.txt)).

2. **Hijos: se valida siempre la cardinalidad (min/max); el orden de aparición NO se valida.**
   El enfoque actual de [schema/schema_validator.txt](schema/schema_validator.txt)
   (contar apariciones, independiente del orden) es **correcto**; no tocar.

## Estado actual y trabajo pendiente

Análisis hecho el 2026-08-03 comparando contra las specs (`Last modif: 2026-08-02`) y
stxt-js 0.6.0 (npm, 2026-08-02). Pendiente, en orden recomendado:

1. **Corregir `normalizeChars`** según la decisión semántica 1 (única corrección de fondo).
2. **Regla tab/espacio**: la spec (desde 2026-07-25) dice que mezclar tab y espacios en la misma
   línea de indentación es error de parseo. [core/line_indent.txt](core/line_indent.txt) hace lo
   contrario (resetea el contador de espacios al ver un TAB y continúa). Corregir.
3. **Tipos que faltan** en [schema/types.txt](schema/types.txt): TIME, UUID, BINARY (anotados como
   no registrados) y **MARKDOWN** (añadido a la spec el 2026-07-25). El objetivo es paridad con la
   spec (§ tipos de STXT-SCHEMA-SPEC); java registra 19, js 18.
4. **`ParseResult` / modo acumulador de errores**: `parse()` es fail-fast; añadir la variante
   `parseResult()` que acumula errores + nodos, como en js/java.
5. **Contratos `Observer` / `Validator`**: nuevo fichero en `core/` (o `processors/`) definiendo
   las interfaces (`onCreate`, `onTextLine`, `onComment`, `onFinish`; `validate` al cerrar nodo)
   y el registro en el parser (`registerObserver` / `registerValidator`).
6. **`discovery/`**: módulo nuevo completo según STXT-DISCOVERY-SPEC. Referencia: `src/discovery/`
   de stxt-js (diseño ya agnóstico: filesystem y entorno inyectados, errores coleccionados en vez
   de lanzados, precedencia por namespace, cadena `.stxt/` ancestros → usuario → sistema,
   override `STXT_PATH`, duplicados en el mismo nivel = error).
7. **`NodeWriter`**: serialización a STXT con estilo de indentación (tabs | 4 espacios) y garantía
   de round-trip.
8. **Traducción al inglés**: el repo debe quedar íntegramente en inglés — README.md (guía de
   estilo) y todos los comentarios de los ficheros `.txt` (hoy mezclan español e inglés).
   Estrategia: los ficheros que se toquen en los puntos 1-7 se traducen al revisarlos; al final,
   pasada de barrido sobre los que no se hayan tocado, README.md y este CLAUDE.md.
9. **Trazabilidad**: referencias a secciones de la spec en cada fichero (ya hay alguna, estilo
   "STXT-SCHEMA-SPEC §9.1") y una tabla fichero ↔ clase stxt-js ↔ clase stxt-java.
10. **Oficialización** (al final, cuando todo lo anterior esté bien): mencionar stxt-impl desde
    stxt-web y los CLAUDE.md de js/java; valorar adoptar el versionado común (js va por 0.6.0)
    para que "misma versión = mismo comportamiento" incluya el pseudocódigo.

## Estructura actual

```
README.md                       Guía de estilo del pseudolenguaje
core/
  constants.txt                 COMMENT_CHAR, TAB_SPACES=4, separadores...
  line_indent.txt               parseLine: indentación → LineIndent
  name_namespace.txt            NameNamespaceParser: "Nombre (ns)" → (name, namespace)
  node.txt                      Clase Node (árbol inmutable tras freeze)
  parser.txt                    parse(): pila de nodos abiertos por nivel
  platform.txt                  Funciones dependientes de plataforma (declaradas, sin cuerpo)
  string_utils.txt              Utilidades de cadena (normalizeChars PENDIENTE de corregir)
  validations.txt               validateNamespaceFormat
schema/
  schema.txt                    Schema: namespace + NodeDefinitions
  node_definition.txt           NodeDefinition: tipo, hijos, valores ENUM
  child_definition.txt          ChildDefinition: cardinalidad min/max
  schema_parser.txt             Documento @stxt.schema → Schema
  schema_provider.txt           SchemaProvider + Cache + Resources + Meta (bootstrap)
  schema_validator.txt          Valida nodo contra schema (tipo + cardinalidades, sin orden)
  types.txt                     TypeRegistry + tipos (INCOMPLETO: faltan TIME, UUID, BINARY, MARKDOWN)
template/
  template_parser.txt           Documento @stxt.template → Schema
  child_line.txt                ChildLine: (cardinalidad) TIPO [valores]
  child_line_parser.txt         Parser del RuleSpec de Structure >>
  template_schema_provider.txt  Providers de template + meta-template
```

## Convenciones del repo

- Ficheros de pseudocódigo en `.txt`, siguiendo estrictamente la guía de [README.md](README.md)
  (keywords en MAYÚSCULAS inglés, variables snake_case, comentarios `#`).
- **Idiomas**: la conversación con Joan es siempre en español (de España), pero **todo el
  contenido del repositorio debe estar en inglés**: README.md, comentarios de los `.txt`,
  identificadores, mensajes de error y este CLAUDE.md (se traduce en la pasada final, punto 8
  del trabajo pendiente). Hoy los comentarios mezclan español e inglés: al tocar un fichero,
  dejarlo en inglés.
- Códigos de error en MAYÚSCULAS, idénticos a los de stxt-js/stxt-java.
- Commits: mensajes cortos; `M+` es la convención local para cambios menores.
