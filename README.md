# Kairós La Paz - Frontend

Frontend oficial del Sistema Digital Kairós La Paz.

Aplicación web desarrollada para centralizar la interacción de integrantes, participantes, servidores, directores de campamento, tesorería y coordinación con el sistema administrativo de Kairós.

El frontend consume la API REST desarrollada en FastAPI y no contiene lógica de negocio crítica ni información sensible que deba considerarse confiable.

---

## 1. Objetivo

El frontend proporciona las interfaces necesarias para:

* Sitio público de Kairós La Paz.
* Creación y gestión de cuentas.
* Autenticación y sesiones.
* Consulta de campamentos.
* Registro a campamentos.
* Formularios dinámicos.
* Consulta de información personal.
* Gestión de comisiones.
* Gestión de inventarios.
* Gestión de tribus y líderes.
* Gestión administrativa.
* Gestión de pagos.
* Gestión financiera.
* Tienda Kairós.
* Pedidos y seguimiento.
* Notificaciones.
* Reportes y exportaciones.

La aplicación debe funcionar correctamente en dispositivos de escritorio, tabletas y teléfonos.

---

# 2. Stack tecnológico

| Tecnología      | Uso                                         |
| --------------- | ------------------------------------------- |
| React 18        | Biblioteca principal de interfaz            |
| TypeScript      | Tipado estático                             |
| Vite            | Build tool y servidor de desarrollo         |
| Axios           | Cliente HTTP                                |
| React Router    | Routing                                     |
| TanStack Query  | Estado remoto, caché y sincronización       |
| Zustand         | Estado global complejo cuando sea necesario |
| Context API     | Estado global pequeño y transversal         |
| CSS             | Estilos                                     |
| Node.js 18+     | Runtime de desarrollo                       |
| npm             | Gestión de dependencias                     |
| Vitest          | Testing                                     |
| Testing Library | Testing de componentes                      |
| Playwright      | Pruebas E2E                                 |
| OpenAPI         | Contrato entre frontend y backend           |

Las dependencias adicionales deberán justificarse antes de incorporarse al proyecto.

---

# 3. Arquitectura general

La arquitectura sigue una separación clara entre presentación, estado, acceso a datos y servicios.

```text
Usuario
   |
   v
React Application
   |
   +-- Pages
   |
   +-- Components
   |
   +-- Layouts
   |
   +-- Hooks
   |
   +-- State
   |
   +-- TanStack Query
   |
   +-- Services
   |
   v
Axios HTTP Client
   |
   v
FastAPI REST API
   |
   v
PostgreSQL
```

El frontend no accede directamente a PostgreSQL.

Toda operación de datos debe realizarse mediante la API.

---

# 4. Principios arquitectónicos

El proyecto seguirá los siguientes principios:

1. Separación de responsabilidades.
2. Componentes reutilizables.
3. Tipado estricto.
4. Validación en frontend para UX.
5. Validación obligatoria en backend para seguridad.
6. No duplicar lógica de negocio innecesariamente.
7. Evitar estado global cuando no sea necesario.
8. Uso de caché para datos remotos.
9. Interfaces accesibles.
10. Diseño responsive.
11. Manejo explícito de estados de carga y error.
12. No almacenar secretos en el cliente.
13. No confiar en permisos enviados o modificados por el cliente.
14. Mantener sincronizados los tipos con OpenAPI.
15. Evitar dependencias innecesarias.

---

# 5. Estructura del proyecto

Estructura propuesta:

```text
kairos-frontend/
├── public/
├── src/
│   ├── assets/
│   │
│   ├── components/
│   │   ├── common/
│   │   ├── forms/
│   │   ├── camps/
│   │   ├── commissions/
│   │   ├── inventory/
│   │   ├── treasury/
│   │   └── store/
│   │
│   ├── pages/
│   │   ├── public/
│   │   ├── auth/
│   │   ├── camps/
│   │   ├── coordination/
│   │   ├── treasury/
│   │   └── store/
│   │
│   ├── layouts/
│   │   ├── PublicLayout/
│   │   ├── DashboardLayout/
│   │   └── AdminLayout/
│   │
│   ├── routes/
│   │
│   ├── services/
│   │   ├── api/
│   │   ├── auth/
│   │   ├── camps/
│   │   ├── registrations/
│   │   ├── commissions/
│   │   ├── inventory/
│   │   ├── treasury/
│   │   └── store/
│   │
│   ├── hooks/
│   │
│   ├── store/
│   │
│   ├── context/
│   │
│   ├── types/
│   │
│   ├── utils/
│   │
│   ├── styles/
│   │
│   ├── App.tsx
│   └── main.tsx
│
├── tests/
├── .env.example
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

La estructura puede evolucionar conforme aumente el tamaño del proyecto, pero las responsabilidades deben mantenerse separadas.

---

# 6. Responsabilidades

## Components

Elementos visuales reutilizables.

Ejemplos:

* Button
* Input
* Modal
* Table
* Card
* FormField
* LoadingSpinner
* ErrorMessage
* Pagination
* CampCard

Los componentes no deben contener lógica de negocio compleja.

## Pages

Representan pantallas completas.

Ejemplos:

```text
HomePage
LoginPage
RegisterPage
CampListPage
CampDetailsPage
RegistrationPage
TreasuryDashboardPage
StorePage
```

## Services

Centralizan la comunicación con el backend.

Ejemplo:

```text
services/camps/
services/registrations/
services/payments/
services/store/
```

## Hooks

Contienen lógica reutilizable relacionada con React.

Ejemplos:

```text
useAuth()
useCurrentUser()
useCamp()
useRegistration()
usePermissions()
```

## Utils

Funciones puras y auxiliares.

No deben contener lógica específica de una pantalla.

---

# 7. Cliente HTTP

Toda comunicación con el backend debe pasar por una instancia centralizada de Axios.

Ejemplo conceptual:

```typescript
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  withCredentials: true,
});
```

## Request interceptor

El cliente HTTP podrá:

* Generar o propagar `X-Request-ID`.
* Agregar headers requeridos.
* Configurar credenciales.
* Normalizar determinadas solicitudes.

El JWT no será leído directamente desde `localStorage` ni `sessionStorage`.

---

## Response interceptor

El interceptor deberá manejar de forma centralizada:

### 401 Unauthorized

Flujo:

```text
Request
   |
   v
401
   |
   v
Intentar renovar sesión
   |
   +-- Éxito --> repetir request
   |
   +-- Error --> limpiar sesión --> Login
```

Debe evitarse un loop infinito de renovación.

---

### 403 Forbidden

No implica logout.

Debe mostrarse una respuesta adecuada indicando que el usuario no tiene autorización para realizar la operación.

---

### 429 Too Many Requests

La aplicación deberá:

1. Leer `Retry-After` si el backend lo proporciona.
2. Esperar el tiempo indicado.
3. Reintentar únicamente operaciones seguras.
4. Evitar reintentos infinitos.

---

### 5xx

Se podrán realizar reintentos limitados para operaciones idempotentes.

Ejemplos:

```text
GET /camps
GET /users/me
GET /notifications
```

No se deberán reintentar automáticamente operaciones críticas como:

```text
POST /payments
POST /orders
POST /registrations
```

Esto evita duplicar operaciones financieras o registros.

---

## Timeout

Las solicitudes tendrán un timeout definido.

Un timeout debe convertirse en un error controlado de interfaz:

```text
La solicitud tardó demasiado.
Comprueba tu conexión e inténtalo nuevamente.
```

---

# 8. Autenticación

La autenticación utilizará sesiones basadas en cookies seguras administradas por el backend.

## Estrategia

El backend establecerá las cookies utilizando atributos apropiados:

```text
HttpOnly
Secure
SameSite
```

El frontend no debe tener acceso directo al JWT.

Esto evita almacenar tokens de autenticación en:

```text
localStorage
sessionStorage
```

La aplicación utilizará:

```typescript
withCredentials: true
```

cuando corresponda.

---

# 9. Flujo de autenticación

```text
Login
  |
  v
POST /auth/login
  |
  v
Backend valida credenciales
  |
  v
Backend establece cookie
  |
  v
Frontend obtiene sesión
  |
  v
GET /users/me
  |
  v
Auth State
  |
  v
Protected Routes
```

Al recargar la página:

```text
Application Start
       |
       v
GET /users/me
       |
   +---+---+
   |       |
Success   401
   |       |
 Auth     Guest
