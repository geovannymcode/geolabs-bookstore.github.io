## Integración de PayPal

### Configuracion Inicial
Para comenzar con la integración de PayPal en tu proyecto de Spring Boot, sigue estos pasos:

1. **Crear una cuenta de desarrollador en PayPal:**
   - Visita [PayPal Developer](https://developer.paypal.com) y regístrate o inicia sesión.
   - Crea una nueva aplicación desde el dashboard para obtener las credenciales necesarias (Client ID y Secret).

2. **Configurar las credenciales en tu aplicación Spring Boot:**
   - Agrega las credenciales en tu archivo `application.properties` o `application.yml`:

```properties
paypal.client.id=YOUR_CLIENT_ID
paypal.client.secret=YOUR_CLIENT_SECRET
paypal.mode=sandbox
```

Configurar el APIContext de PayPal en Spring Boot:
```java
@Configuration
public class PayPalConfig {
    @Value("${paypal.client.id}")
    private String clientId;
    @Value("${paypal.client.secret}")
    private String clientSecret;
    @Value("${paypal.mode}")
    private String mode;

    @Bean
    public APIContext apiContext() {
        return new APIContext(clientId, clientSecret, mode);
    }
}
```