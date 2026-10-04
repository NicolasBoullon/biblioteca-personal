# CLAUDE.md: Biblioteca personal

Contexto del proyecto para Claude Code. Los requerimientos completos están en `docs/requerimientos.md`; este archivo resume el contexto, las decisiones tomadas y cómo quiero trabajar.

## Cómo quiero que me ayudes (importante)

- **El código lo escribo yo.** Es un proyecto para aprender y para el CV. No generes implementaciones completas, archivos enteros ni features terminadas, salvo que te lo pida explícitamente.
- Tu rol es de **mentor y revisor**: explicar conceptos, señalar errores, revisar mi código, sugerir mejoras, responder dudas y orientarme sobre qué hacer después.
- Si un ejemplo ayuda a explicar algo, que sea corto e ilustrativo (unas pocas líneas), no el código final de mi proyecto.
- Antes de modificar archivos del repo, preguntame.
- Si algo de lo que hago contradice este documento o los requerimientos, avisame.
- Las APIs de Angular y NgRx cambian rápido: basate en la versión instalada (revisá `package.json`) y en la documentación oficial, no en ejemplos viejos.
- Respondeme en español.

## Qué es el proyecto

Aplicación web para organizar mis libros (los que tengo, estoy leyendo o quiero leer), agrupados por autor. Es un ABM con relaciones, filtros, paginación, reglas de negocio y exportación/importación. Repo público en GitHub para el portfolio y el CV: chico pero bien hecho.

Fuera de alcance: autenticación, multiusuario, imágenes y APIs externas (Open Library queda como extra opcional).

## Estructura del repositorio

Monorepo:

```
biblioteca-personal/
├── biblioteca-personal-front/   # Angular
├── biblioteca-personal-back/    # NestJS
├── docs/                        # requerimientos.md y screenshots
├── CLAUDE.md
├── README.md
└── LICENSE (MIT)
```

Hay un único repositorio git en la raíz. Los subproyectos no deben tener su propio `.git` (ya se resolvió una vez el problema de repos anidados creados por `ng new` / `nest new`).

## Stack

**Frontend (Angular 22)**

- Standalone, **zoneless** (sin `zone.js`), OnPush (por defecto en v22).
- Estado: **@ngrx/signals** (`signalStore`, `withState`, `withComputed`, `withMethods`, `withEntities`, `withHooks`, `withProps`, `rxMethod`) en la misma versión mayor que Angular. `@ngrx/operators` para `tapResponse`.
- Lecturas: **Resource API** (`rxResource` con `params` / `stream`, `httpResource`), estable desde v22.
- Formularios: **Signal Forms** (`@angular/forms/signals`), estables desde v22.
- Lazy loading: rutas con `loadComponent` y servicios con **`injectAsync()`** (requiere un servicio auto-provisto: `providedIn: 'root'` o `@Service()`), con prefetch opcional `onIdle`.
- `debounced()` de Angular para hacer debounce de signals.
- Control flow nuevo, `input()` / `output()`, `inject()`.
- Estilos: **Tailwind CSS v4** (ya instalado, vía `@tailwindcss/postcss`). Tokens con `@theme` en `styles.css`, tema oscuro con `@custom-variant dark` basado en una clase en `<html>`. Utilidades en los templates, sin valores arbitrarios sueltos y `@apply` con moderación. **Sin librerías de componentes.**

**Backend (NestJS)**

- TypeScript estricto, **SQLite**, ORM a elección (TypeORM o Prisma).
- Validación con `class-validator` / `class-transformer` y `ValidationPipe` global.
- Swagger. Prefijo global `/api`.

**Calidad**

- ESLint y Prettier, tests (reglas de negocio en el back y store en el front), GitHub Actions con lint, test y build.

## Dominio

Entre paréntesis, el nombre en código (ver "Idioma" en Convenciones).

**Autor (`Author`):** nombre (`name`, obligatorio, único sin distinguir mayúsculas, máx. 120) y nacionalidad opcional (`nationality`).
**Libro (`Book`):** título (`title`, obligatorio, máx. 200), autor (`authorId`, obligatorio), año opcional (`publicationYear`, no mayor al actual), estado (`status`: `PENDING` / `READING` / `READ`, por defecto `PENDING`), puntuación 1–5 opcional (`rating`), fecha de fin de lectura (`finishedAt`), notas (`notes`, máx. 1000).

**Reglas de negocio (validadas en el backend, no solo en el front):**

- Solo se puede puntuar un libro en estado `READ`.
- La fecha de fin de lectura la gestiona el servidor: se completa al pasar a `READ` y se borra al salir de ese estado.
- No se puede borrar un autor con libros asociados (409, indicando cuántos tiene).
- No puede haber dos autores con el mismo nombre (409).

