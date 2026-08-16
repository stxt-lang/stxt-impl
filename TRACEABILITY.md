# STXT implementation traceability

## Purpose and authority

This file is the maintenance map for the normative implementation blueprint.
The authority chain is:

```text
STXT specifications (stxt-web) -> this pseudocode (stxt-impl) -> language ports
```

The specifications decide the language. This repository turns their normative rules
into platform-neutral algorithms and data contracts. A port must be corrected when it
disagrees with either one; this document does not create independent language rules.

The canonical specifications are `../stxt-web/es/stxt-*-ref.stxt`. At the time of this
map (2026-08-16), STXT-SPEC has `Last modif: 2026-08-10`, STXT-TREE-SPEC has
`Last modif: 2026-08-09`; schema, template and discovery specifications have
`Last modif: 2026-08-02`. The ports this map refers to are `@stxt-lang/core` 0.7.0
(`../stxt-js`) and `dev.stxt:stxt-core` 0.7.0 (`../stxt-java`); both implement the five
specifications and share the 0.7.0 node model described in `core/node.txt`. Since 2026-08-16
there is a third port, `stxt` 0.7.0 for Python (`../stxt-python`), written directly from this
pseudocode: it mirrors it module by module (`core/node.txt` -> `stxt/core/node.py`,
`schema/types.txt` -> `stxt/schema/types.py`, `discovery/discovery_resolver.txt` ->
`stxt/discovery/discovery_resolver.py`, and so on, in `snake_case`), which is why the table
below does not need a column for it.

## Scope

Included here is language semantics: parsing, logical nodes and writing, exceptions,
parser hooks, schemas, templates, canonical tree JSON and discovery. Convenience
facades, concrete host adapters, caches and resource loaders are deliberately outside
the pseudocode's normative scope.

