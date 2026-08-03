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
- `discovery/` — resolución de definiciones en filesystem (STXT-DISCOVERY-SPEC).
  ✅ Escrito (2026-08-03).
- **NodeWriter** (serializador) — el round-trip (escribir y reparsear produce el mismo árbol) es
  una propiedad semántica del lenguaje. **Pendiente de escribir.**
- **Interfaces `Observer` y `Validator`** con sus puntos de enganche al parser — definen *cuándo*
  se procesa/valida un nodo (validación en streaming al cerrar cada nodo), que es comportamiento
  observable. ✅ Escritas en `processors/` (2026-08-03).

**Fuera (libertad de diseño de cada port):**

- Fachadas de conveniencia (`STXT.parser()`...), `UnifiedSchemaProvider`, caches y cargadores de
  recursos (`SchemaProviderCache`/`ResourcesLoader`, patrón de stxt-java), adaptadores concretos
  de filesystem/entorno (Node `fs`, `vscode.workspace.fs`...).
- Como mucho, una nota de una línea: "cada implementación puede ofrecer una fachada idiomática".

## Decisiones semánticas resueltas (confirmadas por Joan, 2026-08-03)

1. **Nombre canónico: conserva acentos, pasa a minúsculas.**
   `Año == año`, y ambos son **distintos** de `ano`/`ANO` (que son iguales entre sí).
   Es el modelo IDN de STXT-SPEC §4.3: trim → NFC → minúsculas (case folding) → toda secuencia
   de separadores (`-`, `_`, espacios) pasa a un solo `-` → sin `-` en los extremos.
   ✅ Corregido en [core/string_utils.txt](core/string_utils.txt) (2026-08-03; antes describía
   NFKD + eliminación de diacríticos, que era incorrecto).

2. **Hijos: se valida siempre la cardinalidad (min/max); el orden de aparición NO se valida.**
   El enfoque actual de [schema/schema_validator.txt](schema/schema_validator.txt)
   (contar apariciones, independiente del orden) es **correcto**; no tocar.

## Estado actual y trabajo pendiente

Análisis hecho el 2026-08-03 comparando contra las specs (`Last modif: 2026-08-02`) y
stxt-js 0.6.0 (npm, 2026-08-02).

**Hecho (2026-08-03):**

1. ✅ **`normalizeChars`** corregido según la decisión semántica 1
   ([core/string_utils.txt](core/string_utils.txt), traducido a inglés; se eliminó `cleanSpaces`,
   que ya no tiene consumidores tras el cambio de los tipos binarios).
2. ✅ **Regla tab/espacio** ([core/line_indent.txt](core/line_indent.txt), traducido): mezclar tab
   y espacios en la indentación de una misma línea → `MIXED_INDENTATION` (STXT-SPEC §8.1/§8.3).
   Matices de la spec: comentarios y líneas vacías nunca dan error de indentación (§11); dentro de
   un bloque `>>` el chequeo aplica al prefijo que cubre el nivel de bloque y las líneas vacías
   están exentas (§10.2 regla 1, §10.3).
3. ✅ **Tipos** ([schema/types.txt](schema/types.txt), traducido): registrados los 18 de la spec.
   Añadidos TIME, UUID, BINARY, MARKDOWN. Además, correcciones de paridad con spec/js detectadas
   al comparar: `RegexValue` (y ENUM, INLINE, URL) ahora rechazan la forma bloque con
   `NOT_ALLOWED_TEXT` usando `isTextNode()`; BLOCK exige forma bloque con el código
   `BLOCK_FORM_REQUIRED`; los tipos binarios (HEXADECIMAL, BINARY, BASE64) validan sobre
   `binaryValue(node)` (§9.5: en bloque, concatenación de líneas con trim por línea — el
   whitespace interior NO se ignora); HEXADECIMAL ya no admite prefijo `#` ni exige longitud par.

4. ✅ **`ParseResult` / modo acumulador de errores** (2026-08-03): nuevo
   [core/parse_result.txt](core/parse_result.txt); [core/parser.txt](core/parser.txt) reescrito
   como clase `Parser` con `parse()` (fail-fast, delega en `parseResult()` y lanza el primer
   error) y `parseResult()` (acumula errores sintácticos y de validación por línea con
   `TRY/CATCH`; los errores inesperados se recogen como `UNEXPECTED_ERROR` /
   `VALIDATION_ERROR`). Detalle de la spec §10.3: el salto de línea final termina la última
   línea, no crea una línea vacía espuria en un bloque `>>` a fin de fichero.
