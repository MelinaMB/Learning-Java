# Guía de VS Code

VS Code es el IDE recomendado para esta guía porque es liviano, fácil de usar y tiene extensiones muy buenas para Java.

## Qué necesitas instalar

Antes de abrir VS Code, instala estas dos cosas:

1. `Azul Zulu JDK 17 o superior`
2. `VS Code`

Si el JDK no está instalado, VS Code no podrá compilar ni ejecutar los ejemplos de Java.

## Cómo instalar Java con Azul Zulu

1. Descarga el JDK desde [Azul Zulu](https://www.azul.com/downloads/?package=jdk).
2. Instálalo con las opciones por defecto.
3. Verifica la instalación en una terminal:

```bash
java -version
javac -version
```

Si ambos comandos muestran una versión, Java quedó bien instalado.

## Instalación por sistema operativo

### macOS

1. Descarga el instalador `.pkg` de Azul Zulu para macOS.
2. Ábrelo y sigue el asistente.
3. Verifica en Terminal con `java -version` y `javac -version`.
4. Instala VS Code desde el sitio oficial o con `brew install --cask visual-studio-code` si usas Homebrew.

### Windows

1. Descarga el instalador `.msi` de Azul Zulu para Windows.
2. Ejecútalo como administrador si hace falta.
3. Asegúrate de marcar la opción para agregar Java al `PATH` si el instalador la ofrece.
4. Verifica en PowerShell o CMD con `java -version` y `javac -version`.
5. Instala VS Code desde el instalador oficial.

### Linux

1. Descarga el paquete `.deb` o `.rpm` de Azul Zulu según tu distribución.
2. Instálalo con el gestor de paquetes correspondiente.
3. Verifica en la terminal con `java -version` y `javac -version`.
4. Instala VS Code desde el gestor de paquetes o desde el sitio oficial.

## Qué revisar después de instalar

- Que `java -version` muestre Azul Zulu.
- Que `javac -version` también responda.
- Que VS Code reconozca Java al abrir una carpeta con archivos `.java`.

## Cómo instalar VS Code

1. Descarga VS Code desde [code.visualstudio.com](https://code.visualstudio.com/).
2. Instálalo normalmente.
3. Ábrelo y espera a que cargue por completo.

## Extensiones recomendadas

Instala estas extensiones desde el panel de extensiones de VS Code:

- `Extension Pack for Java`
- `Language Support for Java(TM) by Red Hat`
- `Debugger for Java`
- `Test Runner for Java`
- `Maven for Java`
- `Spring Boot Extension Pack`

Opcionalmente puedes usar:

- `Code Runner` si quieres ejecutar archivos simples rápido

## Instalación paso a paso

1. Instala Java JDK.
2. Instala VS Code.
3. Abre VS Code.
4. Instala `Extension Pack for Java`.
5. Instala `Maven for Java`.
6. Instala `Spring Boot Extension Pack`.
7. Abre la carpeta del proyecto con `File > Open Folder`.
8. Espera a que VS Code detecte el proyecto y descargue dependencias.

## Cómo comprobar que todo funciona

Abre una terminal y ejecuta:

```bash
java -version
javac -version
```

Luego, dentro de una carpeta simple, crea un archivo `Hola.java`:

```java
public class Hola {
    public static void main(String[] args) {
        System.out.println("Hola desde VS Code");
    }
}
```

Compila y ejecuta:

```bash
javac Hola.java
java Hola
```

Si ves `Hola desde VS Code`, ya está todo listo.

## Cómo ejecutar la guía

- Para ejemplos simples, usa el botón `Run` o la terminal integrada.
- Para proyectos con Maven, usa `mvn test` y `mvn compile`.
- Para Spring Boot, ejecuta la clase principal o usa la paleta de comandos.

## Consejos útiles

- Activa el guardado automático si te ayuda a aprender.
- Usa el panel de problemas para corregir errores rápido.
- Revisa los imports sugeridos por VS Code antes de seguir.
- Si una extensión no aparece, recarga la ventana con `Developer: Reload Window`.

## Problemas comunes

- Si `java -version` no funciona, Azul Zulu no está instalado o no está en el `PATH`.
- Si VS Code no reconoce Java, cierra y abre de nuevo el editor después de instalar las extensiones.
- Si un proyecto Maven no descarga dependencias, espera unos segundos y revisa la conexión a internet.

## Nota

Si ya usas otro IDE, puedes seguir la guía igual. VS Code es solo la opción preferida para este learning.
