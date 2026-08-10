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
map, STXT-SPEC has `Last modif: 2026-08-10`, STXT-TREE-SPEC has `Last modif: 2026-08-09`;
schema, template and discovery specifications have `Last modif: 2026-08-02`.

## Scope

Included here is language semantics: parsing, logical nodes and writing, exceptions,
parser hooks, schemas, templates, canonical tree JSON and discovery. Convenience
facades, concrete host adapters, caches and resource loaders are deliberately outside
the pseudocode's normative scope.

| Pseudocode area | Normative source | TypeScript reference | Java port |
|---|---|---|---|
| `core/constants.txt`, `platform.txt` | STXT-SPEC §§3, 5, 6, 8, 9 | `src/core/Constants.ts` | `dev.stxt.Constants` |
| `core/string_utils.txt`, `validations.txt` | STXT-SPEC §§4, 7, 10 | `src/core/StringUtils.ts`, `NamespaceValidator.ts` | `dev.stxt.utils.StringUtils`, `NamespaceValidator` |
| `core/line_indent.txt` | STXT-SPEC §§8–10 | `src/core/Line.ts`, `LineParser.ts` | `dev.stxt.LineIndent`, `LineIndentParser` |
| `core/name_namespace.txt` | STXT-SPEC §§4.1, 7 | `src/core/NameNamespace.ts`, `NameNamespaceParser.ts` | `dev.stxt.NameNamespace`, `NameNamespaceParser` |
| `core/node.txt` | STXT-SPEC §§4–7, 8.4 | `src/core/Node.ts`, `NodeCreator.ts` | `dev.stxt.Node` |
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
| `discovery/discovery_environment.txt`, `discovery_file_system.txt` | STXT-DISCOVERY-SPEC §§4, 6 | `src/discovery/DiscoveryEnvironment.ts`, `DiscoveryFileSystem.ts` | Not implemented |
| `discovery/discovery_error.txt`, `discovery_result.txt`, `discovery_resolver.txt` | STXT-DISCOVERY-SPEC §§3–10 | `src/discovery/DiscoveryError.ts`, `DiscoveryResult.ts`, `DiscoveryResolver.ts` | Not implemented |

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
| TypeScript discovery | Reference implementation since 0.6.0. |
| Java discovery | Not implemented yet; Java is therefore not yet conformant with STXT-DISCOVERY-SPEC. |
| Java name canonicalization | Aligned: NFC normalization preserves diacritics and treats decomposed and precomposed forms as equivalent. |
| Java exception hierarchy | Port-specific hierarchy; its stable error codes must still agree with this blueprint. |
| Caches, filesystem/environment adapters and public facades | Intentionally left to each port. |

## Audit record — 2026-08-09

The spec-to-pseudocode review corrected four mismatches:

1. Node-name validation now happens after NFC, accepting decomposed Unicode spelling.
2. `Schema/Node` and `Child` values now use the full STXT node-name validation.
3. Template `Structure` lines now reject the core BLOCK (`>>`) form and require `:`.
4. A same-level discovery conflict now blocks fallback to a more distant definition.

The TypeScript port covers all five cases in `core.test.ts`, `providers.test.ts` and
`discovery.test.ts`. The Java port covers the first four in `NodeNameValidationTest`,
`SchemaParserTest` and `TemplateParserTest`; discovery remains outside its implemented scope.

## Required regression cases

These are behaviour-level cases, not a new fixture format. Every conforming port
should cover them through its normal test framework and, where appropriate, through
the shared `stxt-web` corpus.

| Area | Input / setup | Expected result |
|---|---|---|
| Core name NFC | A node whose last character is `e` followed by a combining acute accent. | It parses successfully and its canonical name equals that of the equivalent precomposed `é` spelling. |
| Schema node name | A schema with `Node: Invalid!`. | The schema is rejected because the `Node` value is not a valid STXT node name. |
| Schema child name | A schema with `Child: Invalid!`. | The schema is rejected for the same reason. |
| Template Structure grammar | A `Structure >>` containing `Field >>`. | The template is rejected: every non-empty Structure line requires `:`. |
| Discovery conflict | The nearest level has two valid definitions for `com.example.docs`; a farther level has one valid definition for that namespace. | Discovery reports the nearest conflict and exposes no active definition for `com.example.docs`; it must not return the farther definition. |