5. ✅ **Contratos `Observer` / `Validator`** (2026-08-03): nuevos
   [processors/observer.txt](processors/observer.txt) (`onCreate`, `onFinish`, `onComment`,
   `onTextLine`) y [processors/validator.txt](processors/validator.txt) (`validate(node)`
   **devuelve** `ValidationException[]`, no lanza). Cambios arrastrados por el contrato y por
   paridad con spec/js:
   - [core/line_indent.txt](core/line_indent.txt): `LineIndent` ahora lleva `is_comment`,
     `is_block` e `indent_length`; `parseLine` ya no devuelve NULL (el parser decide con los
     flags, necesario para notificar comentarios y líneas de texto a los observers).
   - [core/node.txt](core/node.txt) [EN]: validación de caracteres del nombre (`VALID_NAME`,
     STXT-SPEC §4.2/§11.8, letras y dígitos Unicode + `-`, `_`, espacio); `getChild`/`getChildren`
     filtran también por namespace (por defecto el del propio nodo); `AMBIGUOUS_CHILD`
     (antes `MORE_THAN_ONE_CHILD`); **eliminado `freeze()`** — ni js ni java congelan nodos
     (inmutabilidad por convención: mutable durante el parseo, solo lectura después).
   - [schema/schema_validator.txt](schema/schema_validator.txt) [EN]: reescrito al contrato
     acumulador; añadido **`validateChildrenDeclared`** (modelo de contenido cerrado,
     STXT-SCHEMA-SPEC §6, código `CHILD_NOT_DECLARED` en la línea del hijo — antes faltaba);
     máximo excedido → error en el padre y además en cada hijo sobrante.
   - [core/platform.txt](core/platform.txt) [EN]: añadido `splitLines`; eliminados `freezeArray`
     e `isHexDigit` (sin consumidores). [core/constants.txt](core/constants.txt): añadido
     `SEP_TEXT_NODE = ">>"`.
   - README.md: nueva sección 15 (Excepciones: `THROW`, `TRY`/`CATCH` tipado) y `pushAll` en
     los métodos de array (secciones posteriores renumeradas).

6. ✅ **Providers de schema/template** (2026-08-03):
   [schema/schema_provider.txt](schema/schema_provider.txt) y
   [template/template_schema_provider.txt](template/template_schema_provider.txt) reescritos [EN].
   - **Contrato decidido**: `getSchema(namespace)` devuelve **NULL** si el provider no tiene
     schema para ese namespace; los providers NO lanzan "not found" — es el consumidor
     (`SchemaValidator`) quien emite `SCHEMA_NOT_FOUND`. Esto es más limpio que js, donde los
     providers Meta lanzan `RESOURCE_NOT_FOUND` para otros namespaces (con el efecto raro de que
     `SchemaProviderMemory`→Meta lanza en vez de devolver null). ⚠️ Divergencia deliberada del
     blueprint respecto a js/java: alinearlos upstream cuando toque.
   - Estructura alineada con js: **`SchemaProviderMemory`** (mapa + fallback a un provider padre,
     por defecto el meta; `addSchema(text)` parsea → valida contra el meta → registra) y
     **`TemplateSchemaProviderMemory`** (hereda de Memory, `addTemplate`). **Eliminados**
     `SchemaProviderCache`, `SchemaProviderResources` y `ResourcesLoader` (patrón de stxt-java,
     ergonomía de plataforma, fuera del alcance normativo).
   - Los `META_TEXT` ya no son placeholders: el meta-schema (§15.2, con los 18 valores de Type)
     y el meta-template (§16) están embebidos íntegros en comentario.
   - 🐞 **Bug detectado en stxt-js** al comparar: `SchemaProviderMemory.addSchema` y
     `TemplateSchemaProviderMemory.addTemplate` descartan el array devuelto por
     `validator.validate(node)` → registran definiciones inválidas en silencio (residuo del
     cambio de contrato de Validator; `DiscoveryResolver.compile` sí lo comprueba). El blueprint
     lo describe correcto (si hay errores, lanzar el primero). Avisado para corregir en stxt-js.