## Funcionalidades clave

- ABM de autores y libros. Cambio rápido de estado desde el listado.
- Listado paginado de libros con búsqueda (título o autor, con debounce), filtros por estado y autor, y vuelta a la página 1 al cambiar un filtro.
- Filtros, búsqueda y página **sincronizados con la URL**.
- Resumen de toda la biblioteca (`GET /api/books/stats`).
- Exportar e importar la biblioteca en JSON.

## Decisiones de arquitectura del front

- **Lecturas con resources, escrituras con el store.** Listado, detalle y resumen son resources reactivos a signals. Crear, editar, borrar y cambiar estado pasan por métodos del store (`rxMethod`), que después refrescan lo afectado.
- Mientras se recarga, se siguen mostrando los datos anteriores con un indicador sutil (sin pantalla en blanco). Los pedidos viejos se cancelan.
- Un store por feature. Al menos uno usa `withEntities`.
- Los valores derivados se calculan con `computed`, nunca se duplican en el estado.
- Ningún componente usa `HttpClient` directo: todo pasa por servicios de API tipados.
- Manejo de errores HTTP centralizado (interceptor o helper).
- La búsqueda con debounce se prueba de dos formas (`debounced()` y RxJS en `rxResource`), se elige una y se justifica en el README.
- El servicio de exportar/importar se carga con `injectAsync()` para sacarlo del bundle inicial.

## Convenciones

- **Idioma:** todo el código en inglés (archivos, componentes, selectores, clases, variables, rutas del front, endpoints, entidades, campos y enums). Solo lo que ve el usuario (textos de la UI y mensajes de error) va en español. Ejemplo: ruta `/books`, componente `BooksPage`, enum `READ`, label "Leído".
- **Estructura por feature** (guía de estilo oficial de Angular), lo más plana posible. Todo el código va dentro de `src/`; fuera de `src/` solo hay configuración.
  ```
  biblioteca-personal-front/src/
  ├── index.html, main.ts
  ├── styles.css               # Tailwind: @import, @theme (tokens), dark variant
  └── app/
      ├── core/                # helpers transversales, interceptor, layout
      ├── shared/              # UI reutilizable
      ├── features/
      │   ├── books/
      │   │   ├── pages/
      │   │   ├── components/
      │   │   ├── books.store.ts
      │   │   ├── books.api.ts
      │   │   └── book.model.ts
      │   └── authors/         # misma estructura
      ├── app.ts
      ├── app.config.ts
      └── app.routes.ts
  ```
- El state no tiene carpeta propia: cada store vive dentro de su feature. Un helper usado por una sola feature va en esa feature; si es transversal, en `core/`.
- Nombres de archivo sin sufijos de tipo (convención v20+): `books-page.ts`, no `books-page.component.ts`.
- **Modelos del front con `interface`** (los datos de la API son objetos planos). `type` para uniones y tipos derivados (`Omit`, `Pick`, `Partial`). En el back, entidades y DTOs son **clases** (por los decoradores). Esto es una convención de código, no una regla de negocio.
- Estado normalizado: entidades relacionadas por id, sin anidar estructuras en el estado.
- Git: una rama y un PR por feature, issues por hito, Conventional Commits (`feat:`, `fix:`, `refactor:`, `chore:`...).

## Hitos

1. Backend base: Nest, SQLite, CRUD, validaciones y Swagger.
2. Filtros, paginación, reglas de negocio, `/books/stats` y seed (~10 autores, ~30 libros).
3. Frontend base: zoneless, Tailwind con tokens y tema, layout y rutas.
4. ABM de autores con su store.
5. ABM de libros: listado con resource y filtros, URL sincronizada, Signal Forms, borrado y cambio de estado.
6. Pulido: resumen, exportar/importar lazy, tema oscuro, responsive, accesibilidad y estados vacíos/error.
7. Tests, CI, README final (screenshots, cómo correrlo, decisiones técnicas) y deploy (front en Vercel/Netlify, back en Render/Railway con seed al arrancar).

## Estado actual

- Repo creado y conectado a GitHub. Monorepo con front y back, sin repos anidados.
- `@ngrx/signals` y Tailwind CSS v4 instalados en el front.
- `.gitignore` raíz, `LICENSE` (MIT) creados. Layout en `core/layout` como ruta padre (`children` vacío). Falta mover `pages/books` a `features/books/pages/`.
- README inicial (objetivo y alcance) y `docs/requerimientos.md` cargados.
- Próximo paso: definir si se arranca por el backend (hito 1) o por la base del front (hito 3).
