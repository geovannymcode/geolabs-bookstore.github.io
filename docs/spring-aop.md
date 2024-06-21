## Creación y Uso de Anotaciones con Spring AOP
Simplifica la gestión de errores creando anotaciones personalizadas y utilizando Spring AOP:

1. Crear una anotación personalizada:
```java 
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface HandleException {
    String value() default "";
}
```

2. Implementar un aspecto para manejar la anotación:
```java
@Aspect
@Component
public class ExceptionHandlingAspect {
    @Around("@annotation(com.example.HandleException)")
    public Object handleException(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            return joinPoint.proceed();
        } catch (Exception e) {
            // Manejo de la excepción
            throw e;
        }
    }
}
```