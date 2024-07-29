# **Creación de Entidades Order e Item**

## **Introducción**

En esta sección del taller, vamos a crear tres clases fundamentales para gestionar órdenes de venta y los ítems asociados a estas órdenes en nuestro sistema: `SalesOrderEntity`, `SalesOrderItemEntity`, y `PaymentStatus`. Estas clases nos permitirán manejar de forma estructurada y eficiente la información relacionada con las ventas y sus detalles.

### 1. **Entidad Sales Order**

#### **Descripción**

La clase `SalesOrderEntity` representa una orden en nuestro sistema, que incluye el total de la orden, el estado del pago y los ítems asociados a esa orden. Esta entidad es esencial para mantener un registro completo y detallado de cada venta realizada.

- **Propiedades de SalesOrderEntity**
    - **ID**: Identificador único de la orden.
    - **Total**: Monto total de la orden.
    - **Estado del Pago**: Indica si el pago ha sido completado, está pendiente, etc.
    - **Fecha de Creación**: Fecha en la que se creó la orden.
    - **Cliente**: Referencia a la entidad UserEntity que representa al cliente que realizó la orden.
    - **Ítems**: Lista de ítems incluidos en la orden.

- **Importancia**
La entidad `SalesOrderEntity` es crucial porque nos permite:
    - **Rastrear Ventas**: Registrar cada venta realizada, incluyendo detalles como el monto total y el estado del pago.
    - **Generar Informes**: Proporcionar datos que pueden ser utilizados para generar informes de ventas y análisis financieros.
    - **Gestionar Pedidos**: Facilitar la gestión de pedidos, permitiendo a los administradores ver el estado y los detalles de cada orden.

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

#### **Descripción**

La enumeración `PaymentStatus` define los posibles estados de pago para una orden. Esto nos permite estandarizar y controlar los valores que representan el estado del pago.

Valores de `PaymentStatus`

- **PENDING**: El pago está pendiente.
- **PAID**: El pago ha sido completado.

```java title="PaymentStatus.java" linenums="1"
public enum PaymentStatus {
    PENDING,
    PAID
}
```

### 3. **Entidad Sales Order Item**

#### **Descripción**

La clase `SalesOrderItemEntity` representa un ítem individual dentro de una orden de venta. Cada ítem está asociado a un producto específico y tiene propiedades como la cantidad y el precio.

**Propiedades de `SalesOrderItemEntity`**

- **ID**: Identificador único del ítem, generado mediante una secuencia.
- **Precio**: Precio del ítem en la orden.
- **Descargas Disponibles**: Número de descargas disponibles para el ítem, si aplica.
- **Libro**: Referencia a la entidad BookEntity que representa el libro asociado a este ítem.
- **Orden de Venta**: Referencia a la orden de venta (SalesOrderEntity) a la que pertenece este ítem.

**Importancia**
La entidad `SalesOrderItemEntity` es fundamental porque nos permite:

- **Desglosar Ventas**: Ver los detalles específicos de cada producto vendido en una orden.
- **Calcular Totales**: Calcular el total de la orden sumando los precios de cada ítem.
- **Analizar Productos**: Analizar qué productos se venden más y ajustar inventarios y estrategias de marketing en consecuencia.

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

    private final SalesOrderRepository orderRepository;

    public SalesOrderService(BookRepository bookRepository, SalesOrderRepository orderRepository) {
        this.bookRepository = bookRepository;
        this.orderRepository = orderRepository;
    }

    public SalesOrderEntity createOrder(List<Long> bookIds){
        SalesOrderEntity salesOrderEntity = new SalesOrderEntity();
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
            salesOrderItem.setSalesOrderEntity(salesOrderEntity);

            items.add(salesOrderItem);
            total = total.add(salesOrderItem.getPrice());
        }
        salesOrderEntity.setPaymentStatus(PaymentStatus.PENDING);
        salesOrderEntity.setCreatedAt(LocalDateTime.now());
        salesOrderEntity.setTotal(total);
        salesOrderEntity.setItems(items);

        return orderRepository.save(salesOrderEntity);
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

### **Conlusión**

En esta sesión, hemos desarrollado una funcionalidad completa para la gestión de órdenes de venta en una aplicación Spring Boot. Desde la creación de las entidades, la implementación de repositorios y servicios, hasta la configuración del controlador, hemos cubierto todos los componentes necesarios para una solución robusta y bien estructurada. Con estos componentes, hemos construido una aplicación Spring Boot sólida y preparada para gestionar órdenes de venta de manera eficaz, asegurando una arquitectura modular y escalable.