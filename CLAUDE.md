# CLAUDE.md — stxt-impl

## What this repository is

A **language-neutral pseudocode** description of the STXT (Semantic Text)
implementation. It is the blueprint for porting STXT to new platforms: an implementer
must be able to write a conformant port reading only this repository and the
specification, without reading TypeScript or Java.

The pseudo-language style guide lives in [README.md](README.md) (keywords, types,
naming conventions). Every `.txt` file in this repo follows it.

## The STXT ecosystem

Sibling repos under `C:\desarrollo\workspace_java` (GitHub org `stxt-lang`, web https://stxt.dev):

| Repo | Role |
|---|---|
| `../stxt-web` | **Normative specification**, written in STXT itself. Canonical in `es/`: `stxt-core-ref.stxt`, `stxt-schema-ref.stxt`, `stxt-template-ref.stxt`, `stxt-discovery-ref.stxt`. English mirror in `en/`. No version numbers: tracked via `Last modif`. Also holds the example corpus used by the js and java conformance suites. |
| `../stxt-js` | Official implementation. npm `@stxt-lang/core` (TypeScript). Its `CLAUDE.md` documents the architecture. |
| `../stxt-java` | Java port with deliberate parity with js (same error codes, aligned versions). Maven Central `dev.stxt:stxt-core`. |
| `stxt-impl` (this one) | Language-neutral pseudocode of the implementation. |
| Others | `../stxt-vscode` (extension), `stxt-cli`, `../stxt-python` (skeleton, first candidate to use this blueprint), `../stxt-cms`. |

**Authority order for ambiguities (target):**

```
spec (stxt-web)  →  stxt-impl (this repo)  →  stxt-js  →  stxt-java
```

The current order in force is spec → js → java; this repo will enter second position
**once it is complete and correct**. At that point the CLAUDE.md of the other repos and
the spec will be updated to reflect it. Until then, do not announce that status
anywhere else.

## Normative scope (decision closed, 2026-08-03)

The boundary is **"language semantics"**, not "whatever stxt-js does".

**In (this repo describes it and it is normative):**

- `core/` — syntactic parser, nodes, indentation, names and namespaces, `NodeWriter`.
- `exceptions/` — the exception hierarchy and its error-code discipline.
- `processors/` — the `Observer` and `Validator` contracts and their hook points into
  the parser (they define *when* a node is processed/validated — streaming validation
  when each node closes — which is observable behavior).
- `schema/` — semantic validation, types, schema providers.
- `template/` — compilation of `@stxt.template` into a schema.
- `discovery/` — definition resolution on the filesystem (STXT-DISCOVERY-SPEC).
- **NodeWriter** — round-trip (write + re-parse yields the same tree) is a semantic
  property of the language.

**Out (design freedom of each port):**

- Convenience facades (`STXT.parser()`...), `UnifiedSchemaProvider`, caches and
  resource loaders (`SchemaProviderCache`/`ResourcesLoader`, the stxt-java pattern),
  concrete filesystem/environment adapters (Node `fs`, `vscode.workspace.fs`...).
- At most a one-line note: "each implementation may offer an idiomatic facade".

## Resolved semantic decisions (confirmed by Joan, 2026-08-03)

1. **Canonical name: keeps accents, lower-cases.**
   `Año == año`, and both are **different** from `ano`/`ANO` (which equal each other).
   It is the IDN model of STXT-SPEC §4.3: trim → NFC → lowercase (case folding) →
   every run of separators (`-`, `_`, spaces) becomes a single `-` → no leading or
   trailing `-`. Fixed in [core/string_utils.txt](core/string_utils.txt) (2026-08-03;
   it previously described NFKD + diacritic stripping, which was wrong).

2. **Children: cardinality (min/max) is always validated; the order of appearance is
   NOT validated.** The approach of
   [schema/schema_validator.txt](schema/schema_validator.txt) (counting occurrences,
   order independent) is correct.

3. **`SchemaProvider.getSchema` contract: returns NULL when the provider has no schema
   for that namespace; providers never throw "not found"** — the consumer
   (`SchemaValidator`) reports `SCHEMA_NOT_FOUND`. Deliberate divergence of the
   blueprint from js/java, where the Meta providers throw `RESOURCE_NOT_FOUND` for
   other namespaces: align them upstream when the time comes.

## Work log (all against the specs at `Last modif: 2026-08-02` and stxt-js 0.6.0)

1. ✅ **`normalizeChars`** fixed per semantic decision 1
   ([core/string_utils.txt](core/string_utils.txt); `cleanSpaces` removed — no
   consumers left after the binary types change).
2. ✅ **Tab/space rule** ([core/line_indent.txt](core/line_indent.txt)): mixing tabs
   and spaces in the indentation of a single line → `MIXED_INDENTATION` (STXT-SPEC
   §8.1/§8.3). Spec nuances: comments and empty lines never produce indentation errors
   (§11); inside a `>>` block the check applies to the prefix covering the block level
   and empty lines are exempt (§10.2 rule 1, §10.3).
3. ✅ **Types** ([schema/types.txt](schema/types.txt)): all 18 spec types registered.
   Added TIME, UUID, BINARY, MARKDOWN. Parity fixes found while comparing:
   `RegexValue` (and ENUM, INLINE, URL) reject the block form with `NOT_ALLOWED_TEXT`
   via `isTextNode()`; BLOCK requires the block form with code `BLOCK_FORM_REQUIRED`;
   the binary types (HEXADECIMAL, BINARY, BASE64) validate over `binaryValue(node)`
   (§9.5: in block form, concatenation of per-line-trimmed lines — inner whitespace is
   NOT ignored); HEXADECIMAL no longer takes a `#` prefix nor requires even length.
   Later addition: `TypeRegistry.admitsChildren()` (only INLINE and GROUP).
4. ✅ **`ParseResult` / multi-error mode**: new
   [core/parse_result.txt](core/parse_result.txt); [core/parser.txt](core/parser.txt)
   rewritten as a `Parser` class with `parse()` (fail-fast, delegates to
   `parseResult()` and throws the first error) and `parseResult()` (collects syntax
   and validation errors per line with `TRY/CATCH`; unexpected errors are collected as
   `UNEXPECTED_ERROR` / `VALIDATION_ERROR`). Spec detail §10.3: the final line break
   terminates the last line — it does not create a spurious empty line in a `>>` block
   at EOF.
5. ✅ **`Observer` / `Validator` contracts**: new
   [processors/observer.txt](processors/observer.txt) (`onCreate`, `onFinish`,
   `onComment`, `onTextLine`) and [processors/validator.txt](processors/validator.txt)
   (`validate(node)` **returns** `ValidationException[]`, it does not throw). Dragged
   by the contract and by spec/js parity:
   - [core/line_indent.txt](core/line_indent.txt): `LineIndent` now carries
     `is_comment`, `is_block` and `indent_length`; `parseLine` no longer returns NULL
     (the parser decides using the flags — needed to notify comments and text lines).
   - [core/node.txt](core/node.txt): name character validation (`VALID_NAME`,
     STXT-SPEC §4.2/§11.8, Unicode letters and digits plus `-`, `_`, space);
     `getChild`/`getChildren` also filter by namespace (defaulting to the node's own);
     `AMBIGUOUS_CHILD` (formerly `MORE_THAN_ONE_CHILD`); **`freeze()` removed** —
     neither js nor java freeze nodes (immutability by convention: mutable while
     parsing, read-only afterwards).
   - [schema/schema_validator.txt](schema/schema_validator.txt): rewritten to the
     collecting contract; added **`validateChildrenDeclared`** (closed content model,
     STXT-SCHEMA-SPEC §6, code `CHILD_NOT_DECLARED` on the child's line — previously
     missing); max exceeded → error on the parent plus one on each offending child.
   - [core/platform.txt](core/platform.txt): added `splitLines`;
     [core/constants.txt](core/constants.txt): added `SEP_TEXT_NODE = ">>"`.
   - README.md: exceptions section (`THROW`, typed `TRY`/`CATCH`) and array methods
     `pushAll`/`contains`.
6. ✅ **Schema/template providers**:
   [schema/schema_provider.txt](schema/schema_provider.txt) and
   [template/template_schema_provider.txt](template/template_schema_provider.txt)
   rewritten. NULL contract (semantic decision 3). Structure aligned with js:
   **`SchemaProviderMemory`** (map + fallback to a parent provider, the meta by
   default; `addSchema(text)` parses → validates against the meta → registers) and
   **`TemplateSchemaProviderMemory`** (extends Memory, `addTemplate`). **Removed**
   `SchemaProviderCache`, `SchemaProviderResources` and `ResourcesLoader` (stxt-java
   pattern, platform ergonomics, outside the normative scope). The `META_TEXT`
   placeholders are gone: the meta-schema (§15.2, with the 18 Type values) and the
   meta-template (§16) are embedded in full.
   🐞 **Bug spotted in stxt-js** while comparing: `SchemaProviderMemory.addSchema` and
   `TemplateSchemaProviderMemory.addTemplate` discarded the array returned by
   `validator.validate(node)` → they silently registered invalid definitions (leftover
   of the Validator contract change; `DiscoveryResolver.compile` does check it). The
   blueprint describes it correctly (throw the first error if any).
   ✅ **Fixed in stxt-js on 2026-08-03** (commit `5847d99`, regression tests in
   `src/test/providers.test.ts`, suite at 256) — first bug caught by this blueprint.
   Unreleased: npm 0.6.0 predates the fix.
7. ✅ **`discovery/`**: complete new module per STXT-DISCOVERY-SPEC, mirroring
   `src/discovery/` of stxt-js:
   [discovery_file_system.txt](discovery/discovery_file_system.txt) and
   [discovery_environment.txt](discovery/discovery_environment.txt) (injected
   abstractions; the NULL vs empty distinction of `STXT_PATH` is normative §6),
   [discovery_error.txt](discovery/discovery_error.txt) (`DISCOVERY_*` errors
   collected, never thrown, §8), [discovery_result.txt](discovery/discovery_result.txt)
   (levels + active definition per namespace with provenance; implements
   `SchemaProvider` and serves the two meta-schemas) and
   [discovery_resolver.txt](discovery/discovery_resolver.txt) (§4/§6 chain with
   bounded ascent `max_ascent=32`, user/system deduplication, per-level cache §7,
   files sorted by path, same-level duplicates §8.1 without picking a winner — the
   namespace has no active definition while the conflict lasts).
8. ✅ **`NodeWriter`** (2026-08-03): [core/node_writer.txt](core/node_writer.txt) —
   `toSTXT(node, style)` / `toSTXTDocs(docs, style)` with `INDENT_TABS` /
   `INDENT_SPACES_4`, round-trip guarantee stated, namespace written only where it is
   not inherited (js/java host this class in their `runtime` package).
9. ✅ **English translation sweep** (2026-08-03): every `.txt`, README.md and this
   CLAUDE.md are now in English. Parity fixes found during the sweep:
   - [core/name_namespace.txt](core/name_namespace.txt): the namespace inside `( )` is
     NOT trimmed (the grammar §7/§16 does not allow spaces there; `( com.example )`
     must fail the later format validation).
   - [exceptions/exceptions.txt](exceptions/exceptions.txt) (new): the hierarchy is
     exactly `ParseException(line, code, message)`, `ValidationException EXTENDS
     ParseException` and `RuntimeException(code, message)`; `SchemaException` and
     `STXTException` no longer exist (js model). `withLine()` defined for shifting
     re-parsed-block errors.
   - [schema/schema.txt](schema/schema.txt) / [node_definition.txt](schema/node_definition.txt):
     optional `description` support; `addValue` now throws `VALUE_DUPLICATED`
     (§13.9/§14.14 — duplicates were silently ignored before); definition clashes are
     `ValidationException`.
   - [schema/schema_parser.txt](schema/schema_parser.txt): schema/node `Description`
     support; `CHILDREN_NOT_ALLOWED_FOR_TYPE` (§13.5) via
     `TypeRegistry.admitsChildren`; `MIN_GREATER_THAN_MAX` (§10/§13.7).
   - [template/child_line_parser.txt](template/child_line_parser.txt): cardinalities
     are non-negative integers (`isNatural`); `MIN_GREATER_THAN_MAX` (§7.1); empty
     brackets `[]` count as an EXPLICIT empty values definition (non-NULL) — before,
     an empty list collapsed to NULL; exceptions are `ValidationException`.
   - [template/template_parser.txt](template/template_parser.txt): rewritten to js
     parity — errors of the re-parsed `Structure >>` block are re-thrown with
     `withLine(+offset)` preserving the subtype; external-namespace nodes may declare
     cardinality only (`TYPE_DEFINITION_NOT_ALLOWED`,
     `VALUES_NOT_ALLOWED_IN_EXTERNAL_NAMESPACE`,
     `CHILDREN_NOT_ALLOWED_IN_EXTERNAL_NAMESPACE`); reference rules §14.11-14.13
     (`REFERENCE_NOT_FOUND`, `REFERENCE_WITH_TYPE_NOT_ALLOWED`,
     `NODE_REFERENCE_NOT_VALID`, `VALUES_NOT_ALLOWED_IN_REFERENCE`,
     `CHILDREN_NOT_ALLOWED_IN_REFERENCE`); `TYPE_NOT_VALID` and
     `CHILDREN_NOT_ALLOWED_FOR_TYPE` (§14.9); the `Description >>` block (§12) with
     `addDescriptions` and its error codes.
   - [core/platform.txt](core/platform.txt): `isNatural` added; `isValidMinMax`
     removed (no consumers).

10. ✅ **Traceability** (2026-08-03): new [TRACEABILITY.md](TRACEABILITY.md) — table
    pseudocode file ↔ spec sections ↔ stxt-js source ↔ stxt-java class, per package,
    plus the out-of-scope list and the known deliberate divergences (NULL provider
    contract, 3-exception model, the addSchema/addTemplate bug flagged in stxt-js).
    Every § reference in the `.txt` files verified against the section indexes of the
    four specs; fixed `admitsChildren` (TEMPLATE §8.2, not §15) and added the missing
    references to [core/constants.txt](core/constants.txt) and the multiple-root-nodes
    note (§8.5) to [core/parser.txt](core/parser.txt).

## Pending

11. **Officialization** (last, once everything above is right): mention stxt-impl from
    stxt-web and the CLAUDE.md of js/java; consider adopting the shared versioning
    (js is at 0.6.0) so that "same version = same behavior" includes the pseudocode.

## Current structure

```
README.md                       Pseudocode style guide (§15: TRY/CATCH exceptions)
TRACEABILITY.md                 File ↔ spec ↔ stxt-js ↔ stxt-java mapping + divergences
core/
  constants.txt                 COMMENT_CHAR, TAB_SPACES=4, SEP_NODE, SEP_TEXT_NODE...
  line_indent.txt               parseLine: line → LineIndent (is_comment/is_block flags; MIXED_INDENTATION)
  name_namespace.txt            NameNamespaceParser: "Name (ns)" → (name, namespace)
  node.txt                      Node class (VALID_NAME §4.2; mutable while parsing, then read-only)
  node_writer.txt               NodeWriter: toSTXT/toSTXTDocs, round-trip guarantee
  parse_result.txt              ParseResult: root nodes + collected errors
  parser.txt                    Parser class: parse()/parseResult(), hooks, level stack
  platform.txt                  Platform-dependent functions (declared, no body)
  string_utils.txt              String utilities (normalizeChars = canonical name §4.3)
  validations.txt               validateNamespaceFormat (§7)
exceptions/
  exceptions.txt                ParseException / ValidationException / RuntimeException
processors/
  observer.txt                  INTERFACE Observer: onCreate/onFinish/onComment/onTextLine
  validator.txt                 INTERFACE Validator: validate(node) → ValidationException[]
discovery/
  discovery_file_system.txt     INTERFACE DiscoveryFileSystem + DiscoveryEntry (injected FS)
  discovery_environment.txt     INTERFACE DiscoveryEnvironment: STXT_PATH, user/system dirs
  discovery_error.txt           DiscoveryError: DISCOVERY_* codes, collected (§8)
  discovery_result.txt          DiscoveryDefinition/Level/Result (SchemaProvider + provenance)
  discovery_resolver.txt        DiscoveryResolver: chain, per-level loading, precedence
schema/
  schema.txt                    Schema: namespace + description + NodeDefinitions
  node_definition.txt           NodeDefinition: type, children, description, ENUM values
  child_definition.txt          ChildDefinition: min/max cardinality
  schema_parser.txt             @stxt.schema document → Schema
  schema_provider.txt           SchemaProvider (NULL = not found) + Memory + Meta (bootstrap §15.2)
  schema_validator.txt          Node vs schema (type + closed model + cardinalities, no order)
  types.txt                     TypeRegistry (admitsChildren) + the 18 spec types + binaryValue (§9.5)
template/
  template_parser.txt           @stxt.template document → Schema (references, Description block)
  child_line.txt                ChildLine: (cardinality) TYPE [values]
  child_line_parser.txt         RuleSpec parser for Structure >> lines
  template_schema_provider.txt  TemplateSchemaProviderMemory + meta-template (§16)
```

## Repo conventions

- Pseudocode files in `.txt`, strictly following the [README.md](README.md) guide
  (UPPERCASE English keywords, snake_case variables, `#` comments).
- **Languages**: conversation with Joan is always in Spanish (from Spain), but **all
  repository content is in English**: README.md, `.txt` comments, identifiers, error
  messages and this CLAUDE.md.
- UPPERCASE error codes, identical to those of stxt-js/stxt-java.
- Commits: short messages; `M+` is the local convention for minor changes. Joan makes
  every commit and push himself — never run `git commit`/`git push`; leave changes in
  the working tree for his review.
