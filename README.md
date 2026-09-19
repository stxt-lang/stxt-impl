# stxt-impl

The platform-neutral implementation of [**STXT**](https://stxt.dev), written in a small
pseudo-language. A port to any programming language can be written from this repository and
the specifications alone.

It is the second authority of the ecosystem, after the specifications in
[stxt-lang](https://github.com/stxt-lang/stxt-lang). When a port disagrees with either one,
the port is corrected.

An excerpt of [`core/node.txt`](core/node.txt):

```text
FUNCTION getChildren(cname: STRING, child_namespace: STRING): Node[]
    key: STRING = normalizeChars(cname)
    result: Node[] = []
    FOREACH child IN children
        IF child.getCanonicalName() == key AND child.getNamespace() == child_namespace
            result.push(child)
        END IF
    END FOREACH
    RETURN result
END FUNCTION
```

| Directory | Contents |
|---|---|
| `core/` | The parser and its result, the node model, the writer, the formatter, the canonical JSON tree, constants, string utilities and validations |
| `processors/` | The extension points: `Observer`, `StreamObserver` and `Validator` |
| `schema/` | Schemas: parser, provider, validator, definitions and value types |
| `template/` | Templates and their compilation to schemas |
| `discovery/` | The resolution of definitions (STXT-DISCOVERY-SPEC) |
| `exceptions/` | The exception hierarchy and its error codes |

- [`TRACEABILITY.md`](TRACEABILITY.md) maps every rule of the specifications to the file that
  implements it, and records each change.
- The ports written from this pseudocode are [stxt-js](https://github.com/stxt-lang/stxt-js),
  [stxt-java](https://github.com/stxt-lang/stxt-java) and
  [stxt-python](https://github.com/stxt-lang/stxt-python).

The rest of this file is the style guide that every `.txt` file follows.

# Pseudocode style guide

## 1. Purpose

This pseudo-language is used to **describe logic, control flow and responsibilities**
of a system in a clear and consistent way, without depending on the syntax
or the constraints of any specific programming language.

## 2. Keywords

Keywords represent control flow, structure definition and fundamental operations.

**Convention:**

* Always in **UPPERCASE**
* Written in English

**Examples:**

```text
IF
ELSE IF
ELSE
WHILE
FOREACH line IN text
BREAK
CONTINUE
RETURN
FUNCTION
PROCEDURE
CLASS
END
NULL
CONSTRUCTOR
IS
FIELD
CONSTANT
INTERFACE
IMPLEMENTS
EXTENDS
ABSTRACT
THROW
TRY
CATCH
```

For readability, explicit block endings are recommended:

```text
END IF
END FUNCTION
END PROCEDURE
END CLASS
END STRUCTURE
```

## 3. Variables

Variables represent state or temporary data.

**Convention:**

* `snake_case`
* Lowercase
* Descriptive names (nouns or noun phrases)

**Examples:**

```text
user_id
total_price
is_active
retry_count
```

## 4. Variable typing

Variables may show their type to improve clarity, as follows:

variable_name: TYPE

**Definition example**

```text
variable_name: STRING = "Hello"

```

### 4.1 Common basic types

The following basic types are defined:

* INTEGER: Whole numbers
* FLOAT: Floating point numbers
* STRING: Character strings
* BOOLEAN: Boolean (true/false)
* REGEX: Regular expressions, highly language dependent

### 4.2 STRING

STRING is assumed to have at least the following common methods:

* length()
* charAt(pos)
* trim()
* substring(ini_included,end_excluded)
* indexOf(str)

If missing, they should be implemented.

### 4.3 Arrays

Arrays are shown as follows:

```text
array_of_strings: STRING[] = []
array_of_objects: SomeClass[] = []
```

Arrays are also assumed to have the following methods:

* length()
* push(value)
* pushAll(array): Appends every value of another array at the end
* pop(): Retrieves and removes the last value
* last(): Retrieves the last value
* contains(value): Returns true if the value exists in the array
* A value is retrieved with variable_name[position]

### 4.4 Maps (dictionaries)

Key-value structure (equivalent to `HashMap` in Java, `Map`/object in JavaScript and `dict` in Python).

```text
prices: MAP<STRING, FLOAT> = {}
prices["coffee"] = 1.50
config: MAP = { "host": "localhost", "port": 8080 }
```

A map is assumed to have the following methods:

* put(key, value): Inserts or updates
* get(key): Retrieves the value of a key
* containsKey(key): Returns true if the key exists
* remove(key): Removes an entry
* keys(): Returns the keys
* size(): Number of entries
* A value is retrieved/assigned with variable_name[key]

## 5. Functions and procedures

Functions and procedures represent behavior.

### FUNCTION

* Returns a value
* Uses `RETURN`

### PROCEDURE

* Returns no value
* Produces effects (modifies state, writes logs, sends messages, etc.)

**Common convention:**

* `camelCase`
* Start with a verb

**Examples:**

```text
FUNCTION calculateTotalPrice
PROCEDURE sendNotification
```

## 6. Classes

Classes represent domain concepts or logical groupings of behavior and state.

**Convention:**

* `PascalCase`
* Singular nouns

**Examples:**

```text
Order
PaymentService
UserRepository
```

A class name may coincide with a basic type name (§4.1) when mirroring a domain
concept (e.g. the schema value types `BOOLEAN` or `INTEGER`): context disambiguates:
`x: INTEGER` is a type annotation, `CLASS INTEGER` declares a class.

Inside a method, `this_node` (or, generally, `this_<class>`) names the object the method
runs on, when it has to be passed or compared explicitly; ports render it as `this`/`self`.

`ABSTRACT CLASS` declares a class that is never instantiated on its own; inside it,
`ABSTRACT FUNCTION` / `ABSTRACT PROCEDURE` declare a member that every concrete subclass
(`CLASS X EXTENDS Base`) must provide. When the blueprint says the subclasses listed are the
*only* ones (a closed hierarchy), it says so in a comment; the pseudocode has no keyword for it,
and each port uses its own means (a sealed class, a discriminated union, a check in the
constructor).

```text
ABSTRACT CLASS Node
    ABSTRACT FUNCTION getText(): STRING
END CLASS

CLASS InlineNode EXTENDS Node
    FUNCTION getText(): STRING
        RETURN value
    END FUNCTION
END CLASS
```

## 7. Assignment

Assignment uses a single operator.

**Convention:**

```text
variable = expression
```

**Examples:**

```text
total_price = unit_price * quantity
retry_count = retry_count + 1
```

## 8. Arithmetic operators

The arithmetic operators common to every language are used.

```text
+   -   *   /   %
```

* `%` is the modulo (remainder of the integer division)

**Examples:**

```text
total_price = unit_price * quantity
retry_count = retry_count + 1
is_even = (n % 2) == 0
```

## 9. Comparison operators

Common and recognizable operators are used, avoiding non-standard symbols.

**Comparison:**

```text
==   !=   <   <=   >   >=
```

**Example:**

```text
IF age >= 18
```

## 10. Boolean operators

Logical operations are expressed explicitly and readably.

**Convention:**

* UPPERCASE
* In English

```text
AND
OR
NOT
```

**Example:**

```text
IF is_active == true AND NOT is_blocked
```

## 11. Booleans

Boolean values are the two literals:

```text
true
false
```

## 12. NULL and the IS operator

`NULL` represents the **absence of a value** (equivalent to `null` in Java/JavaScript and `None` in Python).

To check whether a value is `NULL` or not, the `IS` / `IS NOT` operator is used, more readable than `==`:

```text
IF user IS NULL
    RETURN
END IF

IF result IS NOT NULL
    process(result)
END IF
```

* `IS` / `IS NOT` are used to compare against `NULL`, and to test the concrete class of a
  value: `IF node IS InlineNode` (a port renders it as `instanceof`, pattern matching, a
  discriminant field...). Inside that branch the value is used as that class.
* Every other comparison uses the operators of §9.

## 13. Blocks and indentation

* Indentation is mandatory and significant for reading.
* No braces `{}` or special characters are used.
* Each block starts after a keyword and ends with its matching `END`.

**Example:**

```text
IF age >= 65
    applyDiscount("senior")
ELSE IF age >= 18
    applyDiscount("standard")
ELSE
    applyDiscount("minor")
END IF
```

## 14. Loop control

Inside a loop (`WHILE`, `FOREACH`) the flow can be altered:

* `BREAK`: ends the loop immediately
* `CONTINUE`: jumps to the next iteration

**Example:**

```text
FOREACH item IN items
    IF item.is_invalid == true
        CONTINUE
    END IF
    IF item.is_last == true
        BREAK
    END IF
    process(item)
END FOREACH
```

## 15. Exceptions

Exceptions are thrown with `THROW` and caught with `TRY` / `CATCH`.

* There may be several `CATCH` blocks, from the most specific to the most general.
* The caught variable may be typed with `name: ExceptionType`; without a type, it catches any exception.
* Every exception has `getMessage()`. STXT exceptions also carry an UPPERCASE error
  code, which must match across implementations.

**Example:**

```text
TRY
    processLine(line)
CATCH pe: ParseException
    result.addError(pe)
CATCH e
    result.addError(ParseException(line_number, "UNEXPECTED_ERROR", e.getMessage()))
END TRY
```

## 16. Comments

Comments are used to explain intent or context, not to describe the obvious.

**Convention:**

* Start with `#`
* In English

**Example:**

```text
# Validate user before processing the order
```

## 17. Complete example

```text
CLASS Order

    FIELD id
    FIELD total_price
    FIELD is_paid
    
    CONSTRUCTOR (id, total_price, is_paid)
    	id = id
    	total_price = total_price
    	is_paid = is_paid    	
    END CONSTRUCTOR

    PROCEDURE markAsPaid()
        is_paid = true
    END PROCEDURE

END CLASS

FUNCTION processOrder(order, payment_service)

    payment_successful = payment_service.processPayment(order)

    IF payment_successful == true
        order.markAsPaid()
        RETURN true
    ELSE
        logError("Payment failed for order " + order.id)
        RETURN false
    END IF

END FUNCTION
```

## 18. Key principles

* Consistency over exhaustiveness.
* Clarity over brevity.
* Independence from any concrete language.
* Easy translation to real code.