| Pseudocode area | Normative source | TypeScript reference | Java port |
|---|---|---|---|
| `core/constants.txt`, `platform.txt` | STXT-SPEC §§3, 5, 6, 8, 9 | `src/core/Constants.ts` | `dev.stxt.Constants` |
| `core/string_utils.txt`, `validations.txt` | STXT-SPEC §§4, 7, 10 | `src/core/StringUtils.ts`, `NamespaceValidator.ts` | `dev.stxt.utils.StringUtils`, `dev.stxt.NamespaceValidator` |
| `core/line_indent.txt` | STXT-SPEC §§8–10 | `src/core/Line.ts`, `LineParser.ts` | `dev.stxt.LineIndent`, `LineIndentParser` |
| `core/name_namespace.txt` | STXT-SPEC §§4.1, 7 | `src/core/NameNamespace.ts`, `NameNamespaceParser.ts` | `dev.stxt.NameNamespace`, `NameNamespaceParser` |
| `core/node.txt` (`Node`, `InlineNode`, `TextNode`) | STXT-SPEC §§4–7, 8.4; STXT-TREE-SPEC (two forms) | `src/core/Node.ts`, `InlineNode.ts`, `TextNode.ts`, `NodeCreator.ts` | `dev.stxt.Node`, `InlineNode`, `TextNode` |
| `core/parse_result.txt`, `parser.txt` | STXT-SPEC §§3–12 | `src/core/ParseResult.ts`, `Parser.ts` | `dev.stxt.ParseResult`, `Parser` |
| `core/node_writer.txt` | STXT-SPEC §§4–10; round-trip property | `src/runtime/NodeWriter.ts` | `dev.stxt.runtime.NodeWriter` |
| `core/tree_json.txt` | STXT-TREE-SPEC §§3–9 | `src/runtime/TreeJson.ts` | `dev.stxt.runtime.TreeJson` |
| `exceptions/exceptions.txt` | Stable error-code contract | `src/exceptions/*.ts` | `dev.stxt.exceptions.*` |
| `processors/observer.txt`, `validator.txt` | STXT-SPEC §§12, 17.3; schema/template §3 | `src/processors/*.ts` | `dev.stxt.processors.*` |
| `schema/child_definition.txt`, `node_definition.txt`, `schema.txt` | STXT-SCHEMA-SPEC §§4, 6–10 | `src/schema/ChildDefinition.ts`, `NodeDefinition.ts`, `Schema.ts` | `dev.stxt.schema.ChildDefinition`, `NodeDefinition`, `Schema` |
| `schema/schema_parser.txt`, `schema_provider.txt` | STXT-SCHEMA-SPEC §§4–8, 13, 15 | `src/schema/SchemaParser.ts`, `SchemaProvider*.ts` | `dev.stxt.schema.SchemaParser`, `SchemaProvider*` |
| `schema/schema_validator.txt`, `types.txt` | STXT-SCHEMA-SPEC §§6, 9–14 | `src/schema/SchemaValidator.ts`, `Type*.ts`, `type/*` | `dev.stxt.schema.SchemaValidator`, `Type*.java`, `type/*` |
| `template/child_line.txt`, `child_line_parser.txt` | STXT-TEMPLATE-SPEC §§6.2, 7, 9 | `src/template/ChildLine.ts`, `ChildLineParser.ts` | `dev.stxt.template.ChildLine`, `ChildLineParser` |
| `template/template_parser.txt`, `template_schema_provider.txt` | STXT-TEMPLATE-SPEC §§4–18 | `src/template/TemplateParser.ts`, `*TemplateSchemaProvider.ts` | `dev.stxt.template.TemplateParser`, `*TemplateSchemaProvider` |
| `discovery/discovery_environment.txt`, `discovery_file_system.txt` | STXT-DISCOVERY-SPEC §§4, 6 | `src/discovery/DiscoveryEnvironment.ts`, `DiscoveryFileSystem.ts` | `dev.stxt.discovery.DiscoveryEnvironment` (+ `SystemDiscoveryEnvironment`); no file-system abstraction — the port reads through `java.nio.file` directly (see below) |
| `discovery/discovery_error.txt`, `discovery_result.txt`, `discovery_resolver.txt` | STXT-DISCOVERY-SPEC §§3–10 | `src/discovery/DiscoveryError.ts`, `DiscoveryResult.ts`, `DiscoveryResolver.ts` | `dev.stxt.discovery.DiscoveryError`, `DiscoveryResult` (+ `DiscoveryDefinition`, `DiscoveryLevel`), `DiscoveryResolver` |

`src/runtime/ConditionalValidator.ts` and `UnifiedSchemaProvider.ts` in TypeScript,
and the Java `runtime.STXT` / resource-loader facades, are consumer conveniences.
They may be documented by ports but are not normative pseudocode modules.

## Conformance obligations by specification

### STXT-SPEC

The parser must accept UTF-8 with optional BOM, LF and CRLF; implement both node
forms; preserve sibling/root order; support multiple roots; calculate indentation by
level; and preserve the literal behaviour of `>>` blocks, including blank lines and
transparent comments. Names use NFC for identity, preserve accents, and namespaces
are ASCII and inherited only vertically. The authoritative algorithms are in
`core/parser.txt`, `line_indent.txt`, `name_namespace.txt`, `node.txt` and
`string_utils.txt`.

The in-memory tree (`core/node.txt`, since the 0.7.0 ports) has exactly two node forms —
`InlineNode` and `TextNode`, each owning only what is its own — linked to their parent,
with the level derived from the parent chain and the effective namespace resolved
through it from the namespace each node *declares*. This is the API contract shared by
the ports; the language and the canonical tree are unaffected by it.

### STXT-TREE-SPEC

`core/tree_json.txt` emits an outer array and exactly the form-specific members:
`name`, `canonicalName`, `namespace`, `form`, plus `value`/`children` for INLINE or
`lines` for BLOCK. Source metadata and derived properties are excluded.

### STXT-SCHEMA-SPEC

