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
# Hallazgos: El Modelo de Caja

Todo elemento HTML es una caja con cuatro capas, de adentro hacia afuera: **contenido → padding → border → margin**. En la tienda trabajé estas capas sobre los `<article>` (las tarjetas de producto) con esta regla:

```css
article {
    padding: 20px;
    margin: 15px;
}
```

### 1. Diferencia práctica entre Margin y Padding

**Padding (espacio interno):** está dentro de la caja, entre el contenido y el borde. Si la caja tiene fondo, el padding se pinta con ese fondo. En los `article` apliqué `padding: 20px`, y así el título, la descripción, el precio y el enlace "Comprar" ya no quedan pegados a los bordes de la tarjeta.

**Margin (espacio externo):** está fuera de la caja y es siempre transparente, no se pinta con el fondo. Con `margin: 15px` cada tarjeta se separa de las demás y de los bordes del `<main>`.

Lo que observé:
- Como los `article` todavía no tienen fondo ni borde, el padding casi no se nota a simple vista, pero en DevTools se resalta en verde. El margin se resalta en naranja.
- Si quiero que un elemento "respire" por dentro, uso padding. Si quiero alejarlo de sus vecinos, uso margin.

| | Padding | Margin |
|---|---|---|
| Ubicación | Dentro del borde | Fuera del borde |
| Se pinta con el fondo | Sí | No |
| Colapsa con vecinos | No | Sí (en vertical) |
| Mi valor en el proyecto | 20px | 15px |

### 2. Por qué es obligatorio usar `box-sizing: border-box`

Por defecto los navegadores usan `content-box`: el `width` mide **solo el contenido**, y el padding y el border se suman por fuera. Ejemplo con una tarjeta de `width: 300px` y mi `padding: 20px`:

- `content-box`: 300 + 20 + 20 = **340px** reales (más el margin).
- `border-box`: **300px** reales, y el contenido se encoge a 260px para dejar espacio al padding.

Con `content-box` tendría que calcular a mano cada vez (`width` menos padding menos border) para que las tarjetas encajen, y si me equivoco se desbordan de su contenedor o se rompen las columnas. Con `border-box` el `width` es el tamaño final de la caja, así que el diseño es predecible aunque cambie el padding.

Por eso agregué la regla global al inicio del CSS:

```css
*, *::before, *::after {
    box-sizing: border-box;
}
```

Nota: en mi página actual los `article` no tienen `width` definido, por lo que ocupan todo el ancho disponible y la diferencia aún no se ve. Se vuelve importante cuando defina anchos (por ejemplo, tarjetas en columnas).

### 3. Cómo se ven afectados los elementos hermanos con un margen grande

Los hermanos son elementos al mismo nivel, como los cuatro `<article>` dentro de `<section>`. Un margen grande en uno de ellos lo aleja de los demás:
- **Margen vertical** (`margin-top` / `margin-bottom`): los hermanos de abajo bajan, y si es `margin-top`, el de arriba queda más lejos.
- **Margen horizontal** (`margin-left` / `margin-right`): como los `article` son elementos de bloque, el margen reduce el ancho disponible y mueve el contenido hacia adentro.

Hallazgos concretos de mi proyecto:
1. **Colapso de márgenes:** entre dos tarjetas seguidas la separación es de **15px, no 30px**. Los márgenes verticales de bloques contiguos no se suman: prevalece el mayor. Si una tarjeta tuviera `margin-bottom: 15px` y la siguiente `margin-top: 50px`, la separación sería 50px, no 65px.
2. **En horizontal no hay colapso:** los márgenes izquierdo y derecho sí se aplican completos (15px por lado).
3. **Auditoría en DevTools (pestaña Computed):** comprobé que el diagrama muestra padding 20 y margin 15 en los cuatro lados, igual que mi código. El ancho horizontal total de una tarjeta es: `contenido + 20 + 20 + 0 + 15 + 15 = contenido + 70px`.
4. **Efecto sobre el contenedor:** como `<main>` no tiene padding ni borde, el margen superior de la primera tarjeta puede "salirse" y empujar a `<main>` entero hacia abajo, dejando una franja sin el color amarillo entre el header y el main. Se puede verificar en DevTools resaltando `<main>`.

### Conclusión

El padding protege el contenido, el margin organiza las cajas entre sí, y `border-box` hace que los tamaños que escribo sean los que realmente se ven. Entender estas tres cosas me permite predecir el espacio que ocupa cada elemento en lugar de ajustar a prueba y error.
