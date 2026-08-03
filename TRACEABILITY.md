# Traceability

Mapping between this blueprint, the normative specification and the two existing
implementations. Use it to port STXT to a new platform (read the pseudocode, peek at
an implementation when in doubt) and to keep the three codebases in sync.

- **Spec** (`../stxt-web/es/`): `stxt-core-ref.stxt` (STXT-SPEC), `stxt-schema-ref.stxt`
  (STXT-SCHEMA-SPEC), `stxt-template-ref.stxt` (STXT-TEMPLATE-SPEC),
  `stxt-discovery-ref.stxt` (STXT-DISCOVERY-SPEC).
- **stxt-js** (`../stxt-js/src/`): npm `@stxt-lang/core` (0.6.0 at the time of writing).
- **stxt-java** (`../stxt-java/src/main/java/dev/stxt/`): Maven `dev.stxt:stxt-core`
  (0.5.3 at the time of writing).

## Core

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [core/constants.txt](core/constants.txt) | STXT-SPEC §3, §5, §6, §8, §9 | `core/Constants.ts` | `Constants` |
| [core/line_indent.txt](core/line_indent.txt) | STXT-SPEC §8, §10.2, §10.3, §11 | `core/Line.ts`, `core/LineParser.ts` | `LineIndent`, `LineIndentParser` |
| [core/name_namespace.txt](core/name_namespace.txt) | STXT-SPEC §4, §7 | `core/NameNamespace.ts`, `core/NameNamespaceParser.ts` | `NameNamespace`, `NameNamespaceParser` |
| [core/validations.txt](core/validations.txt) | STXT-SPEC §7 | `core/NamespaceValidator.ts` | `NamespaceValidator` |
| [core/node.txt](core/node.txt) | STXT-SPEC §4, §5, §6, §11 | `core/Node.ts` | `Node` |
| [core/parser.txt](core/parser.txt) | STXT-SPEC §8, §10, §11 | `core/Parser.ts`, `core/NodeCreator.ts` | `Parser` |
| [core/parse_result.txt](core/parse_result.txt) | — (API) | `core/ParseResult.ts` | `ParseResult` |
| [core/node_writer.txt](core/node_writer.txt) | — (round-trip over §4-§10) | `runtime/NodeWriter.ts` | `runtime.NodeWriter` |
| [core/string_utils.txt](core/string_utils.txt) | STXT-SPEC §4.3, §10 | `core/StringUtils.ts` | `utils.StringUtils` |
| [core/platform.txt](core/platform.txt) | — (platform built-ins) | language built-ins | language built-ins, `utils.FileUtils` |

## Exceptions

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [exceptions/exceptions.txt](exceptions/exceptions.txt) | error codes throughout §11 / §13 / §14 | `exceptions/ParseException.ts`, `exceptions/ValidationException.ts`, `exceptions/RuntimeException.ts` | `exceptions.ParseException`, `exceptions.ValidationException` (+ platform extras: `STXTException`, `SchemaException`, `ResourceNotFoundException`, `STXTIOException`) |

## Processors

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [processors/observer.txt](processors/observer.txt) | — (API; streaming over §8.5) | `processors/Observer.ts` | `processors.Observer` |
| [processors/validator.txt](processors/validator.txt) | — (API; validation on node close) | `processors/Validator.ts` | `processors.Validator` |

## Schema

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [schema/schema.txt](schema/schema.txt) | STXT-SCHEMA-SPEC §4, §5 | `schema/Schema.ts` | `schema.Schema` |
| [schema/node_definition.txt](schema/node_definition.txt) | STXT-SCHEMA-SPEC §7 | `schema/NodeDefinition.ts` | `schema.NodeDefinition` |
| [schema/child_definition.txt](schema/child_definition.txt) | STXT-SCHEMA-SPEC §8, §10 | `schema/ChildDefinition.ts` | `schema.ChildDefinition` |
| [schema/schema_parser.txt](schema/schema_parser.txt) | STXT-SCHEMA-SPEC §4-§10, §13 | `schema/SchemaParser.ts` | `schema.SchemaParser` |
| [schema/schema_validator.txt](schema/schema_validator.txt) | STXT-SCHEMA-SPEC §6, §9-§11, §13 | `schema/SchemaValidator.ts` | `schema.SchemaValidator` |
| [schema/schema_provider.txt](schema/schema_provider.txt) | STXT-SCHEMA-SPEC §15 (meta-schema) | `schema/SchemaProvider.ts`, `schema/SchemaProviderMemory.ts`, `schema/SchemaProviderMeta.ts` | `schema.SchemaProvider`, `schema.SchemaProviderMeta` (+ out-of-scope: `SchemaProviderCache`, `SchemaProviderResources`, `resources.ResourcesLoader*`) |
| [schema/types.txt](schema/types.txt) | STXT-SCHEMA-SPEC §9 | `schema/Type.ts`, `schema/TypeRegistry.ts`, `schema/type/*.ts` (incl. `regexType.ts`, `binaryValue.ts`) | `schema.Type`, `schema.TypeRegistry`, `schema.type.*` (incl. `RegexValue`, `BinaryValue`) |

