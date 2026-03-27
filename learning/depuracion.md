# Depuración

## Qué hacer cuando un programa falla

1. Lee el mensaje de error completo.
2. Identifica la línea indicada.
3. Revisa la causa más simple primero.
4. Agrega `System.out.println` para inspeccionar valores.
5. Prueba un cambio por vez.

## Errores típicos

- Error de compilación: suele ser sintaxis, tipos o nombres.
- Error en tiempo de ejecución: el programa compila, pero falla al correr.
- Error lógico: el programa corre, pero da un resultado incorrecto.

## Ejemplo simple

```java
int total = 10;
int divisor = 0;
System.out.println(total / divisor);
```

Esto falla en tiempo de ejecución. La causa es dividir por cero.

## Cómo pensar

- Si no compila, revisa sintaxis y tipos.
- Si compila pero falla, revisa datos de entrada y estados.
- Si corre pero está mal, revisa la lógica paso a paso.
