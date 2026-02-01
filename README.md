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
ELSE
WHILE
FOR
RETURN
FUNCTION
PROCEDURE
CLASS
STRUCTURE
END
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

### 4.1 Tipos comunes

Se definen los siguientes tipos básicos

* INTEGER: Números enteros
* FLOAT: Números con coma flotante
* STRING: Cadenas de caracteres
* BOOLEAN: Boolean (true/false)

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

## 8. Operadores de comparación

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

## 9. Operadores booleanos

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

## 10. Booleanos

Los valores booleanos se expresan de forma simple.

```text
true
false
```

---

## 11. Bloques e indentación

* La indentación es obligatoria y significativa para la lectura.
* No se utilizan llaves `{}` ni caracteres especiales.
* Cada bloque comienza tras una keyword y termina con su `END` correspondiente.

**Ejemplo:**

```text
IF condition
    DO something
END IF
```

---

## 12. Comentarios

Los comentarios se utilizan para explicar intención o contexto, no para describir lo obvio.

**Convención:**

* Empiezan por `#`
* En inglés

**Ejemplo:**

```text
# Validate user before processing the order
```

---

## 13. Ejemplo completo

```text
CLASS Order

    FIELD id
    FIELD total_price
    FIELD is_paid

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

## 14. Principios clave

* Consistencia antes que exhaustividad.
* Claridad antes que brevedad.
* Independencia de cualquier lenguaje concreto.
* Fácil traducción a código real.

Esta convención es adecuada para documentación técnica, diseño de sistemas, revisión de lógica y comunicación entre equipos.
