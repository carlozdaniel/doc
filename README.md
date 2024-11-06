
# Integración de Autenticación con Doorkeeper y Devise

Hice el cambio para autenticación tanto con Devise (usuarios) como con Doorkeeper (sistemas externos). Ahora los endpoints pueden ser utilizados por ambos métodos de autenticación sin conflictos.

## Implementación

### 1. Autenticación Dual en `ApplicationController`

Agregué el método `authenticate_user_or_doorkeeper!` en `ApplicationController`. Este método permite que el controlador `ProductsController` acepte autenticación de Devise o de Doorkeeper, de modo que los endpoints funcionan para usuarios y sistemas externos.

### 2. Elección de Doorkeeper

Decidí utilizar Doorkeeper para permitir que sistemas externos se autentiquen con un token de acceso sin requerir una cuenta de usuario de Devise. Esto facilita la integración entre el sistema y aplicaciones externas de manera más flexible.

### 3. Configuración de Doorkeeper

Configuré Doorkeeper en `config/initializers/doorkeeper.rb` y habilité el flujo `client_credentials`, el cual permite a aplicaciones externas obtener un `access_token` para autenticarse:

```ruby
grant_flows %w[client_credentials authorization_code]
```

### 4. Cambios en el Controlador

En `ApplicationController`, añadí el método `authenticate_user_or_doorkeeper!`, que verifica si la solicitud contiene un `doorkeeper_token` (autenticación Doorkeeper) o una sesión de Devise:

```ruby
def authenticate_user_or_doorkeeper!
  if doorkeeper_token
    doorkeeper_authorize!
  else
    authenticate_user!
  end
end
```

Luego, en `ProductsController`, reemplacé `authenticate_user!` por `authenticate_user_or_doorkeeper!` en los métodos que ahora estarán accesibles para clientes externos.

### 5. Verificación de Credenciales en la Consola de Rails

Para confirmar que las credenciales de cliente están configuradas correctamente, abrí la consola de Rails y ejecuté el siguiente código para ver los `client_id` y `client_secret` de las aplicaciones registradas:

```ruby
Doorkeeper::Application.all.each do |app|
  puts "Name: #{app.name}, Client ID: #{app.uid}, Client Secret: #{app.secret}"
end
```

### 6. Prueba de Autenticación con `curl`

Probé la autenticación usando el siguiente comando `curl`, reemplazando `<client_id>` y `<client_secret>` con los valores obtenidos en el paso anterior:

```bash
curl -X POST http://localhost:3000/oauth/token \
  -d "grant_type=client_credentials" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  -d "scope=public"
```

Si todo está configurado correctamente, este comando devolverá un `access_token`, que se puede usar en las solicitudes a la API.

### 7. Ejecución de Pruebas con RSpec

Para asegurarme de que todo funciona correctamente, ejecuté las pruebas en RSpec desde la raíz del proyecto:

```bash
rspec
```

## Commit de Referencia

Puedes ver los cambios en el siguiente commit: [Commit de Integración de Doorkeeper](https://github.com/carlozdaniel/ecommerce_api/commit/e8e2eb16c927c751767db07bc82d5991377300cc)

## Contacto

Para cualquier duda, puedes contactarme en mi número **3141004970** o por correo electrónico a **carlozdaniel45@gmail.com**. Quedo atento a cualquier consulta o retroalimentación.
