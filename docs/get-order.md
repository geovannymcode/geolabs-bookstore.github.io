## **Order Entity**

```java title="OrderEntity.java" linenums="1"
public class OrderEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_id_generator")
    @SequenceGenerator(name = "order_id_generator", sequenceName = "order_id_seq")
    private Long id;

    private BigDecimal total;

    @Enumerated(EnumType.STRING)
    private PaymentStatus paymentStatus;

    private LocalDateTime createdAt;

    //@ManyToOne
    //@JoinColumn(name = "customer_id", referencedColumnName = "id")
   // private User customer;

    @OneToMany(mappedBy = "orderEntity", cascade = CascadeType.ALL)
    private List<OrderItemEntity> items;

}
```

## **Payment Status**

```java title="PaymentStatus.java" linenums="1"
public enum PaymentStatus {
    PENDING,
    PAID
}
```

## **OrderItemEntity**

```java title="OrderItemEntity.java" linenums="1"
public class OrderItemEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_items_id_generator")
    @SequenceGenerator(name = "order_items_id_generator", sequenceName = "order_items_id_seq")
    private Integer id;

    private Float price;

    @Column(name = "downs_ava")
    private Integer downloadsAvailable;

    @ManyToOne
    @JoinColumn(name = "book_id", referencedColumnName = "id")
    private BookEntity bookEntity;

    @JsonIgnore
    @ManyToOne
    @JoinColumn(name = "order_id", referencedColumnName = "id")
    private OrderEntity orderEntity;
}
```


* Repositorio

## **OrderRepository**

```java title="OrderRepository.java"
public interface OrderRepository extends JpaRepository<OrderEntity, Long> {}
```

## **OrderItemRepository**

```java title="OrderItemRepository.java"
public interface OrderItemRepository extends JpaRepository<OrderItemEntity, Long> {
    Optional<OrderItemEntity> findByIdAndOrderEntity(Long id, OrderEntity orderEntity);
    Optional<OrderItemEntity> findByIdAndOrderEntityId(Long id, Long orderId);
}
```

* Servicio

```java title="OrderService.java" linenums="1"

@Service
public class OrderService {

    private final BookRepository bookRepository;

    private final OrderRepository orderRepository;

    public OrderService(BookRepository bookRepository, OrderRepository orderRepository) {
        this.bookRepository = bookRepository;
        this.orderRepository = orderRepository;
    }

    public OrderEntity createOrder(List<Long> bookIds){
        OrderEntity orderEntity = new OrderEntity();
        List<OrderItemEntity> items =  new ArrayList<>();
        BigDecimal total = BigDecimal.ZERO;
        for( Long bookId : bookIds) {
            BookEntity book = bookRepository
                    .findById(bookId)
                    .orElseThrow(() -> new BookNotFoundException("Book not found with id: " + bookId));

            OrderItemEntity orderItem = new OrderItemEntity();
            orderItem.setBookEntity(book);
            orderItem.setPrice(book.getPrice());
            orderItem.setDownloadsAvailable(3);
            orderItem.setOrderEntity(orderEntity);

            items.add(orderItem);
            total = total.add(orderItem.getPrice());
        }
        orderEntity.setPaymentStatus(PaymentStatus.PENDING);
        orderEntity.setCreatedAt(LocalDateTime.now());
        orderEntity.setTotal(total);
        orderEntity.setItems(items);

        return orderRepository.save(orderEntity);
    }

    public List<OrderEntity> getOrders() {
        return orderRepository.findAll();
    }

    public OrderEntity getOrderById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order with id " + id + " not found"));
    }
}

```

* Controller

```java title="OrderController.java" linenums="1"
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private BookService bookService;
    private OrderService orderService;
    private FileSystemStorageService fileSystemStorageService;

    public OrderController(BookService bookService, OrderService orderService, FileSystemStorageService fileSystemStorageService) {
        this.bookService = bookService;
        this.orderService = orderService;
        this.fileSystemStorageService = fileSystemStorageService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderEntity> getOrder(@PathVariable Long id) {
        OrderEntity order = orderService.getOrderById(id);

        if (order != null) {
            return ResponseEntity.ok(order);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```