```

No se debe asumir que el usuario está autenticado únicamente porque existe estado local.

---

# 10. Roles y permisos

Roles definidos por el sistema:

```text
INTEGRANTE
PARTICIPANTE
DIRECTOR_CAMPA
TESORERO
COORDINADOR
ASESOR_LAICO
ASESOR_RELIGIOSO
```

El frontend puede ocultar o mostrar funcionalidades dependiendo del rol.

Sin embargo:

> Los permisos del frontend son exclusivamente una medida de UX. El backend es la única autoridad para autorizar operaciones.

Ejemplo:

```text
Frontend:
¿Mostrar botón "Crear campamento"?

Backend:
¿El usuario realmente puede crear campamentos?
```

El backend siempre debe validar la segunda pregunta.

---

# 11. Routing y rutas protegidas

Las rutas se dividirán en:

## Públicas

```text
/
 /camps
 /camps/:id
 /store
 /login
 /register
```

## Autenticadas

```text
/profile
/my-camps
/my-registrations
/notifications
```

## Director de Campa / Coordinación

```text
/dashboard
/camps/manage
/camps/:id/registrations
/camps/:id/commissions
/camps/:id/inventory
/camps/:id/tribes
```

## Tesorería

```text
/treasury
/treasury/payments
/treasury/expenses
/treasury/reports
```

Las rutas protegidas deben validar el estado de sesión antes de mostrar contenido privado.

---

# 12. State Management

Se utilizará una arquitectura híbrida.

## Context API

Se utilizará para estados globales pequeños y transversales.

Ejemplos:

```text
Auth
Theme
Configuración global
```

## Zustand

Se utilizará cuando exista estado global complejo que no sea apropiado para Context API.

Ejemplos:

```text
Carrito de tienda
Preferencias persistentes
Estado global complejo de UI
Filtros compartidos
```

No se utilizará Zustand simplemente por defecto.

---

## TanStack Query

Los datos provenientes del backend serán considerados estado remoto.

Se utilizará TanStack Query para:

* Caché.
* Refetch.
* Invalidación.
* Estados loading/error.
* Sincronización.
* Deduplicación de solicitudes.

Ejemplo conceptual:

```text
Component
    |
    v
useQuery()
    |
    v
TanStack Query
    |
    +-- Cache
    |
    v
API
```

Esto evita implementar manualmente sistemas de caché dentro de cada componente.

---

# 13. Estrategia de caché

No todas las entidades tendrán la misma duración de caché.

Ejemplo inicial:

| Recurso              | Estrategia             |
| -------------------- | ---------------------- |
| Campamentos públicos | Caché corto            |
| Información pública  | Caché corto/medio      |
| Perfil               | Caché con invalidación |
| Inscripciones        | Caché corto            |
| Inventario           | Caché muy corto        |
| Pagos                | Sin caché prolongado   |
| Finanzas             | Sin caché prolongado   |
| Carrito              | Estado local           |
| Notificaciones       | Refetch periódico      |

Después de una mutación importante se invalidarán las queries relacionadas.

Ejemplo:

```text
Crear campamento
      |
      v
POST /camps
      |
      v
invalidateQueries(["camps"])
```

---

# 14. Formularios dinámicos

Los campamentos podrán definir preguntas personalizadas.

Tipos soportados:

```text
text
textarea
number
date
select
radio
checkbox
boolean
file
```

Ejemplo:

```text
¿Tienes alguna alergia?
        |
     Sí / No
        |
       Sí
        |
        v
¿Qué alergia tienes?
```

El sistema deberá soportar:

* Validación requerida.
* Validación por tipo.
* Validación condicional.
* Dependencias entre preguntas.
* Valores mínimos y máximos.
* Longitud mínima y máxima.
* Opciones dinámicas.
* Mostrar/ocultar campos.
* Guardado de borradores.
* Recuperación de formularios.
* Indicadores de progreso.

---

# 15. Borradores de formularios

Cuando sea apropiado, los formularios largos podrán guardarse como borrador.

Ejemplo:

```text
Registro
   |
   +-- Datos personales
   |
   +-- Información médica
   |
   +-- Sacramentos
   |
   +-- Talla
   |
   +-- Confirmación
