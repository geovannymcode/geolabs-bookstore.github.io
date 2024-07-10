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

### **Ejemplo de Spring AOP**

Ahora pongamos esas definiciones en un ejemplo de código donde creamos una anotación Log que registra un mensaje en la consola antes de que comience la ejecución del método.

Primero, incluyamos las dependencias de los starters de AOP y test de Spring.

```yml title="pom.xml" linenums="1"

<dependencies>  
	 <dependency> 
		 <groupId>org.springframework.boot</groupId>  
		 <artifactId>spring-boot-starter-test</artifactId>  
		 <scope>test</scope>  
	 </dependency>  
	 
	 <dependency> 
		 <groupId>org.springframework.boot</groupId>  
		 <artifactId>spring-boot-starter-aop</artifactId>  
		 <version>2.7.4</version>  
	 </dependency>
 </dependencies>
```

Ahora, vamos a crear la anotación Log que queremos utilizar:

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

Lo que esto hace es crear una anotación que sólo es aplicable a los métodos y se procesa en tiempo de ejecución.

El siguiente paso es crear la clase Aspect con un Pointcut y un Advice:

```java title="LogginAspect.java" linenums="1"
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
import org.aspectj.lang.annotation.Pointcut;  
import org.springframework.stereotype.Component;  
  
@Component  
@Aspect  
public class LoggingAspect {  
    
    @Pointcut("@annotation(Log)")  
    public void logPointcut(){  
    }  
    
    @Before("logPointcut()")  
    public void logAllMethodCallsAdvice(){  
        System.out.println("In Aspect");  
    }  
}
```

Vinculando esto con las definiciones que presentamos arriba, notamos la anotación ```@Aspect``` que marca la clase ```LoggingAspect``` como una fuente para ```@Pointcut``` y ```Advice (@Before)```. También observe que anotamos la clase como un ```@Component``` para permitir que Spring gestione esta clase como un Bean.

Además, usamos la expresión ```@Pointcut("@annotation(Log)")``` para describir qué métodos potenciales ```(JoinPoints)``` se ven afectados por el método Advice correspondiente. En este caso, queremos agregar el advice a todos los métodos que están anotados con nuestra anotación ```@Log```.

Esto nos lleva a ```@Before("logPointcut()")``` que ejecuta el método anotado ```logAllMethodCallsAdvice``` antes de la ejecución de cualquier método anotado con ```@Log```.

### **Creación y Uso de Anotaciones con Spring AOP**
Simplifica la gestión de errores creando anotaciones personalizadas y utilizando Spring AOP:

- Creación de anotaciones personalizadas.
- Uso de AOP para interceptar anotaciones.
- Crear una anotación personalizada:

Suponiendo que tenemos en nuestra clase de BookService el método getBookByCode tenemos esto.
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

```java title="Loggable .java" linenums="1"
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {
    String startMessage() default "Entering";
    String endMessage() default "Exiting";
    String errorMessage() default "Exception";
}
```

- Implementar un aspecto para manejar la anotación:
  
```java title="LoggingAspect.java" linenums="1"
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

- Implementar un aspecto para manejar la anotación:
  
```java title="LoggingAspect.java" linenums="1"
public class BookServiceMessages {
    public static final String STARTING_BOOK_BY_CODE_VALIDATIONS = "Starting book by code validations before getting the information";
    public static final String END_BOOK_MESSAGE = "Book retrieval by code completed";
    public static final String ERROR_ACCESSING_DATA_FOR_BOOK = "Error accessing data for book with code: {}";
    public static final String BOOK_NOT_FOUND_WITH_CODE = "Book not found with code: ";
    public static final String UNEXPECTED_ERROR_SAVING_BOOK = "Unexpected error saving book: {}";
}
```

- Implementar un aspecto para manejar la anotación:
  
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

### **Conclusiones**