# Errores Comunes

## Al empezar

- Confundir `=` con `==`.
- Comparar `String` con `==` en lugar de `.equals()`.
- Olvidar el `;` al final de una instrucción.
- Usar nombres poco claros como `x`, `a1` o `dato1`.

## En POO

- Dejar atributos `public` sin necesidad.
- Repetir lógica en varias clases en vez de reutilizar.
- No sobrescribir `toString()` cuando el objeto se imprime mucho.
- Olvidar `equals()` y `hashCode()` cuando el objeto se usa en colecciones.

## En Java intermedio

- No cerrar recursos con `try-with-resources`.
- Usar `double` para dinero.
- Ignorar `Optional` y volver a usar `null`.
- Crear streams y tratar de reutilizarlos.

## En Spring y base de datos

- Mezclar controlador, servicio y acceso a datos en la misma clase.
- Construir SQL con concatenación de strings.
- No validar datos de entrada.
- Hacer consultas más complejas antes de entender CRUD simple.

## En testing

- Probar demasiadas cosas en un solo test.
- Hacer tests dependientes entre sí.
- Mockear todo sin necesidad.
- No separar tests unitarios de tests de integración.
