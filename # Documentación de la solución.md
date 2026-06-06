# Documentación de la solución

Este proyecto consiste en una solución que integra un frontend en **Vue 3** y dos servicios de backend (**Node.js** y **Go**)

---

## Tecnologías y Despliegue

La solución se encuentra en contenedor y desplegada en la nube utilizando las siguientes plataformas:

| Componente | Tecnología | Plataforma de Despliegue | Imagen Docker Hub |
| :--- | :--- | :--- | :--- |
| **Frontend** | Vue 3 | [Vercel](https://vercel.com) | |
| **Backend Node** | Node.js / Express | [Railway](https://railway.com) | [`custox06/interseguro-api-node`](https://hub.docker.com/r/custox06/interseguro-api-node) |
| **Backend Go** | Go (Golang) | [Railway](https://railway.com) | [`custox06/interseguro-api-go`](https://hub.docker.com/r/custox06/interseguro-api-go) |

> **Nota sobre las imágenes:** El nombre de cada imagen de Docker Hub identifica explícitamente el lenguaje en el que ha sido desarrollado el servicio.

---

## Flujo de interacción

El flujo de datos entre los componentes sigue un orden secuencial protegido por autenticación basada en tokens:

1. **Frontend → Node.js:** El cliente envía la matriz de datos junto con el token JWT en las cabeceras de la petición (`Authorization: Bearer <token>`).
2. **Validación en Node.js:** El servicio de Node recibe la información y valida la autenticidad del JWT usando la clave secreta compartida.
3. **Node.js → Go:** Tras la validación, Node.js realiza una petición HTTP POST hacia el servicio de Go, pasando la matriz y el mismo token JWT en el header.
4. **Procesamiento en Go:** El servicio de Go valida nuevamente el JWT, ejecuta la **factorización QR** a la matriz y devuelve los resultados a Node.js.
5. **Node** recibe los datos de Go, y realiza los calculos adicionales como valor maximo, minimo, etc.
6. **Respuesta Final:** Node.js node procesa todo y retorna datos al frontend.

---

## Frontend (Opcional)

Se desarrolló una interfaz de usuario usando Vue3 usando vercel para subir mi proyecto

* **URL de producción:** [https://frontend-sand-six-28.vercel.app](https://frontend-sand-six-28.vercel.app)

**Variables de entorno**
``` env
VITE_API_BACKEND_NODE = 
```


> **Nota de desarrollo:** Para optimizar tiempos de desarrollo siendo opcional utilice IA (Claude) para el diseño visual.

---

## Backend

### API Node.js
* **URL Base:** `https://interseguro-api-node-production.up.railway.app`
* **Seguridad:** Implementación de políticas **CORS** para restringir accesos no autorizados y protección de endpoints mediante **JWT**.

#### Variables de Entorno Requeridas (`.env`)
```env
# URL del frontend permitida por CORS / Consumo de API
VITE_API_BACKEND_NODE={{URL_BASE}}/api/v1/

# Secret Key para la firma y verificación de JWT (Compartido con Go)
JWT_SECRET=tu_clave_secreta_aquí
```

### Enpoints de Node
1. Generar token (GET)
- Ruta ``api/v1/generate-token``
- Sin seguridad
- Aquí te devuelvo un token aleatorio, pero lo identificamos con nuestra clave secreta que pide JWT, simulando la obtención del token una vez que nuestro usuario haya ingresado credenciales a una web.

2. Proceso de la matriz (POST)
- Ruta ``api/v1``
- Autenticacion con JWT
- Enviamos el array de la matriz usando la propiedad ``matrix``
``` JSON 
{
  "matrix": [
    [1, 2],
    [3, 4]
  ]
}
```

**Variables de entorno**
``` env
FRONTEND_URL=
API_GO = {{URL_BASE_GO}}api/qr-factorization
JWT_SECRET=
```

### API GO

URL BASE ``https://interseguro-api-go-production.up.railway.app/api/qr-factorization``

Seguridad con JWT

1. Factoriza QR (POST)

- Ruta ``/api/qr-factorization``
- Aquí también envío por POST la misma propiedad matrix pasándole el array de datos.

**Variables de entorno**
``` env
JWT_SECRET = //mismo secreto que node
```


### Repositorios Públicos
Por cada proyecto he creado una carpeta y su propio repositorio independiente por si se desea revisar el código fuente detalladamente:

- Frontend (Vue 3): https://github.com/cristhiancustodio/frontend
- API Go: https://github.com/cristhiancustodio/api-go
- API Node.js: https://github.com/cristhiancustodio/api-node