7. ✅ **`discovery/`** (2026-08-03): módulo nuevo completo según STXT-DISCOVERY-SPEC, espejo de
   `src/discovery/` de stxt-js [EN]: [discovery_file_system.txt](discovery/discovery_file_system.txt)
   y [discovery_environment.txt](discovery/discovery_environment.txt) (abstracciones inyectadas;
   la distinción NULL vs vacío de `STXT_PATH` es normativa §6),
   [discovery_error.txt](discovery/discovery_error.txt) (errores `DISCOVERY_*` coleccionados,
   nunca lanzados, §8), [discovery_result.txt](discovery/discovery_result.txt) (niveles +
   definición activa por namespace con procedencia; implementa `SchemaProvider` y sirve los dos
   meta-schemas) y [discovery_resolver.txt](discovery/discovery_resolver.txt) (cadena §4/§6 con
   ascenso limitado `max_ascent=32`, deduplicación usuario/sistema, caché por nivel §7, ficheros
   ordenados por path, duplicados por nivel §8.1 sin elegir ganador — el namespace queda sin
   definición activa mientras dure el conflicto).

**Pendiente, en orden recomendado:**

8. **`NodeWriter`**: serialización a STXT con estilo de indentación (tabs | 4 espacios) y garantía
   de round-trip.
9. **Traducción al inglés**: el repo debe quedar íntegramente en inglés — README.md (guía de
   estilo) y todos los comentarios de los ficheros `.txt` (hoy mezclan español e inglés).
   Estrategia: los ficheros que se toquen se traducen al revisarlos; al final, pasada de barrido
   sobre los que no se hayan tocado, README.md y este CLAUDE.md.
10. **Trazabilidad**: referencias a secciones de la spec en cada fichero (ya hay alguna, estilo
    "STXT-SCHEMA-SPEC §9.1") y una tabla fichero ↔ clase stxt-js ↔ clase stxt-java.
11. **Oficialización** (al final, cuando todo lo anterior esté bien): mencionar stxt-impl desde
    stxt-web y los CLAUDE.md de js/java; valorar adoptar el versionado común (js va por 0.6.0)
    para que "misma versión = mismo comportamiento" incluya el pseudocódigo.

## Estructura actual

```
README.md                       Guía de estilo del pseudolenguaje (§15: excepciones TRY/CATCH)
core/
  constants.txt                 COMMENT_CHAR, TAB_SPACES=4, SEP_NODE, SEP_TEXT_NODE... [EN]
  line_indent.txt               parseLine: línea → LineIndent (flags is_comment/is_block; MIXED_INDENTATION) [EN]
  name_namespace.txt            NameNamespaceParser: "Nombre (ns)" → (name, namespace)
  node.txt                      Clase Node (VALID_NAME §4.2; mutable al parsear, luego solo lectura) [EN]
  parse_result.txt              ParseResult: nodos raíz + errores acumulados [EN]
  parser.txt                    Clase Parser: parse()/parseResult(), hooks, pila por niveles [EN]
  platform.txt                  Funciones dependientes de plataforma (declaradas, sin cuerpo) [EN]
  string_utils.txt              Utilidades de cadena (normalizeChars = nombre canónico §4.3) [EN]
  validations.txt               validateNamespaceFormat
processors/
  observer.txt                  INTERFACE Observer: onCreate/onFinish/onComment/onTextLine [EN]
  validator.txt                 INTERFACE Validator: validate(node) → ValidationException[] [EN]
discovery/
  discovery_file_system.txt     INTERFACE DiscoveryFileSystem + DiscoveryEntry (FS inyectado) [EN]
  discovery_environment.txt     INTERFACE DiscoveryEnvironment: STXT_PATH, user/system dirs [EN]
  discovery_error.txt           DiscoveryError: códigos DISCOVERY_* coleccionados (§8) [EN]
  discovery_result.txt          DiscoveryDefinition/Level/Result (SchemaProvider + procedencia) [EN]
  discovery_resolver.txt        DiscoveryResolver: cadena, carga por niveles, precedencia [EN]
schema/
  schema.txt                    Schema: namespace + NodeDefinitions
  node_definition.txt           NodeDefinition: tipo, hijos, valores ENUM
  child_definition.txt          ChildDefinition: cardinalidad min/max
  schema_parser.txt             Documento @stxt.schema → Schema
  schema_provider.txt           SchemaProvider (NULL = not found) + Memory + Meta (bootstrap §15.2) [EN]
  schema_validator.txt          Valida nodo contra schema (tipo + modelo cerrado + cardinalidades, sin orden) [EN]
  types.txt                     TypeRegistry + los 18 tipos de la spec + binaryValue (§9.5) [EN]
template/
  template_parser.txt           Documento @stxt.template → Schema
  child_line.txt                ChildLine: (cardinalidad) TIPO [valores]
  child_line_parser.txt         Parser del RuleSpec de Structure >>
  template_schema_provider.txt  TemplateSchemaProviderMemory + meta-template (§16) [EN]
```

`[EN]` = fichero ya traducido íntegramente al inglés (punto 8 del trabajo pendiente).

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