## Template

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [template/child_line.txt](template/child_line.txt) | STXT-TEMPLATE-SPEC §6.2, §7 | `template/ChildLine.ts` | `template.ChildLine` |
| [template/child_line_parser.txt](template/child_line_parser.txt) | STXT-TEMPLATE-SPEC §6.2, §7, §9 | `template/ChildLineParser.ts` | `template.ChildLineParser` |
| [template/template_parser.txt](template/template_parser.txt) | STXT-TEMPLATE-SPEC §6, §8-§14 | `template/TemplateParser.ts` | `template.TemplateParser` |
| [template/template_schema_provider.txt](template/template_schema_provider.txt) | STXT-TEMPLATE-SPEC §16 (meta-template) | `template/TemplateSchemaProviderMemory.ts`, `template/MetaTemplateSchemaProvider.ts` | `template.TemplateSchemaProvider`, `template.MetaTemplateSchemaProvider` |

## Discovery

| stxt-impl | Spec | stxt-js | stxt-java |
|---|---|---|---|
| [discovery/discovery_file_system.txt](discovery/discovery_file_system.txt) | STXT-DISCOVERY-SPEC §3 | `discovery/DiscoveryFileSystem.ts` | *not ported yet* |
| [discovery/discovery_environment.txt](discovery/discovery_environment.txt) | STXT-DISCOVERY-SPEC §4.2, §6 | `discovery/DiscoveryEnvironment.ts` | *not ported yet* |
| [discovery/discovery_error.txt](discovery/discovery_error.txt) | STXT-DISCOVERY-SPEC §8 | `discovery/DiscoveryError.ts` | *not ported yet* |
| [discovery/discovery_result.txt](discovery/discovery_result.txt) | STXT-DISCOVERY-SPEC §5, §7 | `discovery/DiscoveryResult.ts` | *not ported yet* |
| [discovery/discovery_resolver.txt](discovery/discovery_resolver.txt) | STXT-DISCOVERY-SPEC §4-§8 | `discovery/DiscoveryResolver.ts` | *not ported yet* |

## Out of the blueprint's scope

Platform ergonomics each implementation shapes freely (see the scope section of
[CLAUDE.md](CLAUDE.md)): stxt-js `runtime/STXT... UnifiedSchemaProvider.ts`,
`runtime/ConditionalValidator.ts`; stxt-java `runtime.STXT`, `schema.SchemaProviderCache`,
`schema.SchemaProviderResources`, `resources.*`, `utils.FileUtils`.

## Known deliberate divergences (blueprint ahead of the ports)

Decisions taken here that js/java should adopt (see "Resolved semantic decisions" in
[CLAUDE.md](CLAUDE.md)):

1. `SchemaProvider.getSchema` returns **NULL** for an unknown namespace; the Meta
   providers of js/java still throw `RESOURCE_NOT_FOUND` there.
2. The exception model is exactly three types (`ParseException`, `ValidationException`,
   `RuntimeException`); stxt-java still carries `SchemaException` and others.
3. ~~`SchemaProviderMemory.addSchema` / `TemplateSchemaProviderMemory.addTemplate` MUST
   fail when the definition does not validate against its meta-schema; stxt-js 0.6.0
   discarded those errors.~~ **Resolved**: fixed in stxt-js on 2026-08-03 (commit
   `5847d99`, tests in `src/test/providers.test.ts`); npm releases after 0.6.0 carry
   it. stxt-java should be checked for the same pattern when it adopts the collecting
   `Validator` contract.
