# **Creación de Entidades Order e Item**

### 1. **Entidad Sales Order**

La clase `SalesOrderEntity` representa una orden en nuestro sistema, que incluye el total de la orden, el estado del pago y los ítems asociados a esa orden.

```java title="SalesOrderEntity.java" linenums="1"
import com.jconfdominicana.bookstore.model.enums.PaymentStatus;
import jakarta.persistence.CascadeType;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.OneToMany;
import jakarta.persistence.SequenceGenerator;
import jakarta.persistence.Table;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "sales_orders")
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SalesOrderEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sales_order_id_generator")
    @SequenceGenerator(name = "sales_order_id_generator", sequenceName = "sales_order_id_seq")
    private Long id;

    private BigDecimal total;

    @Enumerated(EnumType.STRING)
    private PaymentStatus paymentStatus;

    private LocalDateTime createdAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", referencedColumnName = "id", nullable = false)
    private UserEntity customer;

    @OneToMany(mappedBy = "salesOrderEntity", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<SalesOrderItemEntity> items;
}

```

- **Línea 39**:
    - **`@Enumerated(EnumType.STRING)`**: Especifica que el campo paymentStatus es un enumerado y se almacenará como una cadena en la base de datos.

- **Línea 44-46**:
    - **`@ManyToOne`**: Esta anotación indica que existe una relación de muchos a uno entre `OrderEntity` y `User`. En términos de base de datos, esto significa que muchas órdenes pueden estar asociadas con un solo usuario. Esta relación permite que una orden esté vinculada a un único cliente.
    - **`@JoinColumn(name = "customer_id", referencedColumnName = "id", nullable = false)`**: Especifica la columna de unión para la relación con `UserEntity`. La columna `customer_id` en `SalesOrderEntity` se refiere a la columna `id` en `UserEntity` y es obligatoria (`nullable = false`).

- **Línea 48-49**:
    - **`@OneToMany(mappedBy = "salesOrderEntity", cascade = CascadeType.ALL, fetch = FetchType.LAZY)`**: Indica una relación de uno a muchos con `SalesOrderItemEntity`. La propiedad `mappedBy` se refiere al campo `salesOrderEntity` en `SalesOrderItemEntity`, `cascade = CascadeType.ALL` indica que todas las operaciones de persistencia se propagarán a los elementos relacionados, y `fetch = FetchType.LAZY` indica que la relación se cargará de manera perezosa.

- **Lombok Annotations**:
    - **`@Getter`**: Genera automáticamente métodos getter para todos los campos.
    - **`@Setter`**: Genera automáticamente métodos setter para todos los campos.
    - **`@AllArgsConstructor`**: Genera un constructor con un argumento para cada campo en la clase.
    - **`@NoArgsConstructor`**: Genera un constructor sin argumentos.
    - **`@Builder`**: Permite construir objetos de esta clase de manera fluida.

### 2. **Enumerado PaymentStatus**
El enumerado PaymentStatus define los posibles estados de pago de una orden.

```java title="PaymentStatus.java" linenums="1"
public enum PaymentStatus {
    PENDING,
    PAID
}
```

- **`PENDING`**: Indica que el pago está pendiente.
- **`PAID`**: Indica que el pago ha sido completado.

### 3. **Entidad Sales Order Item**

La clase `SalesOrderItemEntity` representa un ítem dentro de una orden de venta, incluyendo detalles como el precio, el número de descargas disponibles, y las relaciones con `BookEntity` y `SalesOrderEntity`.

```java title="SalesOrderItemEntity.java" linenums="1"
@Entity
@Table(name = "sales_order_items")
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SalesOrderItemEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sales_order_items_id_generator")
    @SequenceGenerator(name = "sales_order_items_id_generator", sequenceName = "sales_order_items_id_seq")
    private Long id;

    private BigDecimal price;

    @Column(name = "downloads_available")
    private Integer downloadsAvailable;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "book_id", referencedColumnName = "id")
    private BookEntity bookEntity;

    @JsonIgnore
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", referencedColumnName = "id")
    private SalesOrderEntity salesOrderEntity;
}
```

- **Línea 19-21**:
    - **`@ManyToOne(fetch = FetchType.LAZY)`**: Indica una relación de muchos a uno con `BookEntity`. La propiedad `fetch = FetchType.LAZY` indica que la relación se cargará de manera perezosa.
    - **`@JoinColumn(name = "book_id", referencedColumnName = "id")`**: Especifica la columna de unión para la relación con `BookEntity`. La columna `book_id` en `SalesOrderItemEntity` se refiere a la columna `id` en `BookEntity`.

