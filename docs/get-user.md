# **PRÁCTICA N° 1**

## **CRUD USUARIOS – CON POSTGRESQL**

* Crear una clase llamada `User` con los campos `id`, `firstName`, `lastName`, `email`, `phone`, `password`, `role`, `createdAt` y `updatedAt` .

```java
public class User {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String phone;
    private String password;
    private Role role;
    private LocalDateTime createdAt = LocalDateTime.now();;
    private LocalDateTime updatedAt;

    // getters & setters
}
```

1. Convertir la clase User en una entidad de base de datos usando las anotaciones `@Entity`, `@Id`, `@GeneratedValue`.
2. Crear su respectivo repositorio llamado `UserRepository` 
    * Extender `JpaRepository<User, Long>`.
    * Adicionar un método que valide si existe el `email`
3. Crear el schema por Flyway para la tabla `users`.
    * Definir la secuencia y la tabla en un archivo SQL de migración.  
4. Crear el servicio `UserService` 
    * Implementar la logica del negocio en el servicio `UserService`.
5. Crear el DTO `User`
    * Definir el DTO para `User` junto a sus respectivas validaciones.
6. Crear el Mapper UserMapper
    * Implementar el mapeo entre `User` y `UserDTO`.  
7. Crear e implementar un controlador `UserController` 
    * Manejar las operaciones CRUD de `User` a nivel de base de datos.
8. Probar la implementación con POSTMAN.

Finalmente, probar la implementación realizando las siguientes operaciones con POSTMAN:

* `GET` `/api/users` retorne la lista de usuarios. Códigos de respuesta: 200
* `GET` `/api/users/{id}` retorne un usuario por id. Códigos de respuesta: 200 / 404
* `POST` `/api/users` cree un nuevo usuario y lo retorne. Códigos de respuesta: 201 / 400
* `PUT` `/api/users/{id}` actualice un usuario por id y lo retorne. Códigos de respuesta: 200 / 400 / 404
* `DELETE` `/api/users/{id}` elimine un usuario por id. Códigos de respuesta: 204 / 404