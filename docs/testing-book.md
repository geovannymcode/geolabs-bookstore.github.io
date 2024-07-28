# **Pruebas en Aplicaciones de SpringBoot**

## **Pruebas unitarias con JUnit 5 y Mockito**

Empecemos a escribir pruebas unitarias para `UserService`. Deberíamos ser capaces de escribir pruebas unitarias para `UserService` SIN usar ninguna funcionalidad de Spring.

Vamos a crear un simulacro de `UserRepository` utilizando `Mockito.mock()` y crearemos una instancia de `UserService` utilizando el simulacro de `UserRepository`.

```java title="UserServiceTest.java" linenums="1"
public class UserServiceTest {

    private UserService userService;

    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository = Mockito.mock(UserRepository.class);
        userService = new UserService(userRepository);
    }

@Test
void shouldSaveUserSuccessfully() {
    User user = new User(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny@gmail.com", "password123", "1234567890", Role.USER);
    UserEntity userEntity = new UserEntity(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny@gmail.com", "password123", "1234567890", Role.USER, LocalDateTime.now(), null);
    UserEntity savedUserEntity = new UserEntity(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny@gmail.com", "password123", "1234567890", Role.USER, LocalDateTime.now(), LocalDateTime.now());
    User savedUser = new User(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny@gmail.com", "password123", "1234567890", Role.USER);

    given(userRepository.existsByEmail(user.getEmail())).willReturn(false);
    given(userRepository.save(any(UserEntity.class))).willReturn(savedUserEntity);

    User result = userService.createUser(user);

    assertThat(result).isNotNull();
    assertThat(result.getId()).isEqualTo(1L);

    verify(userRepository).existsByEmail(user.getEmail());
    verify(userRepository).save(any(UserEntity.class));
}

@Test
void shouldThrowErrorWhenSavingUserWithExistingEmail() {
    User user = new User(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny1@gmail.com", "password123", "1234567890", Role.USER);
   given(userRepository.existsByEmail(user.getEmail())).willReturn(true);

    assertThrows(EmailAlreadyExistsException.class, () -> {
        userService.createUser(user);
    });

    verify(userRepository).existsByEmail(user.getEmail());
    verify(userRepository, never()).save(any(UserEntity.class));

}
}
```

He creado el objeto mock `UserRepository` y la instancia `UserService` en el método `@BeforeEach` para que cada prueba tenga una configuración limpia. Aquí no estamos utilizando ninguna función de pruebas de Spring o SpringBoot como `@SpringBootTest` porque no es necesario para probar el comportamiento de `UserService`.

No voy a escribir tests para otros métodos porque simplemente están delegando las llamadas a `UserRepository`.

### **Uso de Anotaciones para Crear Mocks**

Si prefieres utilizar la magia de las anotaciones para crear un mock de `UserRepository` e inyectarlo en `UserService`, puedes utilizar `mockito-junit-jupiter` de la siguiente manera:

**Añade la dependencia mockito-junit-jupiter**

```yaml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

Utilice `@Mock` y `@InjectMocks` para crear e inyectar objetos simulados como se indica a continuación:

```java title="UserServiceAnnotatedTest.java" linenums="1"
@ExtendWith(MockitoExtension.class)
class UserServiceAnnotatedTest {

    @Mock
    private UserRepository userRepository;
        
    @InjectMocks
    private UserService userService;

    ...
    ...
}

```

### **Pruebas Unitarias para el Controlador**

Sí, quiero escribir pruebas unitarias para el controlador y quiero comprobar si el punto final REST está dando el ResponseCode HTTP adecuado o no, devolviendo el JSON esperado o no, etc.

SpringBoot proporciona la anotación `@WebMvcTest` para probar los controladores MVC de Spring. Además, las pruebas basadas en `@WebMvcTest` se ejecutan más rápido, ya que cargarán únicamente el controlador especificado y sus dependencias sin cargar toda la aplicación.

Ahora podemos escribir tests para `UserController` inyectando un bean Mock `UserService` e invocar los endpoints de la API usando `MockMvc`.

Como SpringBoot está creando la instancia `UserController`, estamos creando un mock bean UserService utilizando `@MockBean` de Spring en lugar de `@Mock` de Mockito.

```java title="UserControllerTest.java" linenums="1"
@WebMvcTest(UserController.class)
public class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    private User user;

    @BeforeEach
    void setUp() {
        user = new User(1L, "Geovanny", "Mendoza", "Geovanny Mendoza", "geovanny@gmail.com", "password123", "1234567890", Role.USER);
    }

    @Test
    void shouldReturnListOfUsers() throws Exception {
        List<User> users = Collections.singletonList(user);
        given(userService.getUsers()).willReturn(users);

        mockMvc.perform(get("/api/users")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().json("[{'id':1,'firstName':'Geovanny','lastName':'Mendoza','fullName':'Geovanny Mendoza','email':'geovanny@gmail.com','password':'password123','phone':'1234567890','role':'USER'}]"));
    }

    @Test
    void shouldCreateUserSuccessfully() throws Exception {
        given(userService.createUser(any(User.class))).willReturn(user);

        mockMvc.perform(post("/api/users")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"firstName\":\"Geovanny\",\"lastName\":\"Mendoza\",\"fullName\":\"Geovanny Mendoza\",\"email\":\"geovanny@gmail.com\",\"password\":\"password123\",\"phone\":\"1234567890\",\"role\":\"USER\"}"))
                .andExpect(status().isCreated())
                .andExpect(content().json("{'id':1,'firstName':'Geovanny','lastName':'Mendoza','fullName':'Geovanny Mendoza','email':'geovanny@gmail.com','password':'password123','phone':'1234567890','role':'USER'}"));
    }
}