The schema modules implement the closed content model, declared node identities,
cardinality independent of order, type validation, cross-namespace references and
the meta-schema bootstrap. `SchemaProvider.getSchema()` returns `NULL` for absence;
the validator decides when to report it.

### STXT-TEMPLATE-SPEC

The template modules parse the `Structure >>` grammar, resolve local references and
open-ancestor recursion, apply template cardinalities and ENUM values, parse grouped
descriptions and compile the result to the equivalent `Schema` model.

### STXT-DISCOVERY-SPEC

The discovery modules build the complete ascending project/user/system chain or the
`STXT_PATH` replacement, load levels recursively, apply precedence by namespace and
collect resolution errors without throwing away unrelated definitions.

## Deliberate exclusions and known port status

| Item | Status |
|---|---|
| TypeScript discovery | Reference implementation since `@stxt-lang/core` 0.6.0. Host access (files, environment) is injected through `DiscoveryFileSystem` / `DiscoveryEnvironment`, because the same package runs in Node, VS Code and the browser. |
| Java discovery | Implemented since `stxt-core` 0.6.0 (`dev.stxt.discovery`); Java is conformant with STXT-DISCOVERY-SPEC. It injects only `DiscoveryEnvironment` and reads files directly through `java.nio.file` — a port-level choice, since Java has a universal file-system API; the algorithm (chain, per-namespace precedence, same-level conflicts, accumulated errors) is the one in `discovery_resolver.txt`. |
| Java name canonicalization | Aligned: NFC normalization preserves diacritics and treats decomposed and precomposed forms as equivalent. |
| Java exception hierarchy | Port-specific hierarchy; its stable error codes must still agree with this blueprint. |
| Node model (0.7.0) | Aligned on 2026-08-16 in both ports, Java first (the design came from production use of the Java API) and TypeScript as a replica. Port-level differences that are *not* semantic: TypeScript names the lookups `getChild(name, ns?)` / `getChildrenByName(name, ns?)` and inserts with `addChild(node, index?)` (no overloading); Java uses overloads (`getChildren(name[, ns])`, `addChild(index, node)`) and a `sealed` class; both keep `getNormalizedName()` as a deprecated alias of `getCanonicalName()` for one minor version. Covered by `node.test.ts` and `NodeTest`. |
| `SchemaProvider` not-found contract | Aligned on 2026-08-15 in both ports: providers return NULL, never throw. Before, the meta providers of both ports threw `RESOURCE_NOT_FOUND` and the Java cache a third code, `NOT_FOUND_SCHEMA`; through the default provider chains this made a validator throw instead of reporting `SCHEMA_NOT_FOUND`. Covered by `providers.test.ts` (TypeScript) and `SchemaProviderContractTest` (Java). |
| Caches, filesystem/environment adapters and public facades | Intentionally left to each port. |
| Python port | `../stxt-python` (`stxt` on PyPI), 2026-08-16: the five specifications and the 0.7.0 node model, module by module from this pseudocode. Port-level choices: `snake_case` names, positional overloads through `*args` plus keywords, read-only tuples for children and text lines, a closed hierarchy enforced in `Node.__init_subclass__`, all value types in one module, synchronous discovery with `OsDiscoveryFileSystem` / `SystemDiscoveryEnvironment` host adapters. Its `pytest` suite runs the shared `stxt-web` corpus (mandatory, fails at collection when absent) and covers every row of the regression table below. |

## Audit record — 2026-08-09

The spec-to-pseudocode review corrected four mismatches:

1. Node-name validation now happens after NFC, accepting decomposed Unicode spelling.
2. `Schema/Node` and `Child` values now use the full STXT node-name validation.
3. Template `Structure` lines now reject the core BLOCK (`>>`) form and require `:`.
4. A same-level discovery conflict now blocks fallback to a more distant definition.