```

El usuario podrá abandonar temporalmente el formulario y continuar posteriormente.

Los datos sensibles no deben almacenarse innecesariamente en almacenamiento persistente del navegador.

Los borradores que contengan información sensible deberán preferentemente almacenarse en backend mediante un mecanismo autenticado.

---

# 16. Sesiones expiradas

Si la sesión expira mientras el usuario está utilizando un formulario:

```text
Sesión expirada
      |
      v
Intentar renovación
      |
   +--+--+
   |     |
Éxito   Error
   |     |
Continuar  Login
```

Cuando sea posible, el frontend deberá preservar el estado no sensible del formulario para evitar pérdida de trabajo.

---

# 17. Campamentos

El módulo de campamentos permitirá:

* Listar campamentos.
* Ver detalles.
* Consultar fecha.
* Consultar ubicación.
* Consultar capacidad.
* Consultar costo.
* Consultar estado.
* Consultar disponibilidad.
* Registrarse.
* Consultar información relacionada.

Estados esperados:

```text
DRAFT
OPEN
CLOSED
IN_PROGRESS
FINISHED
CANCELLED
```

La transición real de estados será responsabilidad del backend.

---

# 18. Inscripciones

El frontend permitirá:

```text
Consultar campamento
       |
       v
Iniciar registro
       |
       v
Completar formulario
       |
       v
Validación
       |
       v
Enviar registro
       |
       v
Backend
       |
       v
Confirmación
```

Se deberán mostrar estados como:

```text
Pendiente
Confirmada
Cancelada
Lista de espera
Pago pendiente
Pagada
Beca
```

La condición real de inscripción será determinada por el backend.

---

# 19. Comisiones

El sistema contemplará las comisiones:

```text
Cocina
Construcción
Intercesión
Veladas
Líderes
```

El frontend deberá permitir interfaces diferenciadas según los permisos del usuario.

Un servidor asignado a una comisión podrá visualizar la información correspondiente a su comisión.

El Director de Campa podrá administrar las asignaciones.

---

# 20. Inventarios

Las interfaces de inventario permitirán:

* Consultar artículos.
* Agregar artículos.
* Modificar cantidades.
* Consultar costos.
* Consultar estado.
* Registrar responsable.
* Consultar movimientos.

Ejemplo:

```text
Material
Cantidad
Unidad
Costo
Estado
Responsable
```

El frontend nunca calculará por sí solo valores financieros definitivos.

---

# 21. Tesorería

El módulo de tesorería podrá mostrar:

* Campamentos.
* Pagos realizados.
* Pagos pendientes.
* Becas.
* Ventas de tienda.
* Gastos.
* Ingresos.
* Balance.
* Historial de transacciones.
* Reportes.
* Gráficas.
* Exportación.

Los datos financieros deberán considerarse información sensible y no deben almacenarse permanentemente en el navegador.

---

# 22. Pagos

Los pagos se procesarán mediante el backend y el proveedor de pagos correspondiente.

El frontend:

* No almacenará números de tarjeta.
* No almacenará CVV.
* No procesará directamente credenciales bancarias.
* No determinará si un pago está confirmado.
* No considerará un pago exitoso únicamente por una respuesta visual.

Flujo:

```text
Frontend
   |
   v
Backend
   |
   v
Proveedor de pago
   |
   v
Webhook / confirmación
   |
   v
Backend
   |
   v
Frontend consulta estado
```

Los pagos no deberán tener retry automático indiscriminado.

---

# 23. Tienda

El frontend incluirá:

* Catálogo.
* Detalle de producto.
* Variantes.
* Tallas.
* Personalización.
* Carrito.
* Checkout.
* Confirmación.
* Historial de pedidos.

Ejemplo de flujo:

```text
Producto
   |
   v
Agregar al carrito
   |
   v
Carrito
   |
   v
Checkout
   |
   v
Backend
   |
   v
Pago
   |
   v
Confirmación
```

El inventario definitivo será controlado por backend.

---

# 24. Manejo de estados de interfaz

Toda pantalla que consuma datos remotos debe contemplar como mínimo:

```text
Loading
Success
Empty
Error
```

Ejemplo:

```text
Loading:
Cargando campamentos...

Empty:
No hay campamentos disponibles.

Error:
No fue posible cargar los campamentos.

