# Biblioteca personal: documento de requerimientos

## 1. Objetivo

Aplicación web para registrar los libros que tengo, estoy leyendo o quiero leer, organizados por autor. Es un proyecto de portfolio: el foco está en aplicar Angular moderno (zoneless, signals, resources), NgRx Signals y una API simple en NestJS, con código prolijo y un README que explique las decisiones.

**Fuera de alcance:** autenticación, multiusuario, subida de imágenes e integración con APIs externas de libros (se deja como extra opcional).

## 2. Stack y restricciones técnicas

### Frontend

- Angular 22, con componentes standalone.
- Aplicación zoneless, sin `zone.js` en el bundle.
- Estado con `@ngrx/signals` (`signalStore`).
- Lecturas de datos con `rxResource` y/o `httpResource`.
- Formularios con Signal Forms (`@angular/forms/signals`).
- Control flow nuevo (`@if`, `@for`, `@switch`), `input()` / `output()`, `inject()`.
- Estilos con Tailwind CSS v4, sin librerías de componentes (nada de Material ni PrimeNG).
- Rutas con lazy loading.

### Backend

- NestJS con TypeScript en modo estricto.
- Base de datos SQLite (que el repo se pueda clonar y correr sin instalar un motor de base de datos).
- ORM a elección (TypeORM o Prisma).
- Validación de entrada en todos los endpoints.
- Documentación de la API con Swagger.

### Repositorio

- Monorepo con dos carpetas: `biblioteca-personal-front/` y `biblioteca-personal-back/`.
- Se tiene que poder levantar todo con, como máximo, dos comandos documentados en el README.

### Idioma

- Todo el código está en inglés: nombres de archivos, componentes, clases, variables, rutas del front, endpoints, entidades, campos y valores de enums.
- Lo que ve el usuario (textos de la interfaz y mensajes de error) está en español.
- En este documento el dominio se describe en español; la columna "En código" indica el nombre que se usa en el código.

## 3. Modelo de datos

### Autor

En código: `Author`.

| Campo        | En código     | Tipo   | Reglas                                                                |
| ------------ | ------------- | ------ | --------------------------------------------------------------------- |
| id           | `id`          | entero | autogenerado                                                          |
| nombre       | `name`        | texto  | obligatorio, máximo 120 caracteres, único (sin distinguir mayúsculas) |
| nacionalidad | `nationality` | texto  | opcional                                                              |

### Libro

En código: `Book`.

| Campo                   | En código                 | Tipo               | Reglas                                                                         |
| ----------------------- | ------------------------- | ------------------ | ------------------------------------------------------------------------------ |
| id                      | `id`                      | entero             | autogenerado                                                                   |
| título                  | `title`                   | texto              | obligatorio, máximo 200 caracteres                                             |
| autor                   | `authorId`                | referencia a Autor | obligatorio                                                                    |
| año de publicación      | `publicationYear`         | entero             | opcional, entre 0 y el año actual                                              |
| estado                  | `status`                  | enum               | `PENDING`, `READING` o `READ`; por defecto `PENDING`                           |
| puntuación              | `rating`                  | entero             | opcional, de 1 a 5; solo se puede cargar si el estado es `READ`                |
| fecha de fin de lectura | `finishedAt`              | fecha              | se completa automáticamente al pasar a `READ` y se borra si sale de ese estado |
| notas                   | `notes`                   | texto              | opcional, máximo 1000 caracteres                                               |
| creado / actualizado    | `createdAt` / `updatedAt` | fecha y hora       | automáticos                                                                    |

En la interfaz, los estados se muestran como "Pendiente", "Leyendo" y "Leído".

**Relación:** un autor tiene muchos libros y un libro tiene un solo autor.

## 4. Requerimientos funcionales

### Autores

- **RF-01** Listar autores ordenados alfabéticamente, mostrando cuántos libros tiene cada uno.
- **RF-02** Crear un autor. Si ya existe uno con el mismo nombre (sin distinguir mayúsculas), se muestra un error claro.
- **RF-03** Editar el nombre y la nacionalidad de un autor.
- **RF-04** Eliminar un autor. Si tiene libros asociados, no se permite y se informa cuántos tiene.

### Libros

