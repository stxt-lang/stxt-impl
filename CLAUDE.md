# CLAUDE.md — stxt-impl

## Qué es este repositorio

Una descripción de pseudocódigo **neutra respecto al lenguaje** de la implementación de STXT (Semantic Text).
Es la plantilla para portar STXT a nuevas plataformas: un implementador debe poder
escribir un puerto conforme leyendo únicamente este repositorio y la especificación,
sin tener que leer TypeScript ni Java.

La guía de estilo del pseudolenguaje vive en [README.md](README.md) (palabras clave,
tipos, convenciones de nombres). Cada archivo `.txt` de este repositorio sigue esa guía.

## El ecosistema STXT

Repositorios hermanos bajo `C:\desarrollo\workspace_java` (organización GitHub `stxt-lang`, web https://stxt.dev):

| Repositorio | Papel |
|---|---|
| `../stxt-web` | **Especificación normativa**, escrita en STXT. Canonical en `es/`: `stxt-core-ref.stxt`, `stxt-schema-ref.stxt`, `stxt-template-ref.stxt`, `stxt-discovery-ref.stxt`. Espejo en inglés en `en/`. Sin números de versión: se rastrea mediante `Last modif`. Además contiene el corpus de ejemplos utilizado por las suites de conformidad de js y java. |
| `../stxt-js` | Implementación oficial. npm `@stxt-lang/core` (TypeScript). Su `CLAUDE.md` documenta la arquitectura. |
| `../stxt-java` | Puerto a Java con paridad deliberada con js (mismos códigos de error, versiones alineadas). Maven Central `dev.stxt:stxt-core`. |
| `stxt-impl` (este) | Pseudocódigo neutro respecto al lenguaje de la implementación. |
| Otros | `../stxt-vscode` (extensión), `stxt-cli`, `../stxt-python` (esqueleto, primer candidato para usar esta plantilla), `../stxt-cms`. |

**Orden de autoridad para las ambigüedades (objetivo):**

```
spec (stxt-web)  →  stxt-impl (este repositorio)  →  stxt-js  →  stxt-java
```

El orden vigente actualmente es spec → js → java; este repositorio entrará en segunda posición
**una vez esté completo y correcto**. En ese momento, el CLAUDE.md de los otros repositorios y
la especificación se actualizarán para reflejarlo. Mientras tanto, no anuncies ese estado
en ningún otro sitio.

## Alcance normativo (decisión cerrada, 2026-08-03)

El límite es **"la semántica del lenguaje"**, no "lo que haga stxt-js".

**En (esto lo describe este repositorio y es normativo):**

- `core/` — analizador sintáctico, nodos, sangría, nombres y espacios de nombres, `NodeWriter`.
- `exceptions/` — la jerarquía de excepciones y su disciplina de códigos de error.
- `processors/` — los contratos `Observer` y `Validator` y sus puntos de enganche en
  el analizador (definen *cuándo* se procesa/valida un nodo — validación en streaming
  cuando cada nodo se cierra — lo cual es un comportamiento observable).
- `schema/` — validación semántica, tipos, proveedores de esquemas.
- `template/` — compilación de `@stxt.template` en un esquema.
- `discovery/` — resolución de definiciones en el sistema de archivos (STXT-DISCOVERY-SPEC).
- **NodeWriter** — la vuelta a la forma original (escribir y volver a analizar da el mismo árbol) es una
  propiedad semántica del lenguaje.

**Fuera (libertad de diseño de cada puerto):**

- Fachadas de comodidad (`STXT.parser()`...), `UnifiedSchemaProvider`, cachés y
  cargadores de recursos (`SchemaProviderCache`/`ResourcesLoader`, patrón de stxt-java),
  adaptadores concretos de sistema de archivos/entorno (Node `fs`, `vscode.workspace.fs`...).
- Como máximo una nota de una línea: "cada implementación puede ofrecer una fachada idiomática".

## Decisiones semánticas resueltas (confirmadas por Joan, 2026-08-03)

1. **Nombre canónico: conserva acentos y pasa a minúsculas.**
   `Año == año`, y ambos son **distintos** de `ano`/`ANO` (que son iguales entre sí).
   Es el modelo IDN de STXT-SPEC §4.3: trim → NFC → lowercase (case folding) →
   cada secuencia de separadores (`-`, `_`, espacios) pasa a ser un único `-` → sin `-`
   inicial ni final. Corregido en [core/string_utils.txt](core/string_utils.txt) (2026-08-03;
   antes describía NFKD + eliminación de diacríticos, lo cual era incorrecto).

2. **Hijos: la cardinalidad (mín/máx) siempre se valida; el orden de aparición
   NO se valida.** El enfoque de
   [schema/schema_validator.txt](schema/schema_validator.txt) (contando apariciones,
   independiente del orden) es correcto.

3. **Contrato de `SchemaProvider.getSchema`: devuelve NULL cuando el proveedor no tiene un esquema
   para ese espacio de nombres; los proveedores nunca lanzan "not found"** — el consumidor
   (`SchemaValidator`) informa `SCHEMA_NOT_FOUND`. Divergencia deliberada del
   blueprint respecto a js/java, donde los proveedores Meta lanzan `RESOURCE_NOT_FOUND` para
   otros espacios de nombres: alinearlos upstream cuando llegue el momento.

## Registro de trabajo (todo contra las especificaciones en `Last modif: 2026-08-02` y stxt-js 0.6.0)

1. ✅ **`normalizeChars`** corregido según la decisión semántica 1
   ([core/string_utils.txt](core/string_utils.txt); `cleanSpaces` eliminado — no
   quedan consumidores tras el cambio de tipos binarios).
2. ✅ **Regla de tabulaciones/espacios** ([core/line_indent.txt](core/line_indent.txt)): mezclar tabulaciones
   y espacios en la sangría de una misma línea → `MIXED_INDENTATION` (STXT-SPEC
   §8.1/§8.3). Detalles de la especificación: los comentarios y las líneas vacías nunca producen errores de sangría
   (§11); dentro de un bloque `>>` la comprobación se aplica al prefijo que cubre el nivel del bloque
   y las líneas vacías quedan exentas (§10.2 regla 1, §10.3).
3. ✅ **Tipos** ([schema/types.txt](schema/types.txt)): registrados los 18 tipos de la especificación.
   Se añadieron TIME, UUID, BINARY, MARKDOWN. Se encontraron correcciones de paridad al comparar:
   `RegexValue` (y ENUM, INLINE, URL) rechazan la forma en bloque con `NOT_ALLOWED_TEXT`
   mediante `isTextNode()`; BLOCK exige la forma en bloque con el código `BLOCK_FORM_REQUIRED`;
   los tipos binarios (HEXADECIMAL, BINARY, BASE64) validan sobre `binaryValue(node)`
   (§9.5: en forma de bloque, concatenación de líneas recortadas por línea — los espacios internos NO se ignoran); HEXADECIMAL ya no acepta el prefijo `#` ni exige longitud par.
   Añadido después: `TypeRegistry.admitsChildren()` (solo INLINE y GROUP).
4. ✅ **`ParseResult` / modo multi-error**: nuevo
   [core/parse_result.txt](core/parse_result.txt); [core/parser.txt](core/parser.txt)
   reescrito como una clase `Parser` con `parse()` (fail-fast, delega en
   `parseResult()` y lanza el primer error) y `parseResult()` (recoge errores de sintaxis
   y validación por línea con `TRY/CATCH`; los errores inesperados se recogen como
   `UNEXPECTED_ERROR` / `VALIDATION_ERROR`). Detalle de la especificación §10.3: el salto de línea final termina la última línea — no crea una línea vacía espuria en un bloque `>>` al final del archivo.
5. ✅ **Contratos `Observer` / `Validator`**: nuevos
   [processors/observer.txt](processors/observer.txt) (`onCreate`, `onFinish`,
   `onComment`, `onTextLine`) y [processors/validator.txt](processors/validator.txt)
   (`validate(node)` **devuelve** `ValidationException[]`, no lanza). Impulsado
   por el contrato y por la paridad con la especificación/js:
   - [core/line_indent.txt](core/line_indent.txt): `LineIndent` ahora lleva
     `is_comment`, `is_block` y `indent_length`; `parseLine` ya no devuelve NULL
     (el analizador decide usando las banderas — necesario para notificar comentarios y líneas de texto).
   - [core/node.txt](core/node.txt): validación de caracteres de nombre (`VALID_NAME`,
     STXT-SPEC §4.2/§11.8, letras y dígitos Unicode además de `-`, `_`, espacio);
     `getChild`/`getChildren` también filtran por espacio de nombres (por defecto, el del propio nodo);
     `AMBIGUOUS_CHILD` (antes `MORE_THAN_ONE_CHILD`); **`freeze()` eliminado** —
     ni js ni java congelan los nodos (inmutabilidad por convención: mutables mientras se analiza, de solo lectura después).
   - [schema/schema_validator.txt](schema/schema_validator.txt): reescrito al contrato de recopilación; añadido **`validateChildrenDeclared`** (modelo de contenido cerrado,
     STXT-SCHEMA-SPEC §6, código `CHILD_NOT_DECLARED` en la línea del hijo — antes faltaba); el exceso de máximos → error en el padre más uno en cada hijo infractor.
   - [core/platform.txt](core/platform.txt): añadido `splitLines`;
     [core/constants.txt](core/constants.txt): añadido `SEP_TEXT_NODE = ">>"`.
   - README.md: sección de excepciones (`THROW`, `TRY`/`CATCH` tipados) y métodos de array
     `pushAll`/`contains`.
6. ✅ **Proveedores de esquema/plantilla**:
   [schema/schema_provider.txt](schema/schema_provider.txt) y
   [template/template_schema_provider.txt](template/template_schema_provider.txt)
   reescritos. Contrato NULL (decisión semántica 3). Estructura alineada con js:
   **`SchemaProviderMemory`** (mapa + fallback a un proveedor padre, el meta por
   defecto; `addSchema(text)` analiza → valida frente al meta → registra) y
   **`TemplateSchemaProviderMemory`** (extiende Memory, `addTemplate`). **Eliminados**
   `SchemaProviderCache`, `SchemaProviderResources` y `ResourcesLoader` (patrón de stxt-java,
   ergonomía de plataforma, fuera del alcance normativo). Los marcadores `META_TEXT`
   ya no existen: el meta-esquema (§15.2, con los 18 valores de Type) y el meta-template (§16) están incrustados por completo.
   🐞 **Error detectado en stxt-js** al comparar: `SchemaProviderMemory.addSchema` y
   `TemplateSchemaProviderMemory.addTemplate` descartaban el array devuelto por
   `validator.validate(node)` → registraban silenciosamente definiciones inválidas (resto
   del cambio de contrato de Validator; `DiscoveryResolver.compile` sí lo comprueba). El
   blueprint lo describe correctamente (lanza el primer error si existe alguno).
   ✅ **Corregido en stxt-js el 2026-08-03** (commit `5847d99`, pruebas de regresión en
   `src/test/providers.test.ts`, suite a 256) — el primer error detectado por este blueprint.
   Sin publicar: npm 0.6.0 es anterior a la corrección.
7. ✅ **`discovery/`**: nuevo módulo completo según STXT-DISCOVERY-SPEC, imitando
   `src/discovery/` de stxt-js:
   [discovery_file_system.txt](discovery/discovery_file_system.txt) y
   [discovery_environment.txt](discovery/discovery_environment.txt) (abstracciones inyectadas; la distinción NULL frente a vacío de `STXT_PATH` es normativa §6),
   [discovery_error.txt](discovery/discovery_error.txt) (`DISCOVERY_*` errors
   recopilados, nunca lanzados, §8), [discovery_result.txt](discovery/discovery_result.txt)
   (niveles + definición activa por espacio de nombres con procedencia; implementa
   `SchemaProvider` y sirve los dos meta-esquemas) y
   [discovery_resolver.txt](discovery/discovery_resolver.txt) (§4/§6 chain con
   ascenso acotado `max_ascent=32`, deduplicación usuario/sistema, caché por nivel §7,
   archivos ordenados por ruta, duplicados al mismo nivel §8.1 sin elegir ganador — el
   espacio de nombres no tiene definición activa mientras dure el conflicto).
8. ✅ **`NodeWriter`** (2026-08-03): [core/node_writer.txt](core/node_writer.txt) —
   `toSTXT(node, style)` / `toSTXTDocs(docs, style)` con `INDENT_TABS` /
   `INDENT_SPACES_4`, garantía de round-trip indicada, espacio de nombres escrito solo donde no se hereda
   (js/java alojan esta clase en su paquete `runtime`).
9. ✅ **Revisión de traducción al inglés** (2026-08-03): todos los `.txt`, README.md y este
   CLAUDE.md están ahora en inglés. Se encontraron correcciones de paridad durante la revisión:
   - [core/name_namespace.txt](core/name_namespace.txt): el espacio de nombres dentro de `( )` NO se recorta (la gramática §7/§16 no permite espacios; `( com.example )`
     debe fallar en la validación de formato posterior).
   - [exceptions/exceptions.txt](exceptions/exceptions.txt) (nuevo): la jerarquía es
     exactamente `ParseException(line, code, message)`, `ValidationException EXTENDS
     ParseException` y `RuntimeException(code, message)`; `SchemaException` y
     `STXTException` ya no existen (modelo de js). `withLine()` se define para desplazar errores de bloques reanalizados.
   - [schema/schema.txt](schema/schema.txt) / [node_definition.txt](schema/node_definition.txt):
     soporte opcional de `description`; `addValue` ahora lanza `VALUE_DUPLICATED`
     (§13.9/§14.14 — los duplicados antes se ignoraban); los choques de definición son
     `ValidationException`.
   - [schema/schema_parser.txt](schema/schema_parser.txt): soporte de `Description` en schema/node; `CHILDREN_NOT_ALLOWED_FOR_TYPE` (§13.5) mediante
     `TypeRegistry.admitsChildren`; `MIN_GREATER_THAN_MAX` (§10/§13.7).
   - [template/child_line_parser.txt](template/child_line_parser.txt): las cardinalidades
     son enteros no negativos (`isNatural`); `MIN_GREATER_THAN_MAX` (§7.1); los corchetes vacíos `[]` cuentan como una definición explícita de valores vacíos (no NULL) — antes,
     una lista vacía se colapsaba a NULL; las excepciones son `ValidationException`.
   - [template/template_parser.txt](template/template_parser.txt): reescrito con paridad de js
     — los errores del bloque `Structure >>` reanalizado se relanzan con
     `withLine(+offset)` preservando el subtipo; los nodos de espacio de nombres externo pueden declarar
     solo cardinalidad (`TYPE_DEFINITION_NOT_ALLOWED`,
     `VALUES_NOT_ALLOWED_IN_EXTERNAL_NAMESPACE`,
     `CHILDREN_NOT_ALLOWED_IN_EXTERNAL_NAMESPACE`); reglas de referencia §14.11-14.13
     (`REFERENCE_NOT_FOUND`, `REFERENCE_WITH_TYPE_NOT_ALLOWED`,
     `NODE_REFERENCE_NOT_VALID`, `VALUES_NOT_ALLOWED_IN_REFERENCE`,
     `CHILDREN_NOT_ALLOWED_IN_REFERENCE`); `TYPE_NOT_VALID` y
     `CHILDREN_NOT_ALLOWED_FOR_TYPE` (§14.9); el bloque `Description >>` (§12) con
     `addDescriptions` y sus códigos de error.
   - [core/platform.txt](core/platform.txt): añadido `isNatural`; eliminado `isValidMinMax`
     (sin consumidores).

10. ✅ **Trazabilidad** (2026-08-03): nuevo [TRACEABILITY.md](TRACEABILITY.md) — tabla
    pseudocódigo archivo ↔ secciones de la especificación ↔ fuente de stxt-js ↔ clase de stxt-java, por paquete,
    además de la lista fuera de alcance y las divergencias deliberadas conocidas (contrato NULL del proveedor,
    modelo de 3 excepciones, el error addSchema/addTemplate detectado en stxt-js).
    Se verificaron todas las referencias § en los archivos `.txt` frente a los índices de sección de las
    cuatro especificaciones; se corrigió `admitsChildren` (TEMPLATE §8.2, no §15) y se añadieron las referencias faltantes a [core/constants.txt](core/constants.txt) y la nota sobre múltiples nodos raíz (§8.5) en [core/parser.txt](core/parser.txt).

## Pendiente

11. **Oficialización** (al final, una vez todo lo anterior esté bien): mencionar stxt-impl desde
    stxt-web y el CLAUDE.md de js/java; considerar adoptar la versionación compartida
    (js está en 0.6.0) para que "misma versión = mismo comportamiento" incluya también el pseudocódigo.

## Estructura actual

```
README.md                       Guía de estilo del pseudocódigo (§15: excepciones TRY/CATCH)
TRACEABILITY.md                 Mapeo archivo ↔ especificación ↔ stxt-js ↔ stxt-java + divergencias
core/
  constants.txt                 COMMENT_CHAR, TAB_SPACES=4, SEP_NODE, SEP_TEXT_NODE...
  line_indent.txt               parseLine: línea → LineIndent (banderas is_comment/is_block; MIXED_INDENTATION)
  name_namespace.txt            NameNamespaceParser: "Name (ns)" → (name, namespace)
  node.txt                      Clase Node (VALID_NAME §4.2; mutable mientras se analiza, luego de solo lectura)
  node_writer.txt               NodeWriter: toSTXT/toSTXTDocs, garantía de round-trip
  parse_result.txt              ParseResult: nodos raíz + errores recopilados
  parser.txt                    Clase Parser: parse()/parseResult(), hooks, pila de niveles
  platform.txt                  Funciones dependientes de la plataforma (declaradas, sin cuerpo)
  string_utils.txt              Utilidades de cadena (normalizeChars = nombre canónico §4.3)
  validations.txt               validateNamespaceFormat (§7)
exceptions/
  exceptions.txt                ParseException / ValidationException / RuntimeException
processors/
  observer.txt                  INTERFAZ Observer: onCreate/onFinish/onComment/onTextLine
  validator.txt                 INTERFAZ Validator: validate(node) → ValidationException[]
discovery/
  discovery_file_system.txt     INTERFAZ DiscoveryFileSystem + DiscoveryEntry (FS inyectado)
  discovery_environment.txt     INTERFAZ DiscoveryEnvironment: STXT_PATH, directorios usuario/sistema
  discovery_error.txt           DiscoveryError: códigos DISCOVERY_*, recopilados (§8)
  discovery_result.txt          DiscoveryDefinition/Level/Result (SchemaProvider + procedencia)
  discovery_resolver.txt        DiscoveryResolver: cadena, carga por nivel, precedencia
schema/
  schema.txt                    Schema: namespace + description + NodeDefinitions
  node_definition.txt           NodeDefinition: type, children, description, valores ENUM
  child_definition.txt          ChildDefinition: min/max cardinality
  schema_parser.txt             Documento @stxt.schema → Schema
  schema_provider.txt           SchemaProvider (NULL = no encontrado) + Memory + Meta (bootstrap §15.2)
  schema_validator.txt          Nodo frente a esquema (tipo + modelo cerrado + cardinalidades, sin orden)
  types.txt                     TypeRegistry (admitsChildren) + los 18 tipos de la especificación + binaryValue (§9.5)
template/
  template_parser.txt           Documento @stxt.template → Schema (referencias, bloque Description)
  child_line.txt                ChildLine: (cardinality) TYPE [values]
  child_line_parser.txt         Analizador de RuleSpec para líneas Structure >>
  template_schema_provider.txt  TemplateSchemaProviderMemory + meta-template (§16)
```

## Convenciones del repositorio

- Los archivos de pseudocódigo son `.txt`, siguiendo estrictamente la guía [README.md](README.md)
  (palabras clave en inglés en MAYÚSCULAS, variables snake_case, comentarios `#`).
- **Idiomas**: la conversación con Joan siempre es en español (de España), pero **todo
  el contenido del repositorio está en inglés**: README.md, comentarios `.txt`, identificadores, mensajes de error y este CLAUDE.md.
- Códigos de error en MAYÚSCULAS, idénticos a los de stxt-js/stxt-java.
- Commits: mensajes cortos; `M+` es la convención local para cambios menores. Joan hace
  todos los commits y pushes él mismo; nunca ejecutes `git commit`/`git push`; deja los cambios en el árbol de trabajo para su revisión.
