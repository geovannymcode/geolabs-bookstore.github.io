# **Códigos de Estado HTTP**

Los códigos de estado HTTP son códigos de respuesta estándar proporcionados por los servidores web en respuesta a una solicitud realizada al servidor. Estos códigos ayudan a los clientes (como navegadores web o aplicaciones) a entender el resultado de la solicitud. Aquí están los códigos de estado HTTP más comunes que se utilizan en una API.

### **Código de Estado 200 (OK)**

- **Descripción:** La solicitud ha tenido éxito.
- **Uso:** Se utiliza cuando una solicitud GET, PUT o DELETE ha sido procesada correctamente. También puede usarse en una solicitud POST que no crea un nuevo recurso.
- **Ejemplo:**
    - Recuperar una lista de libros.
    - Actualizar la información de un libro existente.

```java
@GetMapping("/books")
public ResponseEntity<List<Book>> getAllBooks() {
    List<Book> books = bookService.findAll();
    return new ResponseEntity<>(books, HttpStatus.OK);
}
```

### **Código de Estado 201 (Created)**

- **Descripción**: La solicitud ha sido cumplida y ha resultado en la creación de un nuevo recurso.
- **Uso**: Se utiliza en una solicitud POST que crea un nuevo recurso.
- **Ejemplo**:
    - Crear un nuevo libro.

```java
@PostMapping("/books")
public ResponseEntity<Book> createBook(@RequestBody @Valid Book book) {
    Book createdBook = bookService.save(book);
    return new ResponseEntity<>(createdBook, HttpStatus.CREATED);
}
```

### **Código de Estado 400 (Bad Request)**

- **Descripción**: El servidor no puede procesar la solicitud debido a algo que es percibido como un error del cliente (por ejemplo, datos de solicitud mal formados).
- **Uso**: Se utiliza cuando la solicitud enviada al servidor tiene errores de validación o está mal formada.
- **Ejemplo**: Enviar datos incompletos o inválidos al crear o actualizar un recurso.

Utilizando anotaciones de validación en el DTO para asegurar que los datos proporcionados sean correctos.

```java
import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.NotNull;

public record Book(
        String id,
        @NotEmpty(message = "Book code is required") String code,
        @NotEmpty(message = "Book name is required") String name,
        String description,
        @NotNull(message = "Book price is required") @DecimalMin("0.1") BigDecimal price,
        @NotBlank String imageUrl,
        @NotBlank String filePath) {}
```

En el controlador, utilizamos `@Valid` para activar la validación.

```java
@PostMapping("/books")
public ResponseEntity<Book> createBook(@RequestBody @Valid Book book) {
    Book createdBook = bookService.save(book);
    return new ResponseEntity<>(createdBook, HttpStatus.CREATED);
}
```

### **Código de Estado 404 (Not Found)**

- **Descripción**: El servidor no puede encontrar el recurso solicitado.
- **Uso**: Se utiliza cuando un recurso específico no se encuentra en el servidor.
- **Ejemplo**:
    - Intentar recuperar un libro que no existe.
    - Refactorizando el servicio para lanzar `BookNotFoundException`.

```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public Book getBookByCode(String code) {
        Optional<BookEntity> bookEntity = bookRepository.findByCode(code);
        if (bookEntity.isEmpty()) {
            throw new BookNotFoundException("Book with code " + code + " not found");
        }
        return BookMapper.toBook(bookEntity.get());
    }
}
```

### **Código de Estado 500 (Internal Server Error)**

- **Descripción**: El servidor ha encontrado una situación que no sabe cómo manejar.
- **Uso**: Se utiliza cuando ocurre un error inesperado en el servidor.
- **Ejemplo**:
    - Fallo en el servidor debido a una excepción no manejada.

Manejo de excepciones en el servicio y lanzamiento de `InternalServerErrorException`.

```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public List<Book> findAll() {
        try {
            List<BookEntity> bookEntities = bookRepository.findAll();
            return bookEntities.stream().map(BookMapper::toBook).toList();
        } catch (Exception e) {
            throw new InternalServerErrorException("Failed to retrieve books");
        }
    }
}
```

### **Conlusión**

En esta sesión, hemos mostrado qué significan los códigos HTTP más usados en una aplicación completa de Spring Boot, abarcando desde la configuración inicial hasta funcionalidades avanzadas. Hemos cubierto la creación de entidades, repositorios, servicios, controladores y validaciones. Además, hemos manejado correctamente los códigos de estado HTTP: 200 (OK), 201 (Created), 400 (Bad Request), 404 (Not Found) y 500 (Internal Server Error).

Con estos pasos, hemos construido una aplicación robusta y bien estructurada, que te permitirá desarrollar proyectos eficientes con Spring Boot.

## **Enlace de Retorno**

Regresar a la [documentación del libro](get-book.md).