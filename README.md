# Guía de estilo del Pseudocódigo

## 1. Propósito

Este pseudolenguaje se utiliza para **describir lógica, flujo de control y responsabilidades** 
de un sistema de forma clara y consistente, sin depender de la sintaxis 
ni de las restricciones de ningún lenguaje de programación específico.

---

## 2. Keywords

Las keywords representan control de flujo, definición de estructuras y operaciones fundamentales.

**Convención:**

* Siempre en **MAYÚSCULAS**
* Escritas en inglés

**Ejemplos:**

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
```

Para mejorar la legibilidad, se recomienda usar finales explícitos:

```text
END IF
END FUNCTION
END PROCEDURE
END CLASS
END STRUCTURE
```
---

## 3. Variables

Las variables representan estado o datos temporales.

**Convención:**

* `snake_case`
* En minúsculas
* Nombres descriptivos (sustantivos o sintagmas nominales)

**Ejemplos:**

```text
user_id
total_price
is_active
retry_count
```

---

## 4. Tipado de las variables

Las variables pueden mostrar el tipo para mejorar la claridad. Se usará lo siguiente:

nombre_variable: TIPO

**Ejemplo de definición**

```text
nombre_variable: STRING = "Hola"

```

### 4.1 Tipos básicos comunes

Se definen los siguientes tipos básicos

* INTEGER: Números enteros
* FLOAT: Números con coma flotante
* STRING: Cadenas de caracteres
* BOOLEAN: Boolean (true/false)
* REGEX: Expresiones regulares, alta dependencia del lenguaje

### 4.2 STRING

Se supone que STRING tiene al menos los siguientes métodos comunes:

* length()
* charAt(pos)
* trim()
* substring(ini_included,end_excluded)
* indexOf(str)

En caso de no existir, se debería implementar.

### 4.3 Arrays

Se muestran arrays como lo siguiente:

```text
array_de_strings: STRING[] = []
array_de_objetos: UnaClase[] = []
```

Suponemos que array también tiene los siguentes métodos:

* length()
* push(valor)
* pop(): Recupera y elimina el último valor
* last(): Recupera el último valor
* Se recupera un valor con nombre_variable[posicion]

### 4.4 Maps (diccionarios)

Estructura clave-valor (equivale a `HashMap` en Java, `Map`/objeto en JavaScript y `dict` en Python).

```text
prices: MAP<STRING, FLOAT> = {}
prices["coffee"] = 1.50
config: MAP = { "host": "localhost", "port": 8080 }
```

Suponemos que un map tiene los siguientes métodos:

* put(clave, valor): Inserta o actualiza
* get(clave): Recupera el valor de una clave
* containsKey(clave): Devuelve true si la clave existe
* remove(clave): Elimina una entrada
* keys(): Devuelve las claves
* size(): Número de entradas
* Se recupera/asigna un valor con nombre_variable[clave]

---

## 5. Funciones y procedimientos

Las funciones y procedimientos representan comportamiento.

### FUNCTION

* Devuelve un valor
* Usa `RETURN`

### PROCEDURE

* No devuelve valor
* Produce efectos (modifica estado, escribe logs, envía mensajes, etc.)

**Convención común:**

* `camelCase`
* Comienzan con verbo

**Ejemplos:**

```text
FUNCTION calculateTotalPrice
PROCEDURE sendNotification
```

---

## 6. Clases

Las clases representan conceptos del dominio o agrupaciones lógicas de comportamiento y estado.

**Convención:**

* `PascalCase`
* Sustantivos en singular

**Ejemplos:**

```text
Order
PaymentService
UserRepository
```

---

## 7. Asignación

La asignación se realiza con un único operador.

**Convención:**

```text
variable = expression
```

**Ejemplos:**

```text
total_price = unit_price * quantity
retry_count = retry_count + 1
```

---

## 8. Operadores aritméticos

Se utilizan los operadores aritméticos comunes a todos los lenguajes.

```text
+   -   *   /   %
```

* `%` es el módulo (resto de la división entera)

**Ejemplos:**

```text
total_price = unit_price * quantity
retry_count = retry_count + 1
is_even = (n % 2) == 0
```

---

## 9. Operadores de comparación

Se utilizan operadores comunes y reconocibles, evitando símbolos no estándar.

**Comparación:**

```text
==   !=   <   <=   >   >=
```

**Ejemplo:**

```text
IF age >= 18
```

---

## 10. Operadores booleanos

Las operaciones lógicas se expresan de forma explícita y legible.

**Convención:**

* En MAYÚSCULAS
* En inglés

```text
AND
OR
NOT
```

**Ejemplo:**

```text
IF is_active == true AND NOT is_blocked
```

---

## 11. Booleanos

Los valores booleanos se expresan de forma simple.

```text
true
false
```

---

## 12. Nulos (NULL) y el operador IS

`NULL` representa la **ausencia de valor** (equivale a `null` en Java/JavaScript y a `None` en Python).

Para comprobar si un valor es o no `NULL` se usa el operador `IS` / `IS NOT`, más legible que `==`:

```text
IF user IS NULL
    RETURN
END IF

IF result IS NOT NULL
    process(result)
END IF
```

* `IS` / `IS NOT` se usan únicamente para comparar con `NULL`.
* Para el resto de comparaciones se utilizan los operadores de §9.

---

## 13. Bloques e indentación

* La indentación es obligatoria y significativa para la lectura.
* No se utilizan llaves `{}` ni caracteres especiales.
* Cada bloque comienza tras una keyword y termina con su `END` correspondiente.

**Ejemplo:**

```text
IF age >= 65
    applyDiscount("senior")
ELSE IF age >= 18
    applyDiscount("standard")
ELSE
    applyDiscount("minor")
END IF
```

---

## 14. Control de bucles

Dentro de un bucle (`WHILE`, `FOREACH`) se puede alterar el flujo:

* `BREAK`: termina el bucle inmediatamente
* `CONTINUE`: salta a la siguiente iteración

**Ejemplo:**

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

---

## 15. Comentarios

Los comentarios se utilizan para explicar intención o contexto, no para describir lo obvio.

**Convención:**

* Empiezan por `#`
* En inglés

**Ejemplo:**

```text
# Validate user before processing the order
```

---

## 16. Ejemplo completo

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

---

## 17. Principios clave

* Consistencia antes que exhaustividad.
* Claridad antes que brevedad.
* Independencia de cualquier lenguaje concreto.
* Fácil traducción a código real.

Esta convención es adecuada para documentación técnica, diseño de sistemas, revisión de lógica y comunicación entre equipos.
