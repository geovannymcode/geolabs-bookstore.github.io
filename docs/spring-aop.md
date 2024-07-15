# **Anotaciones con Spring AOP**

## **¿Qué es AOP?**

La Programación Orientada a Aspectos (**AOP**, por sus siglas en inglés) es un paradigma de programación que busca extraer funcionalidades transversales, como el registro de logs, en lo que se conoce como "Aspectos".

Esto se logra añadiendo comportamiento ("Advice") al código existente sin cambiar el propio código. Especificamos a qué código queremos añadir el comportamiento utilizando expresiones especiales ("Pointcuts").

Por ejemplo, podemos indicarle al framework de AOP que registre todas las llamadas a métodos que ocurren en el sistema sin tener que añadir manualmente la declaración de registro en cada llamada a método.

### **Spring AOP**

**AOP** Es uno de los componentes principales en el framework Spring. Nos proporciona servicios declarativos, como la gestión de transacciones declarativas (la famosa anotación ```@Transactional```). Además, nos ofrece la capacidad de implementar Aspectos personalizados y utilizar el poder de AOP en nuestras aplicaciones.

Spring AOP utiliza proxies dinámicos de JDK o CGLIB para crear el proxy de un objeto objetivo determinado. Los proxies dinámicos de JDK están integrados en el JDK, mientras que CGLIB es una biblioteca común de definición de clases de código abierto (reempaquetada en ```spring-core```).

Si el objeto objetivo a ser proxificado implementa al menos una interfaz, se utiliza un proxy dinámico de JDK. Todas las interfaces implementadas por el tipo objetivo son proxificadas. Si el objeto objetivo no implementa ninguna interfaz, se crea un proxy de CGLIB.

### **Terminología básica de AOP**
Las terminologías que discutiremos no son específicas de Spring, son conceptos generales de **AOP** que Spring implementa.

Empecemos presentando los cuatro bloques principales de cualquier ejemplo de AOP en Spring.

### **JoinPoint**

 En términos simples, un JoinPoint es un punto en el flujo de ejecución de un método donde se puede insertar un Aspecto (nuevo comportamiento).

### **Advice**

Es el comportamiento que aborda preocupaciones a nivel del sistema (registro de logs, comprobaciones de seguridad, etc.). Este comportamiento está representado por un método que se ejecutará en un JoinPoint. Este comportamiento puede ejecutarse Antes, Después o Alrededor del JoinPoint, según el tipo de Advice, como veremos más adelante.

### **Pointcut**

Un Pointcut es una expresión que define en qué JoinPoints se debe aplicar un Advice determinado.

### **Aspect**

Aspect es una clase en la que definimos Pointcuts y Advices.

### **Creación y Uso de Anotaciones con Spring AOP**

En esta sección, vamos a mostrar cómo simplificar la gestión de errores creando anotaciones personalizadas y utilizando Spring AOP.

### **Paso 1: Código Inicial Sin AOP**

Supongamos que tenemos el siguiente método en nuestra clase `BookService`:

```java title="Log.java" linenums="1"
public Optional<Book> getBookByCode(String code) {
        logger.info("Starting book by code validations before getting the information", code);
        try {
            return bookRepository.findByCode(code).map(BookMapper::toBook);
        } catch (Exception e) {
            logger.error("Error accessing data for book with code: {}", code, e);
            throw new BookNotFoundException("Book not found with code: " + code);
        }
    }
```

Estamos registrando manualmente mensajes de log antes y después de la ejecución del método, así como en caso de error. Vamos a simplificar y centralizar este comportamiento utilizando AOP.

### **Paso 2: Incluir Dependencias en el pom.xml**

Primero, incluimos las dependencias necesarias para AOP en nuestro archivo `pom.xml`.

```yml title="pom.xml" linenums="1"

<dependencies> 
	 <dependency> 
		 <groupId>org.springframework.boot</groupId>  
		 <artifactId>spring-boot-starter-aop</artifactId>  
		 <version>2.7.4</version>  
	 </dependency>
 </dependencies>
```

- **`spring-boot-starter-aop`**: Incluye las dependencias necesarias para utilizar Aspect-Oriented Programming (AOP) en Spring Boot.