The four corrections translate into the first five rows of the regression table below
(correction 2 yields two cases: `Node` and `Child`). The TypeScript port covers all five in
`core.test.ts`, `providers.test.ts` and `discovery.test.ts`. The Java port covers all five in
`NodeNameValidationTest`, `SchemaParserTest`, `TemplateParserTest` and
`DiscoveryResolverTest` (since `stxt-core` 0.6.0). The Python port covers them in
`test_core.py`, `test_providers.py` and `test_discovery.py`.

## Alignment record — 2026-08-15

The `SchemaProvider` not-found contract was aligned in both ports (see the status table
above): providers return NULL, never throw. It adds the "SchemaProvider contract" row of
the regression table.

## Alignment record — 2026-08-16 (node model, 0.7.0)

`core/node.txt` was rewritten for the new node model, and `core/parser.txt`,
`node_writer.txt`, `tree_json.txt`, `processors/observer.txt`, `schema/*` and `template/*`
adapted to it (walks ask for the form; the writer emits the namespace where declared; the
parser attaches a node to its parent when it opens it and hands it the declared, not the
resolved, namespace). This alignment went **against** the usual direction of the chain
on purpose: the design was made and tested in `stxt-java` first, replicated in `stxt-js`,
and then written here — it is API shape, not language semantics, and it came from real
use of the Java API. It adds the last three rows of the regression table.

## Required regression cases

These are behaviour-level cases, not a new fixture format. Every conforming port
should cover them through its normal test framework and, where appropriate, through
the shared `stxt-web` corpus. (Python: `test_core.py`, `test_providers.py`,
`test_template.py`, `test_discovery.py` and `test_node.py`.)

| Area | Input / setup | Expected result |
|---|---|---|
| Core name NFC | A node whose last character is `e` followed by a combining acute accent. | It parses successfully and its canonical name equals that of the equivalent precomposed `é` spelling. |
| Schema node name | A schema with `Node: Invalid!`. | The schema is rejected because the `Node` value is not a valid STXT node name. |
| Schema child name | A schema with `Child: Invalid!`. | The schema is rejected for the same reason. |
| Template Structure grammar | A `Structure >>` containing `Field >>`. | The template is rejected: every non-empty Structure line requires `:`. |
| Discovery conflict | The nearest level has two valid definitions for `com.example.docs`; a farther level has one valid definition for that namespace. | Discovery reports the nearest conflict and exposes no active definition for `com.example.docs`; it must not return the farther definition. |
| Node model: two forms | A tree with an inline node and a text node. | Only the inline node has a value, children and child lookups; only the text node has text lines; the base type exposes neither (a walk asks for the form). `getText()` is the value or the joined lines. |
| Node model: parent integrity | `a.addChild(child)`, then `b.addChild(child)`; and `grandchild.addChild(root)`. | The second add fails with `NODE_ALREADY_ATTACHED` and changes nothing; the third fails with `NODE_CYCLE`. `removeChild`/`detach` unlink both ends; `getLevel()` follows the chain of parents (root = 0). |
| Node model: declared vs effective namespace | A parsed `Doc (com.example.docs)` with a child that declares none and a grandchild declaring `org.other.ns`; then `doc.setNamespace("com.example.v2")`, and moving/detaching the child. | The child's effective namespace follows the parent (`com.example.docs`, then `com.example.v2`; `""` when detached, the new parent's when moved); the grandchild keeps `org.other.ns`. `NodeWriter` writes the namespace only where declared, and the canonical tree of the written text equals the original. |
| SchemaProvider contract | A `SchemaValidator` over the port's default provider chain (in-memory provider with the meta-schema provider as parent, or a resource-backed chain with a cache) validates a node whose namespace no provider knows. | `getSchema()` returns NULL at every level — meta providers, resource-backed providers, caches and chains included — and `validate()` returns exactly one finding, `SCHEMA_NOT_FOUND`, without throwing. No `RESOURCE_NOT_FOUND` or other not-found code surfaces from a provider. |