- **RF-05** Listar libros en forma paginada (10 por página por defecto), con título, autor, año, estado y puntuación.
- **RF-06** Buscar por texto sobre título y nombre del autor. La búsqueda se dispara sola mientras se escribe, con debounce (unos 300 ms), sin botón de "buscar".
- **RF-07** Filtrar por estado y por autor. Los filtros se combinan con la búsqueda.
- **RF-08** Cambiar un filtro o la búsqueda vuelve a la página 1.
- **RF-09** Los filtros, la búsqueda y la página actual se reflejan en la URL (query params), de modo que recargar la página o compartir el link mantenga la vista.
- **RF-10** Crear un libro desde un formulario con validaciones visibles por campo.
- **RF-11** Editar un libro. El formulario llega precargado y la URL incluye el id.
- **RF-12** Eliminar un libro con confirmación previa.
- **RF-13** Cambiar el estado de un libro directamente desde el listado, sin entrar al formulario.
- **RF-14** Si se abre la edición de un libro que no existe, se muestra un mensaje y un link para volver al listado.

### Resumen

- **RF-15** Mostrar en la pantalla de libros un resumen con la cantidad total, cuántos hay en cada estado y la puntuación promedio de los leídos. Estos datos corresponden a toda la biblioteca, no solo a la página actual, así que requieren su propio endpoint.

### Exportación e importación

- **RF-16** Exportar la biblioteca completa (autores y libros) a un archivo JSON descargable.
- **RF-17** Importar una biblioteca desde un archivo JSON con el mismo formato que la exportación. Antes de importar se pide confirmación, porque reemplaza los datos actuales. Si el archivo es inválido, se informa el error y no se modifica nada.

## 5. Contrato de la API

Todas las rutas van bajo el prefijo `/api`. Las respuestas de error siguen el formato estándar de NestJS (`statusCode`, `message`, `error`).

| Método | Ruta              | Descripción                                    | Respuestas esperadas                  |
| ------ | ----------------- | ---------------------------------------------- | ------------------------------------- |
| GET    | `/authors`        | Lista de autores con cantidad de libros        | 200                                   |
| POST   | `/authors`        | Crear autor                                    | 201, 400, 409 (nombre duplicado)      |
| PATCH  | `/authors/:id`    | Editar autor                                   | 200, 400, 404, 409                    |
| DELETE | `/authors/:id`    | Eliminar autor                                 | 204, 404, 409 (tiene libros)          |
| GET    | `/books`          | Listado paginado con filtros                   | 200, 400                              |
| GET    | `/books/:id`      | Detalle de un libro con su autor               | 200, 404                              |
| POST   | `/books`          | Crear libro                                    | 201, 400, 404 (autor inexistente)     |
| PATCH  | `/books/:id`      | Edición parcial (incluye el cambio de estado)  | 200, 400, 404                         |
| DELETE | `/books/:id`      | Eliminar libro                                 | 204, 404                              |
| GET    | `/books/stats`    | Resumen para RF-15                             | 200                                   |
| GET    | `/library/export` | Biblioteca completa en JSON (RF-16)            | 200                                   |
| POST   | `/library/import` | Reemplaza la biblioteca con el JSON recibido (RF-17) | 201, 400 (archivo inválido)     |

**Query params de `GET /books`:** `search`, `status`, `authorId`, `page` (por defecto 1), `limit` (por defecto 10, máximo 50). Cualquier valor inválido devuelve 400.

**Respuesta paginada:** un objeto con `items`, `total`, `page` y `limit`.

**Reglas de negocio que valida el backend (no solo el front):**

- El nombre del autor no puede repetirse, sin distinguir mayúsculas (409).
- No se puede eliminar un autor con libros asociados (409, indicando cuántos tiene).
- La puntuación solo se acepta si el estado es `READ`.
- La fecha de fin de lectura la gestiona el servidor; el cliente no la envía.
- El año no puede ser mayor al actual.
- La importación es atómica: si falla, no se modifica ningún dato.

## 6. Requerimientos de arquitectura del frontend

Estos puntos existen para practicar las herramientas nuevas. En el README hay que explicar cómo se resolvió cada uno.

- **RA-01** Separar lecturas de escrituras. Las lecturas (listado, detalle, resumen) se modelan como resources que reaccionan a signals. Las escrituras (crear, editar, borrar, cambiar estado) pasan por métodos del store.
- **RA-02** Después de una escritura exitosa, los datos afectados se refrescan sin recargar la página (incluido el resumen de RF-15).
- **RA-03** Mientras se recarga el listado, se siguen mostrando los datos anteriores con un indicador de carga sutil, sin que la pantalla quede en blanco ni "salte".
- **RA-04** Si un pedido de listado queda viejo porque cambió un filtro, se cancela.
- **RA-05** Un store por feature (autores y libros). Al menos uno usa `withEntities`.
- **RA-06** Los valores derivados (totales, cantidad de páginas, "hay filtros activos") se calculan con `computed`, sin guardarse duplicados en el estado.
- **RA-07** La búsqueda con debounce se prueba de dos formas: con `debounced()` de Angular y con RxJS dentro de un `rxResource`. Se elige una y se justifica en el README. Donde sea un GET simple, se evalúa `httpResource` y también se justifica la elección.
- **RA-08** Ningún componente llama a `HttpClient` directamente. Todo pasa por servicios de API tipados.
- **RA-09** Manejo centralizado de errores HTTP: un interceptor o helper que convierta la respuesta de error en un mensaje legible.
- **RA-10** El formulario de libros se implementa con Signal Forms (estables desde Angular 22).
- **RA-11** El servicio de exportar/importar se carga de forma diferida con `injectAsync()`, para sacarlo del bundle inicial.

