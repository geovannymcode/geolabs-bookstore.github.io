# **PRÁCTICA N° 1**

## **CRUD USUARIOS – CON POSTGRESQL**

Crear una clase llamada ```User``` con los campos id, firstName, lastName, email.
```java
public class User {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String phone;
    private String password;
    private String phone;
    private Role role;
    private LocalDateTime createdAt = LocalDateTime.now();;
    private LocalDateTime updatedAt;

    // getters & setters
}
```

* Convertir la clase User en una entidad de base de datos usando las anotaciones ```@Entity```, ```@Id```, ```@GeneratedValue```.
* Crear su respectivo repositorio llamado ```UserRepository``` que herede de ```JpaRepository<User, Long>```.
    * Adicionamos un metodo que valide si existe el ```email``` 
* Crear su respectivo servicio llamado ```UserService``` para implementar la logica del negocio.
* Crear e implementar un controlador llamado ```UserController``` para las operaciones CRUD de User a nivel de base de datos. 
* De tal forma que:
    * ```GET``` /api/users retorne la lista de usuarios. Códigos de respuesta: ```200```
    * ```GET``` /api/users/{id} retorne un usuario por id. Códigos de respuesta: ```200 / 404```
    * ```POST``` /api/users cree un nuevo usuario y lo retorne. Códigos de respuesta: ```201 / 400```
    * ```PUT``` /api/users/{id} actualice un usuario por id y lo retorne. Códigos de respuesta: ```200 / 400 / 404```
    * ```DELETE``` /api/users/{id} elimine un usuario por id. Códigos de respuesta: ```204 / 404```
    * Probar la implementación con POSTMAN.