```

Ahora tenemos una buena cantidad de pruebas unitarias probando varios componentes de nuestra aplicación. Pero todavía hay muchas posibilidades de que las cosas vayan mal, puede ser que tengamos algunos problemas de configuración de propiedades, podemos tener algunos errores en nuestros scripts de migración de base de datos, etc etc.

Por lo tanto, escribamos Pruebas de Integración para tener más confianza en que nuestra aplicación se está ejecutando correctamente.

### **Pruebas de integración con TestContainer**

SpringBoot proporciona un excelente soporte para las pruebas de integración. Podemos utilizar la anotación `@SpringBootTest` para cargar el contexto de la aplicación y probar varios componentes.

Empecemos escribiendo pruebas de integración para `UserController`. Como he mencionado antes, queremos probar el uso de la base de datos Postgres en lugar de la base de datos en memoria.

Añade las siguientes dependencias.

```yaml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-testcontainers</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.testcontainers</groupId>
			<artifactId>junit-jupiter</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.testcontainers</groupId>
			<artifactId>postgresql</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>io.rest-assured</groupId>
			<artifactId>rest-assured</artifactId>
			<scope>test</scope>
		</dependency>
```

Podemos utilizar el soporte TestContainers para JUnit 5 como se menciona aquí (https://www.testcontainers.org/test_framework_integration/junit_5/). Sin embargo, iniciar y detener los contenedores docker para cada prueba o cada clase de prueba podría causar que las pruebas se ejecuten lentamente. Por lo tanto, vamos a utilizar el enfoque de Singleton Containers mencionado en (https://www.testcontainers.org/test_framework_integration/manual_lifecycle_control/#singleton-containers).

Vamos a crear una clase base `AbstractIT` para que todos nuestros tests de integración puedan extenderse sin repetir la configuración común.

```java title="AbstractIT.java" linenums="1"
@SpringBootTest(webEnvironment = RANDOM_PORT)
@Import(ContainersConfig.class)
public abstract class AbstractIT {
    @LocalServerPort
    int port;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
    }

}
```

```java title="BookstoreApplicationTests.java" linenums="1"
@SpringBootTest
class BookstoreApplicationTests extends AbstractIT {

    @Test
    void contextLoads() {}
}
```

Hemos utilizado `@AutoConfigureMockMvc` para autoconfigurar `MockMvc`, y `@SpringBootTest`(webEnvironment = RANDOM_PORT) para iniciar el servidor en un puerto disponible aleatorio.

Hemos arrancado PostgreSQLContainer y utilizado @ContextConfiguration(initializers={AbstractIntegrationTest.Initializer.class}) para configurar las propiedades de conexión dinámica a la base de datos.

Ahora podemos implementar la Prueba de Integración para BookController de la siguiente manera:

```java title="BookControllerTest.java" linenums="1"
@Sql("/test-data.sql")
public class BookControllerTest extends AbstractIT {

    @Test
    void shouldReturnBooks() {
        given().contentType(ContentType.JSON)
                .when()
                .get("/api/books")
                .then()
                .statusCode(200)
                .body("data", hasSize(10))
                .body("totalElements", is(15))
                .body("pageNumber", is(1))
                .body("totalPages", is(2))
                .body("isFirst", is(true))
                .body("isLast", is(false))
                .body("hasNext", is(true))
                .body("hasPrevious", is(false));
    }
}

```

Las pruebas de BookControllerIT son muy similares a las de UserControllerTest, con la diferencia de cómo cargamos el ApplicationContext. Si utilizamos @SpringBootTest, SpringBoot iniciará la aplicación cargando la aplicación completa, por lo que las pruebas fallarán si hay algún error de configuración.

A continuación, vamos a escribir pruebas para UserRepository utilizando @DataJpaTest. Pero queremos ejecutar las pruebas contra una base de datos real no con una base de datos en memoria. Podemos utilizar @AutoConfigureTestDatabase(replace=AutoConfigureTestDatabase.Replace.NONE) para desactivar el uso de la base de datos en memoria y utilizar la base de datos configurada.

Vamos a crear PostgreSQLContainerInitializer para que cualquier prueba de repositorio pueda utilizarlo para configurar las propiedades dinámicas de la base de datos Postgres.

```java title="ContainersConfig.java" linenums="1"
@TestConfiguration(proxyBeanMethods = false)
public class ContainersConfig {
    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgresContainer() {
        return new PostgreSQLContainer<>(DockerImageName.parse("postgres:16-alpine"));
    }
}
```

Ahora podemos crear BookRepositoryTest de la siguiente manera:

```java title="BookRepositoryTest.java" linenums="1"
@DataJpaTest(
        properties = {
                "spring.test.database.replace=none",
                "spring.datasource.url=jdbc:tc:postgresql:16-alpine:///db",
        })
@Sql("/test-data.sql")
public class BookRepositoryTest {
    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldGetAllProducts() {
        List<BookEntity> books = bookRepository.findAll();
        assertThat(books).hasSize(15);
    }
}

```


Bueno, supongo que hemos aprendido algo sobre cómo escribir pruebas unitarias y pruebas de integración utilizando varias características de SpringBoot.

### **Conclusion**