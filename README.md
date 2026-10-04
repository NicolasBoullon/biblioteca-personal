# 📚 Biblioteca personal

Aplicación web para organizar mis libros: los que tengo, los que estoy leyendo y los que quiero leer, agrupados por autor.

> 🚧 Proyecto en desarrollo.

## Propósito del proyecto

Este proyecto existe para practicar el desarrollo con **Angular moderno** de punta a punta, con un caso de uso simple pero completo: un ABM con relaciones, filtros, paginación y reglas de negocio.

La idea no es construir algo grande, sino algo chico y bien hecho: código prolijo, decisiones técnicas justificadas y un flujo de trabajo como el de un proyecto real (issues, ramas, PRs y CI).

## Qué se busca practicar

**Frontend**

- Angular **zoneless**, sin `zone.js`, con detección de cambios basada en signals.
- Manejo de estado con **NgRx Signals** (`signalStore`), un store por feature.
- Lecturas de datos reactivas con la **Resource API** (`rxResource` / `httpResource`), separando lo que se _lee_ de lo que se _modifica_.
- Búsqueda con debounce, filtros combinables y estado sincronizado con la URL.
- Componentes standalone, control flow nuevo, `input()` / `output()` e `inject()`.
- Estilos con **Tailwind CSS v4**: design tokens con `@theme`, tema claro y oscuro, diseño responsive y accesibilidad básica, sin librerías de componentes.

**Backend**

- API REST con **NestJS**, simple y bien validada.
- Persistencia liviana con **SQLite**, para que el proyecto corra sin instalar nada extra.
- Reglas de negocio validadas en el servidor, no solo en el front.
- Documentación de la API con **Swagger**.

**Calidad y flujo de trabajo**

- TypeScript estricto, lint y formato automático.
- Código en inglés; la interfaz, en español.
- Tests de lo importante: reglas de negocio en el back y store en el front.
- CI con GitHub Actions.

## Objetivos

### Objetivo general

Desarrollar una aplicación web que permita organizar una biblioteca personal: registrar autores y libros, llevar el estado de lectura y la valoración de cada libro, y consultar la información con búsquedas, filtros y un resumen general.

### Objetivos específicos

**Gestión de autores**

- Crear un autor con su nombre y, de forma opcional, su nacionalidad.
- Listar los autores ordenados alfabéticamente, con la cantidad de libros de cada uno.
- Editar el nombre y la nacionalidad de un autor.
- Eliminar un autor que no tenga libros asociados.

**Gestión de libros**

- Crear un libro con título, autor, año de publicación, estado, puntuación y notas.
- Validar los datos del formulario y mostrar los errores en cada campo.
- Editar un libro desde un formulario precargado, con acceso directo desde la URL.
- Eliminar un libro, pidiendo confirmación antes.
- Avisar cuando se intenta editar un libro que no existe y ofrecer volver al listado.

**Seguimiento de lectura**

- Asignar a cada libro un estado: pendiente, leyendo o leído.
- Cambiar el estado de un libro directamente desde el listado.
- Puntuar del 1 al 5 los libros leídos.
- Registrar de forma automática la fecha en que se terminó de leer un libro.
- Agregar notas personales a cada libro.

**Búsqueda y consulta**

- Listar los libros con paginación.
- Buscar libros por título o por autor mientras se escribe.
- Filtrar libros por estado.
- Filtrar libros por autor.
- Combinar la búsqueda y los filtros entre sí.
- Volver a la primera página al cambiar un filtro o la búsqueda.
- Conservar la búsqueda, los filtros y la página en la URL, para poder compartir el enlace o recargar sin perderlos.

**Resumen de la biblioteca**

- Ver la cantidad total de libros.
- Ver cuántos libros hay en cada estado.
- Ver el promedio de puntuación de los libros leídos.

**Exportación e importación**

- Exportar la biblioteca completa a un archivo JSON.
- Importar una biblioteca desde un archivo JSON, con confirmación previa.

**Experiencia de uso**

- Alternar entre tema claro y oscuro, siguiendo por defecto el del sistema y recordando la elección.
- Usar la aplicación cómodamente desde el celular, con el listado en forma de tarjetas.
- Mostrar mensajes claros cuando una lista está vacía o hay un error.
- Navegar la aplicación con el teclado.
- Arrancar con datos de ejemplo para que la aplicación no esté vacía.

### Reglas de negocio

- El nombre del autor es obligatorio y no puede repetirse (sin distinguir mayúsculas).
- El título y el autor del libro son obligatorios.
- El año de publicación no puede ser posterior al actual.
- Un libro nuevo empieza como pendiente.
- Solo se pueden puntuar los libros leídos.
- La fecha de fin de lectura se completa al marcar un libro como leído y se borra si cambia a otro estado.
- No se puede eliminar un autor que tenga libros; el sistema avisa cuántos tiene.
- Si un archivo de importación es inválido, no se modifica ningún dato.

### Fuera de alcance

- Usuarios, inicio de sesión y multiusuario.
- Portadas o imágenes de los libros.
- Integración con servicios externos (Open Library queda como posible extra).

El detalle completo está en [docs/requerimientos.md](docs/requerimientos.md).

## Estructura del repositorio

```
biblioteca-personal/
├── biblioteca-personal-front/   # Aplicación Angular
├── biblioteca-personal-back/    # API NestJS
└── docs/                        # Requerimientos y material del proyecto
```

## Estado

- [ ] Backend base: CRUD, validaciones y Swagger
- [ ] Filtros, paginación, reglas de negocio y seed
- [ ] Frontend base: zoneless, Tailwind, layout y rutas
- [ ] ABM de autores
- [ ] ABM de libros con filtros y URL sincronizada
- [ ] Pulido: resumen, exportar/importar, tema oscuro, responsive y accesibilidad
- [ ] Tests, CI y deploy

## Licencia

[MIT](LICENSE)