- **Línea 23-26**:
    - **`@JsonIgnore`**: Anotación de Jackson para ignorar esta propiedad durante la serialización y deserialización `JSON`, evitando posibles problemas de recursión infinita.
    - **`@ManyToOne(fetch = FetchType.LAZY)`**: Indica una relación de muchos a uno con `SalesOrderEntity`. La propiedad `fetch = FetchType.LAZY` indica que la relación se cargará de manera perezosa.
    - **`@JoinColumn(name = "order_id", referencedColumnName = "id")`**: Especifica la columna de unión para la relación con `SalesOrderEntity`. La columna `order_id` en `SalesOrderItemEntity` se refiere a la columna `id` en `SalesOrderEntity`.

## **Creación de Schemas**

### 1. **Crear la secuencia y tabla sales_orders**
    
- **Crear la Secuencia sales_order_id_seq**

```sql
CREATE SEQUENCE sales_order_id_seq
    START WITH 1
    INCREMENT BY 50;
```

- **`CREATE SEQUENCE sales_order_id_seq`**: Crea una nueva secuencia llamada `sales_order_id_seq`.
START WITH 1: La secuencia comienza en 1.
- **`INCREMENT BY 50`**: La secuencia incrementa en 50 cada vez que se solicita un nuevo valor.

### 2. **Crear la Tabla `sales_orders`**

```sql
CREATE TABLE sales_orders (
    id BIGINT PRIMARY KEY DEFAULT nextval('sales_order_id_seq') NOT NULL,
    total NUMERIC NOT NULL,
    payment_status VARCHAR(255),
    created_at TIMESTAMP NOT NULL,
    customer_id BIGINT,
    FOREIGN KEY (customer_id) REFERENCES users(id)
);
```

- **`id BIGINT PRIMARY KEY DEFAULT nextval('sales_order_id_seq') NOT NULL`**:
    - **`id BIGINT PRIMARY KEY`**: Define la columna `id` como la clave primaria de tipo BIGINT.
    - **`DEFAULT nextval('sales_order_id_seq')`**: El valor por defecto se obtiene de la secuencia `sales_order_id_seq`.
    - **`NOT NULL`**: El campo no puede ser nulo.
- **`total NUMERIC NOT NULL`**: Define la columna `total` como `NUMERIC` y no puede ser nula.
- **`payment_status VARCHAR(255)`**: Define la columna `payment_status` como `VARCHAR(255)`.
- **`created_at TIMESTAMP NOT NULL`**: Define la columna `created_at` como `TIMESTAMP` y no puede ser nula.
- **`customer_id BIGINT`**: Define la columna `customer_id` como `BIGINT`.
- **`FOREIGN KEY (customer_id) REFERENCES users(id)`**: Define `customer_id` como una clave foránea que referencia a `id` en la tabla `users`.

### 3. **Crear la secuencia y `tabla sales_order_items`**

- **Crear la Secuencia sales_order_items_id_seq**

```sql
CREATE SEQUENCE sales_order_items_id_seq
    START WITH 1
    INCREMENT BY 50;
```

- **Crear la Tabla sales_order_items**

```sql
CREATE TABLE sales_order_items (
    id INT DEFAULT nextval('sales_order_items_id_seq') PRIMARY KEY,
    price NUMERIC,
    downloads_available INT,
    book_id BIGINT,
    order_id BIGINT,
    FOREIGN KEY (book_id) REFERENCES books(id),
    FOREIGN KEY (order_id) REFERENCES sales_orders(id)
);
```

- **`id INT DEFAULT nextval('sales_order_items_id_seq') PRIMARY KEY`**:
    - **`id INT PRIMARY KEY`**: Define la columna id como la clave primaria de tipo INT.
    - **`DEFAULT nextval('sales_order_items_id_seq')`**: El valor por defecto se obtiene de la secuencia `sales_order_items_id_seq`.
    - **`price NUMERIC`**: Define la columna price como NUMERIC.
    - **`downloads_available INT`**: Define la columna downloads_available como INT.
    - **`book_id BIGINT`**: Define la columna book_id como BIGINT.
    - **`order_id BIGINT`**: Define la columna order_id como BIGINT.
    - **`FOREIGN KEY (book_id) REFERENCES books(id)`**: Define book_id como una clave foránea que referencia a id en la tabla books.
    - **`FOREIGN KEY (order_id) REFERENCES sales_orders(id)`**: Define order_id como una clave foránea que referencia a id en la tabla sales_orders.

## **Creación de Repositorios**

En esta sección, vamos a detallar los repositorios `SalesOrderRepository` y `SalesOrderItemRepository` que proporcionan la interfaz para realizar operaciones CRUD (Crear, Leer, Actualizar, Eliminar) en las entidades `SalesOrderEntity` y `SalesOrderItemEntity`.