### **Paso 3: Crear la Anotación Log**

Creamos una anotación llamada `Log` que será utilizada para interceptar métodos y registrar mensajes en la consola.

```java title="Log.java" linenums="1"
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
 
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Log {  
}
```

- **`@interface`**: Utilizado para definir una anotación personalizada en Java. Las anotaciones son un mecanismo para agregar metadatos a tu código. En este caso, estamos creando una anotación llamada Log.
- **`@Target(ElementType.METHOD)`**: Indica que esta anotación solo se puede aplicar a métodos.
- **`@Retention(RetentionPolicy.RUNTIME)`**: Indica que la anotación estará disponible en tiempo de ejecución, permitiendo que los aspectos de AOP la intercepten.

### **Paso 4: Implementar un Aspecto para Manejar la Anotación**

En este paso, vamos a implementar un aspecto que maneje la anotación `Log` y registre mensajes personalizados antes, después y en caso de una excepción durante la ejecución del método. Este aspecto permitirá centralizar el logging de manera que no tengamos que repetir código en cada método.

```java title="LogginAspect.java" linenums="1"
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.util.stream.Collectors;
import java.util.stream.IntStream;

@Component
@Aspect
public class LoggingAspect {
    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    @Around("@annotation(com.jconfdominicana.bookstore.aspect.Loggable)")
    public Object logExecution(ProceedingJoinPoint joinPoint) throws Throwable {
        Loggable loggable = getLoggableAnnotation(joinPoint);
        String className = joinPoint.getSignature().toShortString();
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        String[] parameterNames = methodSignature.getParameterNames();
        Object[] arguments = joinPoint.getArgs();
        String argsString = IntStream.range(0, arguments.length)
                .mapToObj(i -> parameterNames[i] + "=" + arguments[i])
                .collect(Collectors.joining(", "));

        try {
            logger.info("{} from {} with arguments {}", loggable.startMessage(), className, argsString);
            Object response = joinPoint.proceed();
            logger.info("{} from {} with result: {}", loggable.endMessage(), className, response);
            return response;
        } catch (Throwable e) {
            logger.error("{} error=[{}] thrown from {} with arguments {}", loggable.errorMessage(), e.getMessage(), className, argsString);
            throw e;
        }
    }

    private Loggable getLoggableAnnotation(ProceedingJoinPoint joinPoint) {
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        return methodSignature.getMethod().getAnnotation(Loggable.class);
    }
}
```

- **Anotaciones y Componentes**
    - **@Component**: Indica que esta clase es un componente gestionado por Spring, permitiendo que sea detectada y registrada como un bean en el contexto de Spring.
    - **@Aspect**: Indica que esta clase define un aspecto, que es una modularización de una preocupación transversal (como el logging) que puede aplicarse a varios puntos de la aplicación.

- **Línea 15**: 
```java
private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);
```
- **`Logger logger`**: Utiliza SLF4J y Logback para registrar mensajes de log. `LoggerFactory.getLogger(LoggingAspect.class)` crea una instancia de logger específica para esta clase.

**Advice `@Around`**

```java
@Around("@annotation(com.jconfdominicana.bookstore.aspect.Loggable)")
public Object logExecution(ProceedingJoinPoint joinPoint) throws Throwable {
```

- **@Around("@annotation(com.jconfdominicana.bookstore.aspect.Loggable)")**: Define un advice que se ejecuta alrededor de la ejecución de métodos 
anotados con @Loggable. Esto significa que el advice se ejecutará antes, durante y después de la ejecución del método.
- **ProceedingJoinPoint joinPoint**: Representa el punto de ejecución del método interceptado, permitiendo ejecutar el método y obtener información sobre él.

**Obtener la Anotación Loggable**

```java
Loggable loggable = getLoggableAnnotation(joinPoint);
```

- **Loggable loggable**: Obtiene la instancia de la anotación Loggable del método interceptado para acceder a sus valores.
Obtener Información del Método

```java
String className = joinPoint.getSignature().toShortString();
MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
String[] parameterNames = methodSignature.getParameterNames();
Object[] arguments = joinPoint.getArgs();
String argsString = IntStream.range(0, arguments.length)
        .mapToObj(i -> parameterNames[i] + "=" + arguments[i])
        .collect(Collectors.joining(", "));
```