Success:
Lista de campamentos.
```

Nunca se debe dejar una pantalla completamente vacía ante un error.

---

# 25. Manejo de errores

Los errores provenientes del backend se mapearán a comportamientos de interfaz.

| HTTP | Significado            | Frontend                  |
| ---- | ---------------------- | ------------------------- |
| 400  | Solicitud inválida     | Mensaje de validación     |
| 401  | No autenticado         | Renovación/login          |
| 403  | Sin permisos           | Acceso denegado           |
| 404  | No encontrado          | Recurso inexistente       |
| 409  | Conflicto              | Explicar conflicto        |
| 422  | Validación             | Mostrar errores de campos |
| 429  | Rate limit             | Esperar/reintentar        |
| 500  | Error servidor         | Error genérico            |
| 503  | Servicio no disponible | Reintento controlado      |

También deben contemplarse:

```text
Network Error
Timeout
DNS failure
Offline
Respuesta corrupta
Servidor temporalmente no disponible
```

---

# 26. Offline

El MVP será **online-first**.

No se implementará inicialmente una sincronización offline completa debido a la complejidad adicional y al carácter sensible de los datos.

Sí se implementará:

* Detección de pérdida de conexión.
* Mensaje de estado offline.
* Manejo de solicitudes fallidas.
* Retry controlado.
* Recuperación al volver la conexión.

Una futura versión podrá incorporar capacidades offline específicas.

---

# 27. Integración con OpenAPI

El backend FastAPI expone un esquema OpenAPI.

El frontend no deberá copiar manualmente las interfaces del backend.

La fuente oficial del contrato será:

```text
OpenAPI
   |
   v
Generador de TypeScript
   |
   v
Tipos / cliente
   |
   v
Frontend
```

Se utilizará generación automática de tipos a partir del esquema OpenAPI.

Proceso esperado:

```text
Backend modifica API
        |
        v
OpenAPI actualizado
        |
        v
Script de generación
        |
        v
Tipos TypeScript actualizados
        |
        v
TypeScript detecta incompatibilidades
```

Esto reduce errores causados por contratos desactualizados.

La implementación concreta del generador se establecerá durante la configuración inicial del repositorio.

---

# 28. Convención de nombres

## TypeScript

Variables y funciones:

```typescript
camelCase
```

Ejemplo:

```typescript
const campStartDate = ...
```

## Componentes

```typescript
PascalCase
```

Ejemplo:

```text
CampCard.tsx
RegistrationForm.tsx
```

## Tipos e interfaces

```typescript
PascalCase
```

Ejemplo:

```typescript
type CampStatus = ...
interface Camp ...
```

## Archivos

Los componentes React utilizarán:

```text
PascalCase.tsx
```

Los hooks:

```text
useSomething.ts
```

Los utilities podrán utilizar:

```text
camelCase.ts
```

El formato de datos de la API seguirá estrictamente el contrato OpenAPI. Las conversiones entre `snake_case` y `camelCase`, si fueran necesarias, se realizarán en una capa definida y no de forma arbitraria dentro de componentes.

---

# 29. Validación

La validación tendrá dos niveles.

## Frontend

Objetivo:

* Mejorar UX.
* Detectar errores antes de enviar.
* Mostrar mensajes claros.
* Evitar solicitudes innecesarias.

## Backend

Objetivo:

* Seguridad.
* Integridad.
* Reglas de negocio.

Nunca se considerará suficiente una validación frontend.

---

# 30. Seguridad del frontend

Reglas obligatorias:

* No almacenar secretos en el código.
* No subir `.env`.
* No almacenar JWT en `localStorage`.
* No almacenar JWT en `sessionStorage`.
* Utilizar HTTPS en producción.
* No confiar en roles enviados por el cliente.
* No confiar en precios calculados por el cliente.
* No confiar en cantidades de inventario.
* No confiar en estados de pago.
* No utilizar `dangerouslySetInnerHTML` salvo necesidad justificada y sanitización.
* Validar archivos antes de enviarlos.
* Evitar exposición innecesaria de información sensible.
* No registrar información médica o financiera en consola.
* No incluir tokens en logs.

El frontend es considerado un entorno manipulable.

---

# 31. Variables de entorno

Archivo:

```text
.env.example
```

Ejemplo:

```env
VITE_API_URL=http://localhost:8000/api/v1
```

En producción:

```env
VITE_API_URL=https://api.kairoslapaz.com/api/v1
```

Las variables `VITE_*` son visibles en el bundle final.

Por lo tanto:

> Nunca deben contener secretos.

---

# 32. Desarrollo local

Requisitos:

```text
Node.js 18+
npm
Git
```

Instalación:

```bash
npm install
```

Ejecutar:

```bash
npm run dev
```

Frontend:

```text
http://localhost:3000
```

El backend debe estar disponible en:

```text
http://localhost:8000
```

Documentación de API:

```text
http://localhost:8000/docs
```

---

# 33. Scripts

El proyecto deberá contar como mínimo con:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "...",
    "typecheck": "...",
    "test": "...",
    "test:coverage": "...",
    "test:e2e": "...",
    "generate:api": "..."
  }
}
```