### 1. **Repositorio SalesOrderRepository**

El repositorio `SalesOrderRepository` proporciona métodos para realizar operaciones CRUD en la entidad `SalesOrderEntity`

```java title="SalesOrderRepository.java"
import org.springframework.data.jpa.repository.JpaRepository;

public interface SalesOrderRepository extends JpaRepository<SalesOrderEntity, Long> {
}
```

- **`public interface SalesOrderRepository extends JpaRepository<SalesOrderEntity, Long>`**: Define una interfaz de repositorio que extiende `JpaRepository`. Esto proporciona automáticamente métodos para realizar operaciones CRUD en la entidad `SalesOrderEntity`.
    - **`JpaRepository<SalesOrderEntity, Long>`**: Especifica que el repositorio trabaja con la entidad SalesOrderEntity y utiliza Long como tipo de dato para la clave primaria.
    - Los métodos CRUD estándar que proporciona `JpaRepository` incluyen:
        - **`save(S entity)`**: Guarda una entidad.
        - **`findById(ID id)`**: Encuentra una entidad por su ID.
        - **`findAll()`**: Encuentra todas las entidades.
        - **`deleteById(ID id)`**: Elimina una entidad por su ID.


### 2. **Repositorio SalesOrderItemRepository**

El repositorio `SalesOrderItemRepository` proporciona métodos para realizar operaciones CRUD en la entidad `SalesOrderItemEntity`, así como un método adicional para encontrar un ítem de una orden específica.

```java title="SalesOrderItemRepository.java"
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface SalesOrderItemRepository extends JpaRepository<SalesOrderItemEntity, Long> {
    Optional<SalesOrderItemEntity> findByIdAndSalesOrderEntity(Long id, SalesOrderEntity salesOrderEntity);
}
```

- **`Optional<SalesOrderItemEntity> findByIdAndSalesOrderEntity(Long id, SalesOrderEntity salesOrderEntity)`**: Método adicional definido en el repositorio para encontrar un ítem de una orden específica.
    - **`Optional<SalesOrderItemEntity> findByIdAndSalesOrderEntity(Long id, SalesOrderEntity salesOrderEntity)`**: Encuentra un ítem de una orden específica por su ID y la entidad de la orden a la que pertenece. Retorna un `Optional` que puede contener el ítem encontrado o estar vacío si no se encuentra ningún ítem que coincida con los criterios.
        - **`Long id`**: El ID del ítem de la orden.
        - **`SalesOrderEntity salesOrderEntity`**: La entidad de la orden a la que pertenece el ítem.

## **Creación del Servicio SalesOrderService**

El servicio `SalesOrderService` se encarga de la lógica de negocio relacionada con las órdenes de venta en la aplicación. Proporciona métodos para crear una nueva orden y para recuperar órdenes existentes.

- **Servicio `SalesOrderService`**

```java title="SalesOrderService.java" linenums="1"
@Service
@Transactional
public class SalesOrderService {

    private final BookRepository bookRepository;

    private final OrderRepository orderRepository;

    public SalesOrderService(BookRepository bookRepository, OrderRepository orderRepository) {
        this.bookRepository = bookRepository;
        this.orderRepository = orderRepository;
    }

    public OrderEntity createOrder(List<Long> bookIds){
        SalesOrderEntity() salesOrderEntity = new SalesOrderEntity();
        List<SalesOrderItemEntity> items =  new ArrayList<>();
        BigDecimal total = BigDecimal.ZERO;
        for( Long bookId : bookIds) {
            BookEntity book = bookRepository
                    .findById(bookId)
                    .orElseThrow(() -> new BookNotFoundException("Book not found with id: " + bookId));

            SalesOrderItemEntity salesOrderItem = new SalesOrderItemEntity();
            salesOrderItem.setBookEntity(book);
            salesOrderItem.setPrice(book.getPrice());
            salesOrderItem.setDownloadsAvailable(3);
            salesOrderItem.setOrderEntity(salesOrderEntityy);

            items.add(salesOrderItem);
            total = total.add(salesOrderItem.getPrice());
        }
        salesOrderEntity.setPaymentStatus(PaymentStatus.PENDING);
        salesOrderEntity.setCreatedAt(LocalDateTime.now());
        salesOrderEntity.setTotal(total);
        salesOrderEntity.setItems(items);

        return orderRepository.save(salesOrderEntity);
    }

    public List<SalesOrderEntity> getOrders() {
        return orderRepository.findAll();
    }

    public SalesOrderEntity getOrderById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order with id " + id + " not found"));
    }
}

```

