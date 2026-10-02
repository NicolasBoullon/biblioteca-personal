# 📚 Biblioteca personal

Aplicación web para organizar mis libros: los que tengo, los que estoy leyendo y los que quiero leer, agrupados por autor.

> 🚧 Proyecto en desarrollo.

## Objetivo

Este proyecto existe para practicar el desarrollo con **Angular moderno** de punta a punta, con un caso de uso simple pero completo: un ABM con relaciones, filtros, paginación y reglas de negocio.

La idea no es construir algo grande, sino algo chico y bien hecho: código prolijo, decisiones técnicas justificadas y un flujo de trabajo como el de un proyecto real (issues, ramas, PRs y CI).

## Qué se busca practicar

**Frontend**

- Angular **zoneless**, sin `zone.js`, con detección de cambios basada en signals.
- Manejo de estado con **NgRx Signals** (`signalStore`), un store por feature.
- Lecturas de datos reactivas con la **Resource API** (`rxResource` / `httpResource`), separando lo que se _lee_ de lo que se _modifica_.
- Búsqueda con debounce, filtros combinables y estado sincronizado con la URL.
- Componentes standalone, control flow nuevo, `input()` / `output()` e `inject()`.
- Estilos con **Sass**: design tokens, tema claro y oscuro, diseño responsive y accesibilidad básica, sin librerías de componentes.

**Backend**

- API REST con **NestJS**, simple y bien validada.
- Persistencia liviana con **SQLite**, para que el proyecto corra sin instalar nada extra.
- Reglas de negocio validadas en el servidor, no solo en el front.
- Documentación de la API con **Swagger**.

**Calidad y flujo de trabajo**

- TypeScript estricto, lint y formato automático.
- Tests de lo importante: reglas de negocio en el back y store en el front.
- CI con GitHub Actions.

## Funcionalidades principales

- ABM de autores y libros.
- Estado de lectura de cada libro (pendiente, leyendo, leído) con puntuación para los leídos.
- Búsqueda por título o autor, filtros por estado y autor, y paginación.
- Resumen de la biblioteca: totales por estado y promedio de puntuación.

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
- [ ] Frontend base: zoneless, estilos, layout y rutas
- [ ] ABM de autores
- [ ] ABM de libros con filtros y URL sincronizada
- [ ] Pulido: resumen, tema oscuro, responsive y accesibilidad
- [ ] Tests, CI y deploy

## Licencia

MIT