Los comandos exactos dependerán de las herramientas instaladas.

---

# 34. Testing

Se utilizarán varios niveles de pruebas.

## Unit tests

Para:

* Utilities.
* Validaciones.
* Funciones puras.
* Transformaciones.

## Component tests

Para:

* Formularios.
* Botones.
* Modales.
* Tablas.
* Estados visuales.

## Integration tests

Para comprobar interacción entre:

```text
Component
+
Hook
+
State
+
API mock
```

## E2E

Para flujos completos:

```text
Registro
Login
Registro a campamento
Pago
Compra
Administración
```

La cobertura objetivo inicial será:

```text
>= 80%
```

con prioridad sobre funcionalidades críticas.

---

# 35. CI/CD

El repositorio deberá utilizar GitHub Actions.

Flujo mínimo:

```text
Push / Pull Request
        |
        v
Install dependencies
        |
        v
Lint
        |
        v
Typecheck
        |
        v
Unit tests
        |
        v
Build
        |
        v
E2E
```

Una Pull Request no deberá fusionarse si falla una comprobación crítica del pipeline.

---

# 36. Despliegue

Arquitectura prevista:

```text
Internet
   |
   v
HTTPS
   |
   v
Nginx / CDN
   |
   v
Frontend estático
```

El frontend generado por Vite podrá desplegarse como contenido estático.

El backend se desplegará independientemente.

```text
Frontend
   |
   | HTTPS
   v
API
   |
   v
PostgreSQL
```

---

# 37. Performance

El frontend deberá utilizar:

* Lazy loading.
* Code splitting.
* Optimización de imágenes.
* Compresión.
* Caché.
* Evitar renders innecesarios.
* Paginación.
* Virtualización cuando existan listas grandes.
* Importaciones selectivas.

Las interfaces administrativas deben poder manejar miles de registros mediante paginación y filtros.

No se deberá cargar toda una tabla de 10,000 registros al navegador.

---

# 38. Bundle size

Se establecerá un presupuesto de rendimiento para producción.

Como referencia inicial:

```text
Objetivo: mantener el bundle inicial lo más pequeño posible.
```

El objetivo de `<200 KB gzipped` podrá utilizarse como referencia para el bundle inicial de la aplicación pública, pero no será una restricción absoluta para módulos administrativos complejos.

Se analizarán:

```text
dist/
bundle analyzer
Lighthouse
Core Web Vitals
```

antes de optimizar prematuramente.

---

# 39. Accesibilidad

La interfaz deberá considerar:

* Navegación mediante teclado.
* Labels asociados a inputs.
* Contraste adecuado.
* Estados de focus.
* Semántica HTML.
* Textos alternativos.
* Mensajes de error accesibles.
* Uso correcto de botones y enlaces.
* Compatibilidad razonable con lectores de pantalla.

---

# 40. Responsive Design

El diseño deberá funcionar en:

```text
Mobile
Tablet
Desktop
```

Especial atención:

* Formularios de inscripción.
* Panel administrativo.
* Tablas.
* Checkout.
* Inventarios.
* Dashboard financiero.

Las tablas administrativas podrán adaptar su presentación para pantallas pequeñas.

---

# 41. SEO

Las páginas públicas deberán considerar:

* Títulos adecuados.
* Meta descriptions.
* URLs legibles.
* Open Graph.
* Imágenes sociales.
* Estructura semántica.
* Rendimiento.

Las áreas privadas del sistema no requieren una estrategia SEO significativa.

---

# 42. Notificaciones