- **Línea 2**:
    - **`@Transactional`**: Asegura que todos los métodos en esta clase se ejecuten dentro de una transacción. Si alguna parte del método falla, se realiza un rollback de todas las operaciones de la base de datos realizadas en ese método.

- **Línea 14-38**:
    - **`createOrder(List<Long> bookIds)`**: Método para crear una nueva orden de venta.
    - **`SalesOrderEntity salesOrderEntity = new SalesOrderEntity();`**: Crea una nueva instancia de `SalesOrderEntity`.
    - **`List<SalesOrderItemEntity> items = new ArrayList<>();`**: Inicializa una lista de ítems de la orden.
    - **`BigDecimal total = BigDecimal.ZERO;`**: Inicializa el total de la orden en cero.
    - **`Bucle for`**: Itera sobre los IDs de los libros proporcionados para crear ítems de la orden.
        - **`BookEntity book = bookRepository.findById(bookId).orElseThrow(...);`**: Busca el libro por su `ID` o lanza una excepción si no se encuentra.
        - **`SalesOrderItemEntity salesOrderItem = new SalesOrderItemEntity();`**: Crea una nueva instancia de `SalesOrderItemEntity`.
        - **`Setters`**: Asigna los valores correspondientes al ítem de la orden.
        - **`items.add(salesOrderItem);`**: Añade el ítem a la lista de ítems.
        - **`total = total.add(salesOrderItem.getPrice());`**: Actualiza el total de la orden.
    - **`Setters`**: Asigna valores a la orden de venta.
    - **`return orderRepository.save(salesOrderEntity);`**: Guarda la orden en la base de datos y retorna la entidad guardada.

- **Línea 40-42**:
    - **`getOrders()`**: Método para recuperar todas las órdenes de venta.
    - **`return orderRepository.findAll();`**: Retorna una lista de todas las entidades `SalesOrderEntity` de la base de datos.

- **Línea 44-47**:
    - **`getOrderById(Long id)`**: Método para recuperar una orden de venta por su `ID`.
    - **`return orderRepository.findById(id).orElseThrow(...);`**: Busca una entidad `SalesOrderEntity` por su `ID` o lanza una excepción `ResourceNotFoundException` si no se encuentra.


### **Creación del Controlador OrderController**

El controlador OrderController se encarga de manejar las solicitudes HTTP relacionadas con las órdenes de venta. Proporciona endpoints para interactuar con las órdenes, permitiendo su recuperación y, potencialmente, otras operaciones CRUD.

**Controlador `OrderControlle`**

```java title="OrderController.java" linenums="1"
@RestController
@RequestMapping("/api/orders")
public class OrderController {

   private final SalesOrderService salesOrderService;

    public OrderController(SalesOrderService salesOrderService) {
            this.salesOrderService = salesOrderService;

    }

    @GetMapping("/{id}")
    public ResponseEntity<SalesOrder> getOrder(@PathVariable Long id) {
        OrderEntity order = orderService.getOrderById(id);

        if (order != null) {
            return ResponseEntity.ok(order);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

- **Línea 12-21**:
    - **`@GetMapping("/{id}")`**: Anotación que mapea las solicitudes HTTP GET a la ruta /api/orders/{id} a este método. {id} es una variable de ruta que representa el ID de la orden.
    - **`@PathVariable Long id`**: Anotación que indica que el valor del parámetro id debe ser tomado de la variable de ruta {id}.
    - **`ResponseEntity<SalesOrderEntity> getOrder(@PathVariable Long id)`**: Método que maneja la solicitud para obtener una orden por su ID.
        - **`SalesOrderEntity order = salesOrderService.getOrderById(id);`**: Llama al servicio para obtener la orden por su ID.
        - **`if (order != null) { ... } else { ... }`**: Verifica si la orden fue encontrada.
            - **`return ResponseEntity.ok(order);`**: Si la orden es encontrada, devuelve una respuesta HTTP 200 OK con la orden en el cuerpo de la respuesta.
            - **`return ResponseEntity.notFound().build();`**: Si la orden no es encontrada, devuelve una respuesta HTTP 404 Not Found.

### **Conlusión**

En esta sesión, hemos desarrollado una funcionalidad completa para la gestión de órdenes de venta en una aplicación Spring Boot. Desde la creación de las entidades, la implementación de repositorios y servicios, hasta la configuración del controlador, hemos cubierto todos los componentes necesarios para una solución robusta y bien estructurada. Con estos componentes, hemos construido una aplicación Spring Boot sólida y preparada para gestionar órdenes de venta de manera eficaz, asegurando una arquitectura modular y escalable.