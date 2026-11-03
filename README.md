# Tienda - Maquetado semántico y jerarquía CSS

Proyecto de práctica de HTML semántico y CSS (SENA - ADSO, ficha 3381732).

## Estructura del proyecto

- `index.html`: catálogo con `header`, `nav`, `main`, `section`, `article`, `aside` y `footer`.
- `estilos.css`: herencia desde `body`, selectores por zona y clases para los botones de compra.

## Cálculo de especificidad

| Tipo de selector | Puntos |
|------------------|--------|
| Etiqueta         | 1      |
| Clase            | 10     |
| ID               | 100    |

Cuando dos reglas entran en conflicto, gana la de mayor puntaje. Si empatan, gana la que está después en el archivo (cascada).

### Selectores del proyecto

| Selector              | Cálculo      | Puntos |
|-----------------------|--------------|--------|
| `body`                | 1 etiqueta   | 1      |
| `header nav a`        | 3 etiquetas  | 3      |
| `aside a`             | 2 etiquetas  | 2      |
| `footer a`            | 2 etiquetas  | 2      |
| `.btn-comprar`        | 1 clase      | 10     |
| `.btn-destacado`      | 1 clase      | 10     |

### Conflicto inicial (versión con ID)

| Selector            | Cálculo    | Puntos |
|---------------------|------------|--------|
| `#btn-destacado`    | 1 ID       | 100    |
| `.btn-comprar`      | 1 clase    | 10     |

El ID (100) superaba a la clase (10), por lo que el enlace se veía rojo aunque la regla verde estuviera después en el archivo.

### Refactorización

Se eliminó el `#id` y se pasó a un sistema plano de etiquetas y clases, sin `!important`:

- `.btn-comprar` es la clase base (verde).
- `.btn-destacado` es el modificador (rojo) y va después de la base.
- Ambas valen 10 puntos, así que el orden decide y no hay colisión.
- Los selectores de zona se aplanaron: `header nav ul li a` (5 pts) pasó a `header nav a` (3 pts).

## Herencia

`font-family` y `color` se declaran solo en `body`. Los `h2`, `p` y `li` los heredan automáticamente. Los enlaces `<a>` tienen su propio color por zona.