El frontend podrá mostrar:

```text
Registro confirmado
Pago confirmado
Pago pendiente
Asignación a comisión
Cambio de campamento
Recordatorio
Pedido confirmado
```

Las notificaciones provenientes del backend deberán considerarse la fuente oficial.

El frontend puede presentar:

```text
Toast
Banner
Notification Center
Email status
```

dependiendo del caso.

---

# 43. Monitoreo

La aplicación deberá estar preparada para integrar un sistema de error tracking, por ejemplo:

```text
Sentry
```

El monitoreo podrá registrar:

* Errores JavaScript.
* Errores de API.
* Stack traces.
* Navegador.
* Versión del frontend.
* Request ID.

No deberán enviarse:

* Contraseñas.
* Tokens.
* Información médica.
* Datos bancarios.
* Información sensible innecesaria.

La activación completa del monitoreo se realizará para el ambiente de producción.

---

# 44. Analytics

La implementación de analytics no será prioritaria durante el desarrollo inicial.

Si posteriormente se incorpora, deberá limitarse a métricas necesarias como:

```text
Visitas
Navegación
Conversión de registro
Uso de tienda
Errores
```

No se deberán recolectar datos personales innecesarios.

---

# 45. Comunicación con Backend

Backend:

```text
FastAPI
```

Frontend:

```text
React + TypeScript
```

API:

```text
REST
```

Versión:

```text
/api/v1/
```

Ejemplos:

```text
GET    /api/v1/camps
GET    /api/v1/camps/{id}
POST   /api/v1/camps
PATCH  /api/v1/camps/{id}
DELETE /api/v1/camps/{id}
```

Los endpoints reales deberán mantenerse sincronizados mediante OpenAPI.

---

# 46. Paginación

Las listas grandes utilizarán paginación.

Ejemplo:

```text
GET /camps?page=1&page_size=20
```

La interfaz deberá mostrar:

```text
Página actual
Total de resultados
Página anterior
Página siguiente
```

También se podrán implementar:

```text
Filtros
Búsqueda
Ordenamiento
```

siempre que el backend los soporte.

---

# 47. Filtros y búsquedas

Los filtros deberán enviarse al backend cuando el volumen de datos sea elevado.

Ejemplo:

```text
/camps?status=OPEN&page=1&page_size=20
```

No se deberá descargar toda la base de datos para filtrar localmente.

---

# 48. Archivos

Los archivos subidos por usuarios deberán:

* Validar extensión.
* Validar MIME type.
* Validar tamaño.
* Mostrar progreso cuando sea necesario.
* Manejar errores.
* No asumir que la extensión garantiza seguridad.

La validación definitiva será realizada por backend.

---

# 49. Gestión de datos sensibles

El sistema puede manejar información potencialmente sensible relacionada con:

* Salud.
* Alergias.
* Medicamentos.
* Información personal.
* Pagos.
* Finanzas.

El frontend deberá minimizar su almacenamiento local.

Especialmente:

```text
No guardar información médica en localStorage.
No guardar información financiera innecesariamente.
No imprimir información sensible en console.log.
```

El acceso a esta información estará controlado por permisos del backend.

---

# 50. Git

Ramas recomendadas:

```text
main
develop
feature/*
fix/*
hotfix/*
```

Ejemplo:

```text
feature/camp-registration
feature/store-cart
fix/login-session
```

Commits:

```text
feat: add camp registration form
fix: handle expired session
refactor: improve API client
test: add registration tests
docs: update frontend architecture
```

Se recomienda utilizar Conventional Commits.

---

# 51. Pull Requests

Toda funcionalidad importante deberá:

1. Crear branch.
2. Implementar cambios.
3. Ejecutar lint.
4. Ejecutar typecheck.
5. Ejecutar tests.
6. Ejecutar build.
7. Crear Pull Request.
8. Revisar cambios.
9. Corregir observaciones.
10. Fusionar.

---

# 52. Compatibilidad con Backend

El frontend depende directamente del contrato de la API.

Cuando el backend cambie:

```text
Backend change
      |
      v
OpenAPI
      |
      v
Generate TypeScript
      |
      v
Type errors
      |
      v
Update frontend
      |
      v
Tests
```

Los cambios incompatibles deberán documentarse antes de implementarse.

---

# 53. Ambiente de producción

