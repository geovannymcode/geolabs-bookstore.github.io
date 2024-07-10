# **Obtener BookStore**

1. Crear Proyecto de Spring
2. Crear Entidad JPA para Bookstore
3. Crear Repositorio con Spring Data JPA
4. Crear Bookstore Service
5. Crear los EndPoint GET & POST para Bookstore

## **Crear Proyecto de Spring**

Ingresa a [Spring initialzr](https://start.spring.io/) y realiza la configuración del proyecto de la siguiente forma:

- **Project:** Maven
- **Language:** Java
- **Spring Boot:** 3.+ (la última versión estable)
- **Project Metadata:**
      - **Group:** com.jconfdominicana
      - **Artifact:** bookstore
      - **Name:** bookstore
- **Packaging:** Jar
- **Java:** 21

## **Adicionar Dependencias**

Agregar las siguientes dependencias:

![GeoLabs BookStore](./files/figura1.png "GeoLabs BookStore")

### **Dependencias:**

```xml title="pom.xml"
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-database-postgresql</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
```

- Luego hacer clic en el botón **"Generate"**. Automaticamente se descargará el proyecto de forma comprimida en un archivo extension **.zip**.
- Descomprimir el archivo que contiene el proyecto.
- Abrir el proyecto bookstore usando el IDE IntelliJ IDEA.

## **Creación Docker Componse**

```yml title="dev-stack.yml" linenums="1"
name: 'bookstore'
services:
  bookstore-db:
    image: postgres:16-alpine
    container_name: bookstore-db
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - "15432:5432"
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U postgres" ]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 500m
```
Comando para ejecutar docker compose: 

- Subir la base de datos: ```docker compose -f dev-stack.yml up -d```
- Detener la base de datos: ```docker compose -f dev-stack.yml down```

## **Creacion CRUD**

### **Creacion Entidad**

1. Crear los paquetes que vamos a utilizar en el proyecto ```model```, ```repository```, ```service```, ```controller```, ```dto``` y ```mapper```. 
2. Crear una clase **BookEntity** con los siguientes campos: ```id```, ```code```, ```name```, ```description```, ```price```.
3. Convertir la clase **BookEntity** en una entidad de base de datos usando la anotación ```@Entity```.
4. Agregamos las siguientes anotaciones de **Lombok**: ```@Getter``` ```@Setter``` ```@AllArgsConstructor``` ```@NoArgsConstructor```.
5. En la parte superior del campo ```id``` agregamos el siguiente código.

```java hl_lines="2 3"
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "book_id_generator")
   @SequenceGenerator(name = "book_id_generator", sequenceName = "book_id_seq")
```

### **Validando Book**

1. Adicionar las siguientes anotaciones ```@Column``` ```@NotEmpty``` y ```@DecimalMin```

### **Creacion Repositorio**

1. Crear un repositorio para la entidad **BookEntity**, representada por una interfaz Java que hereda de ```JpaRepository<Book, Integer>```.
2. Adicionamos el siguiente metodo ```Optional<BookEntity> findByCode(String code);```

### **Creacion DTO**

1. Crear un dto representada por un **record** Java con los campos ```title``` ```price``` de la entidad **BookEntity**.
  
```java title="Book.java"
public record Book(String code, String name, String description, BigDecimal price) {}
```

### **Creacion Servicio**

1. Crear un servicio para la entidad **BookEntity**, usando la anotación ```@Service``` y ```@Transactional```.
2. Crear el metodo para recuperar los libros usando paginación ```getBooks``` 
```java linenums="1"
public PagedResult<Book> getBooks(int pageNo) {
        Sort sort = Sort.by("name").ascending();
        pageNo = pageNo <= 1 ? 0 : pageNo - 1;
        Pageable pageable = PageRequest.of(pageNo, properties.pageSize(), sort);
        Page<Book> booksPage = bookRepository.findAll(pageable).map(BookMapper::toBook);

        return new PagedResult<>(
                booksPage.getContent(),
                booksPage.getTotalElements(),
                booksPage.getNumber() + 1,
                booksPage.getTotalPages(),
                booksPage.isFirst(),
                booksPage.isLast(),
                booksPage.hasNext(),
                booksPage.hasPrevious());
}
```
3. Crear el metodo para recuperar libros por el campo code ```getBookByCode```
4. Crear el metodo ```saveBook```

### **Creacion Paginación**

```java title="PagedResult.java" linenums="1"
public record PagedResult<T>(
        List<T> data,
        long totalElements,
        int pageNumber,
        int totalPages,
        boolean isFirst,
        boolean isLast,
        boolean hasNext,
        boolean hasPrevious) {}
```

### **Creacion Properties**

```java title="ApplicationProperties.java"
@ConfigurationProperties(prefix = "book")
public record ApplicationProperties(@DefaultValue("10") @Min(1) int pageSize) {}
```

- Adicionar en la clase principal Bookstore la siguiente anotación ```@ConfigurationPropertiesScan```
 
### **Creacion Mapper**

```java title="BookMapper.java" linenums="1"
public class BookMapper {
    public static Book toBook(BookEntity entity) {
        return new Book(entity.getCode(), entity.getName(), entity.getDescription(), entity.getPrice());
    }
}
```

### **Creacion Controller**

| Method      | Description                          |
| ----------- | ------------------------------------ |
| `GET`       | :material-check:     Fetch resource  |
| `POST`      | :material-content-save-check: Save resource |
| `PUT`       | :material-check-all: Update resource |
| `DELETE`    | :material-close:     Delete resource |

- En el paquete controller de la aplicación, crear una clase llamada ```BookController```.
- Convertir la clase en un Controlador Rest con la anotación ```@RestController```.
- Crear un método que devuelva un saludo de forma interactiva.
- Este método debe mapear la ruta "/hello" mediante el verbo http GET con la anotación ```@GetMapping```.
- El método debe tener un parámetro para capturar el nombre del usuario y usar su valor para incluirlo en el saludo con la anotación ```@RequestParam```. En caso que el usuario no indique el valor del parámetro establecer un valor por defecto.
- Desplegar la aplicación, desde la clase Principal del proyecto y realizar las pruebas usando el navegador.
- El resultado de la aplicación debe ser similar al siguiente:

| Method      | Description                          |
| ----------- | ------------------------------------ |
| `GET`       | :material-check:     http://localhost:8080/hello|

```Hola mundo!```

- El metodo getBooks
  
```java title="getBooks"
@GetMapping
PagedResult<Book> getBooks(@RequestParam(name = "page", defaultValue = "1") int pageNo) {
    return bookService.getBooks(pageNo);
}
```
- El metodo getBookByCode
  
```java title="getBookByCode" linenums="1"
@GetMapping("/{code}")
ResponseEntity<Book> getBookByCode(@PathVariable String code) {
    return bookService
            .getBookByCode(code)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> BookNotFoundException.forCode(code));
}
```

- El metodo post saveBook
  
```java title="saveBook" linenums="1"
@PostMapping
public ResponseEntity<Book> createBook(@RequestBody Book book) {
    Book createdBook = bookService.saveBook(book);
    return new ResponseEntity<>(createdBook, HttpStatus.CREATED);
}
```


### **Manejando Las Excepciones De La API**

Crear las excepción personalizada ResourceNotFoundException que retorne el código de estado http 404.
Refactorizar BookController para lanzar el error ResourceNotFoundException.
Refactorizar FileSystemStorageService para lanzar el error ResourceNotFoundException.
Crear la excepción BadRequestException (status 400) y refactorizar la validación del slug de los libros.

```java title="BookNotFoundException.java"
public class BookNotFoundException extends RuntimeException {
    public BookNotFoundException(String message) {
        super(message);
    }

    public static BookNotFoundException forCode(String code) {
        return new BookNotFoundException("Book with code " + code + " not found");
    }
}
```

### **Adicionar configuracion Properties**

```properties title="application.properties" linenums="1"
spring.application.name=bookstore
server.port=8081
server.shutdown=graceful
management.endpoints.web.exposure.include=*
management.info.git.mode=full

######## Book Service Configuration  #########
book.page-size=10

######## Database Configuration  #########
spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:15432/postgres}
spring.datasource.username=${DB_USERNAME:postgres}
spring.datasource.password=${DB_PASSWORD:postgres}
spring.jpa.open-in-view=false
```

### **Adicionar flyway**

```sql title="V1__create_books_table.sql"
create sequence book_id_seq start with 1 increment by 50;

create table books (
    id bigint default nextval('book_id_seq') not null ,
    code text not null unique,
    name varchar(255) not null,
    description text,
    price numeric not null,
    cover_path varchar(250) DEFAULT NULL,
    file_path varchar(250) DEFAULT NULL,
    created timestamp NOT NULL,
    modified timestamp DEFAULT NULL,
    primary key (id)
);
```

### **Datos**

```sql title="V2__add_books_data.sql"
insert into books(code, name, description, price, cover_path, file_path,created) values
('P100','The Hunger Games','Winning will make you famous. Losing means certain death...', 34.0,'','',current_timestamp),
('P101','To Kill a Mockingbird','The unforgettable novel of a childhood in a sleepy Southern town and the crisis of conscience that rocked it...', 45.40,'','', current_timestamp),
('P102','The Chronicles of Narnia','Journeys to the end of the world, fantastic creatures, and epic battles between good and evil—what more could any reader ask for in one book?...', 44.50,'','', current_timestamp);
```

## **Subida de Archivos**

### **Crear la clase StorageService**

```java title="FileSystemStorageService.java" linenums="1"
@Service
public class FileSystemStorageService {
    private final static String STORAGE_LOCATION = "mediafile";

    @PostConstruct
    public void init() {
        try {
            Files.createDirectories(Paths.get(STORAGE_LOCATION));
        } catch (IOException e) {
            throw new RuntimeException("Could not initialize storage location", e);
        }
    }

    public String store(MultipartFile file) {
        String originalFilename = file.getOriginalFilename();
        String filename = UUID.randomUUID() + "." + StringUtils.getFilenameExtension(originalFilename);

        if (file.isEmpty()) {
            throw new RuntimeException("Failed to store empty file " + filename);
        }

        try {
            InputStream inputStream = file.getInputStream();
            Files.copy(inputStream, Paths.get(STORAGE_LOCATION).resolve(filename), StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new RuntimeException("Failed to store file " + filename, e);
        }
        return filename;
    }

    public Path load(String filename) {
        return Paths.get(STORAGE_LOCATION).resolve(filename);
    }

    public Resource loadAsResource(String filename) {
        try {
            Path file = load(filename);
            Resource resource = new UrlResource(file.toUri());
            if (resource.exists() || resource.isReadable()) {
                return resource;
            } else {
                throw new ResourceNotFoundException("Could not read file: " + filename);
            }
        } catch (MalformedURLException e) {
            throw new ResourceNotFoundException("Could not read file: " + filename, e);
        }
    }

    public void delete(String filename) {
        Path file = load(filename);
        try {
            FileSystemUtils.deleteRecursively(file);
        } catch (IOException e) {
        }
    }
}
```

- Incrementar el tamaño máximo de los archivos subidos, en el archivo de ```application.properties```

- configuracion para el tamaño máximo de los archivos subidos por el user

```properties title="application.properties"
spring.servlet.multipart.max-file-size=100MB
spring.servlet.multipart.max-request-size=100MB
```

### **Conlusión**