- **className**: Obtiene el nombre del método interceptado.
- **MethodSignature methodSignature**: Convierte la firma del punto de ejecución en una firma de método para obtener más detalles.
- **parameterNames**: Obtiene los nombres de los parámetros del método.
- **arguments**: Obtiene los valores de los argumentos del método.
- **argsString**: Convierte los nombres y valores de los parámetros en una cadena para su registro.

**Registro de Mensajes y Ejecución del Método**

```java
try {
    logger.info("{} from {} with arguments {}", loggable.startMessage(), className, argsString);
    Object response = joinPoint.proceed();
    logger.info("{} from {} with result: {}", loggable.endMessage(), className, response);
    return response;
} catch (Throwable e) {
    logger.error("{} error=[{}] thrown from {} with arguments {}", loggable.errorMessage(), e.getMessage(), className, argsString);
    throw e;
}
```

- **logger.info**: Registra el mensaje de inicio con los argumentos del método.
- **Object response = joinPoint.proceed()**: Ejecuta el método interceptado y obtiene su resultado.
- **logger.info**: Registra el mensaje de finalización con el resultado del método.
- **catch (Throwable e)**: Captura cualquier excepción lanzada durante la ejecución del método.
- **logger.error**: Registra el mensaje de error con la excepción lanzada.
- **throw e**: Vuelve a lanzar la excepción para que el flujo de control original no se vea alterado.

**Método Auxiliar `getLoggableAnnotation`**

```java
private Loggable getLoggableAnnotation(ProceedingJoinPoint joinPoint) {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    return methodSignature.getMethod().getAnnotation(Loggable.class);
}
```

- **MethodSignature methodSignature**: Convierte la firma del punto de ejecución en una firma de método.
- **methodSignature.getMethod().getAnnotation(Loggable.class)**: Obtiene la anotación Loggable del método interceptado.

### **Paso 5: Crear Mensajes de Servicio**

Esta clase contiene constantes con mensajes utilizados en el logging.
  
```java title="LoggingAspect.java" linenums="1"
public class BookServiceMessages {
    public static final String STARTING_BOOK_BY_CODE_VALIDATIONS = "Starting book by code validations before getting the information";
    public static final String END_BOOK_MESSAGE = "Book retrieval by code completed";
    public static final String ERROR_ACCESSING_DATA_FOR_BOOK = "Error accessing data for book with code: {}";
    public static final String BOOK_NOT_FOUND_WITH_CODE = "Book not found with code: ";
    public static final String UNEXPECTED_ERROR_SAVING_BOOK = "Unexpected error saving book: {}";
}
```

### **Paso 6: Aplicar la Anotación Loggable en el Servicio**

Finalmente, aplicamos la anotación `Loggable` en el método `getBookByCode` del servicio `BookService`.
  
```java title="LoggingAspect.java" linenums="1"
@Loggable(startMessage = BookServiceMessages.STARTING_BOOK_BY_CODE_VALIDATIONS,
          endMessage = BookServiceMessages.END_BOOK_MESSAGE,
          errorMessage = BookServiceMessages.ERROR_ACCESSING_DATA_FOR_BOOK)
public Optional<Book> getBookByCode(String code) {
    try {
        return bookRepository.findByCode(code).map(BookMapper::toBook);
    } catch (Exception e) {
        throw new BookNotFoundException(BookServiceMessages.BOOK_NOT_FOUND_WITH_CODE + code);
    }
}
```

- **@Loggable(startMessage = ..., endMessage = ..., errorMessage = ...)**: Aplica la anotación `Loggable` al método, especificando los mensajes personalizados para inicio, fin y error.

### **Conclusiones**

En esta sesión, hemos mostrado cómo configurar Spring AOP para crear una anotación personalizada `Loggable` que intercepta métodos y registra mensajes personalizados antes, después y en caso de una excepción durante la ejecución del método. Esta configuración facilita el registro de logs detallados y consistentes, mejorando la capacidad de depuración y monitoreo de la aplicación.