Configuración prevista:

```text
Frontend:
https://kairoslapaz.com

API:
https://api.kairoslapaz.com
```

Los dominios definitivos dependerán de la infraestructura final.

La comunicación deberá utilizar HTTPS.

---

# 54. Roadmap

## v0.1 - Base del proyecto

* Configuración React.
* TypeScript.
* Vite.
* Routing.
* Axios.
* OpenAPI.
* Testing.
* CI.
* Estructura base.

## v0.2 - Autenticación

* Login.
* Registro.
* Sesiones.
* Cookies seguras.
* Perfil.
* Rutas protegidas.
* Roles.

## v0.3 - Campamentos

* Listado.
* Detalle.
* Creación.
* Edición.
* Estados.
* Capacidad.
* Registro.

## v0.4 - Coordinación

* Comisiones.
* Inventarios.
* Tribu.
* Líderes.
* Formularios dinámicos.
* Panel administrativo.

## v0.5 - Finanzas

* Pagos.
* Confirmaciones.
* Gastos.
* Ingresos.
* Reportes.
* Dashboard.

## v0.6 - Tienda

* Catálogo.
* Productos.
* Carrito.
* Checkout.
* Pedidos.
* Inventario.

## v1.0 - MVP

Sistema integrado funcional con:

* Usuarios.
* Campamentos.
* Inscripciones.
* Formularios.
* Comisiones.
* Inventarios.
* Tesorería.
* Pagos.
* Tienda básica.
* Notificaciones.
* Seguridad.
* Testing.
* CI/CD.

## v1.1

* Mejoras de rendimiento.
* Mejoras UX.
* Analytics.
* Monitoreo.
* Exportaciones adicionales.
* Integraciones adicionales.

## v2.0

Posibles funcionalidades:

* Aplicación móvil.
* Capacidades offline.
* Integración con WhatsApp.
* Chatbot.
* Herramientas de IA.
* Funciones avanzadas para otras comunidades.

---

# 55. Arquitectura de seguridad

Modelo:

```text
Frontend
  |
  | Solicitud autenticada
  v
FastAPI
  |
  +-- Authentication
  |
  +-- Authorization
  |
  +-- Business Validation
  |
  v
PostgreSQL
```

El frontend nunca sustituye:

```text
Authentication
Authorization
Business Validation
Data Integrity
Payment Verification
```

Estas responsabilidades pertenecen al backend.

---

# 56. Criterios de calidad

Antes de considerar terminada una funcionalidad:

```text
[ ] TypeScript sin errores
[ ] Lint sin errores críticos
[ ] Tests implementados
[ ] Estados loading/error/empty
[ ] Responsive
[ ] Accesibilidad básica
[ ] Manejo de errores HTTP
[ ] Sin secretos
[ ] Sin información sensible en logs
[ ] Integración con API validada
[ ] Build exitoso
```

Para funcionalidades críticas:

```text
[ ] Integration tests
[ ] E2E test
[ ] Revisión de permisos
[ ] Manejo de sesión expirada
[ ] Manejo de errores de red
```

---

# 57. Estado actual

```text
Estado: En desarrollo
Versión objetivo: 1.0.0
Arquitectura: Definida
API: FastAPI
Base de datos: PostgreSQL
```

La arquitectura descrita representa la arquitectura objetivo del frontend. Algunas funcionalidades podrán permanecer pendientes hasta completar las correspondientes implementaciones del backend.

---

# 58. Repositorios relacionados

Backend:

```text
https://github.com/KAIROS-LAPAZ/kairos-backend
```

Frontend:

```text
https://github.com/KAIROS-LAPAZ/kairos-frontend
```

Documentación:

```text
https://github.com/KAIROS-LAPAZ/kairos-docs
```

---

# 59. Licencia

Pendiente de definir.

---

# 60. Mantenimiento

El frontend deberá mantenerse sincronizado con:

* API.
* OpenAPI.
* Reglas de negocio.
* Roles y permisos.
* Diseño visual.
* Documentación técnica.

Cualquier cambio importante de arquitectura deberá documentarse antes de incorporarse al proyecto.

---

## Kairós La Paz

Sistema Digital Kairós La Paz.

Arquitectura orientada a escalabilidad, seguridad, mantenibilidad y evolución progresiva.