## 7. Requerimientos no funcionales

### UI y estilos

- **RNF-01** Estilos con Tailwind CSS v4. Los design tokens (colores semánticos, tipografía, radios) se definen una sola vez con `@theme` en `styles.css`, y los componentes los consumen con las utilidades de Tailwind, sin valores arbitrarios sueltos (`bg-[#1e293b]`, `p-[13px]`). `@apply` se usa con moderación: los patrones que se repiten se resuelven con componentes de Angular.
- **RNF-02** Tema claro y oscuro con la variante `dark:` de Tailwind, configurada con `@custom-variant` para que dependa de una clase en `<html>`. Por defecto sigue la preferencia del sistema, con un toggle manual que se recuerda.
- **RNF-03** Responsive desde 360 px de ancho. En mobile, el listado pasa de tabla a tarjetas.
- **RNF-04** Accesibilidad básica: foco visible, labels en todos los campos, errores asociados al campo, navegación completa con teclado y contraste AA.
- **RNF-05** Estados vacíos con una acción clara (por ejemplo, "Todavía no cargaste libros" junto al botón para agregar uno). Los mensajes de error dicen qué pasó y qué hacer.

### Calidad

- **RNF-06** TypeScript estricto en ambos proyectos, sin `any` explícitos.
- **RNF-07** ESLint y Prettier configurados, con lint sin errores.
- **RNF-08** Tests mínimos:
  - Backend: tests unitarios del servicio de libros que cubran las reglas de negocio de la sección 5, y un test e2e de `GET /books` con filtros.
  - Frontend: tests del store de libros (filtros, reseteo de página y refresco después de guardar) y de un componente.
- **RNF-09** GitHub Actions que corra lint, tests y build de ambos proyectos en cada push y PR.

### Datos

- **RNF-10** Script de seed que cargue unos 10 autores y 30 libros de ejemplo, para que la demo no arranque vacía.

## 8. Criterios de aceptación generales

- Clonar el repo, instalar dependencias y seguir el README deja la app funcionando en menos de 5 minutos.
- En el `package.json` del front no figura `zone.js` y la app funciona igual.
- Swagger accesible y con todos los endpoints documentados, incluidos sus DTOs.
- Recargar la página en cualquier pantalla mantiene el estado visible (filtros en la URL y edición por id).
- Exportar la biblioteca y volver a importar ese mismo archivo deja los datos iguales.
- El pipeline de CI pasa en verde.

## 9. README (entregable)

El README es parte del proyecto y tiene que incluir:

- Una línea que describa qué es, un screenshot o GIF y el link a la demo si la hay.
- Los objetivos y funcionalidades del sistema.
- El stack con versiones.
- Cómo correrlo: requisitos, instalación, seed y comandos.
- Decisiones técnicas: cómo se aplicaron RA-01, RA-03, RA-07 y RA-11, y el enfoque de estilos. Es la sección que más va a leer un entrevistador.
- Qué haría distinto o qué sigue.

## 10. Hitos

1. **Backend base:** Nest, SQLite, CRUD de autores y libros, validaciones y Swagger.
2. **Reglas y extras de API:** filtros, paginación, reglas de negocio, `/books/stats` y seed.
3. **Frontend base:** proyecto zoneless, Tailwind con tokens y tema, layout y rutas.
4. **Autores:** ABM de autores con su store.
5. **Libros:** listado con resource y filtros, sincronización con la URL, formulario con Signal Forms, borrado y cambio rápido de estado.
6. **Pulido:** resumen, exportar/importar con carga diferida, estados vacíos y de error, tema oscuro, responsive y accesibilidad.
7. **Entrega:** tests, CI, README y deploy (front en Vercel o Netlify, back en Render o Railway con seed al arrancar).

## 11. Extras opcionales

- Autocompletar datos del libro con la API pública de Open Library.
- Vista "estantería" con los libros agrupados por estado (tipo tablero).
- Docker Compose para levantar todo con un solo comando.
