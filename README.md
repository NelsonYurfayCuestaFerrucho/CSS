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


# Hallazgos: Centralización de Identidad Visual y Espaciado

### 1. Variables declaradas en `:root`

**Colores**

| Variable | Valor | Uso |
|---|---|---|
| `--color-primario` | `#d4a62a` | Borde dorado de las tarjetas (`article`) |
| `--color-secundario` | `#2e7d32` | Enlaces del `aside` y botón `.btn-comprar` |
| `--color-acento` | `#c62828` | Enlaces del `footer` y botón `.btn-destacado` |
| `--color-texto` | `#333333` | Color de texto base del `body` |
| `--color-texto-claro` | `#ffffff` | Enlaces del menú sobre el banner |
| `--color-fondo-header` | `#e3f2fd` | Fondo de respaldo del `header` |
| `--color-fondo-main` | `#fff8e1` | Fondo del `main` |
| `--color-fondo-aside` | `#e8f5e9` | Fondo del `aside` |
| `--color-fondo-footer` | `#ffebee` | Fondo del `footer` |

**Espaciados**

| Variable | Valor | Uso |
|---|---|---|
| `--espaciado-pequeno` | `8px` | Padding vertical del `header` |
| `--espaciado-medio` | `16px` | Margen externo de las tarjetas |
| `--espaciado-grande` | `24px` | Padding de las tarjetas y separación entre opciones del menú |
| `--espaciado-menu` | `100px` | Margen superior que baja el menú dentro del banner |

**Tipografía:** `--fuente-principal` (`'Poppins', sans-serif`), cargada desde Google Fonts en el `<head>`.

### 2. Ventaja de reutilizar medidas de margen y relleno

Antes, cada selector tenía su propio número (30px, 20px, 15px, 10px...). Eran valores sueltos que no seguían ninguna lógica y que había que recordar o buscar uno por uno. Con una escala de tokens (8, 16 y 24px) todo el diseño usa las mismas proporciones, así que las tarjetas y secciones tienen el mismo "aire" y se ven ordenadas y consistentes. Además, si queremos que la página sea más espaciosa o más compacta, cambiamos un solo valor en `:root` y todo se ajusta, en lugar de editar muchos selectores. También se escribe menos código y se reducen los errores de digitación.

### 3. Cómo la centralización previene colisiones visuales

Cuando la hoja de estilos crece, es fácil que aparezcan versiones ligeramente distintas de un mismo color (por ejemplo, dos verdes parecidos en el `aside` y en un botón) o espaciados que no coinciden entre secciones. Al declarar cada valor una sola vez en `:root`, todos los selectores leen la misma fuente de verdad, por lo que no puede haber dos tonos o medidas "casi iguales" por descuido. Si un color o espacio cambia, se modifica en un único lugar y el cambio se refleja de forma uniforme en toda la página, sin dejar elementos con valores viejos. Esto facilita el mantenimiento y evita que el diseño se desordene a medida que se agregan nuevos estilos.


# Hallazgos: Pseudoclases y Pseudoelementos

En esta subactividad le di interactividad y decoración visual al Catálogo de Productos usando solo CSS, sin modificar el archivo HTML. Trabajé en la rama `feature/interactividad-css` para no afectar la versión estable del proyecto.

### 1. Diferencia funcional entre pseudoclase y pseudoelemento

**Pseudoclase (reacción a eventos o estados):**
Una pseudoclase selecciona un elemento que ya existe en el HTML, pero solo mientras se encuentra en un estado determinado. El navegador escucha las acciones del usuario y activa o desactiva el estilo automáticamente. En el proyecto usé tres:
- `:hover`: se activa cuando el cursor pasa por encima. En el `<aside>`, el enlace cambia su fondo a verde y su texto a blanco con una transición suave de 0.3 s. En los botones "Comprar", el fondo cambia al color primario.
- `:active`: se activa mientras el usuario mantiene presionado el clic. En los botones aplicé `transform: translateY(3px)` y reduje la sombra para simular que el botón se hunde.
- `:focus`: se activa cuando el elemento recibe el foco con el teclado (tecla Tab). En los enlaces del `<aside>` muestra un contorno con `outline: 3px solid var(--color-secundario)`.

**Pseudoelemento (inyección de contenido virtual):**
Un pseudoelemento no depende de un evento. Crea una parte virtual dentro del elemento, antes (`::before`) o después (`::after`) de su contenido, y se muestra con la propiedad `content`. Ese contenido existe en el DOM virtual del navegador, pero no en el archivo HTML. En el proyecto usé:
- `aside a::before` con `content: "• "` para poner una viñeta dorada antes de cada categoría.
- `article::after` con `content: "¡Oferta!"` para crear una insignia roja en la esquina de cada tarjeta, y `article:nth-child(2)::after` con `content: "Nuevo"` para que la segunda tarjeta tenga una insignia verde distinta.

**En resumen:** la pseudoclase cambia **cómo se ve** un elemento existente según su estado, y el pseudoelemento **agrega** una pieza visual que no está en el HTML.

### 2. Importancia técnica de `:focus` para la accesibilidad web

No todos los usuarios navegan con mouse. Personas con discapacidad motriz, personas con baja visión que usan lectores de pantalla y usuarios que simplemente prefieren el teclado se mueven por la página con la tecla Tab, que va saltando de un enlace o botón al siguiente. Para ellos, el único indicador de "en qué parte de la página estoy" es el contorno que muestra `:focus`.

Si ese contorno se elimina o es muy tenue, el usuario pierde la ubicación y no sabe qué enlace activará al presionar Enter. Por eso las pautas de accesibilidad web (WCAG) exigen que el foco del teclado sea visible. En mi proyecto personalicé el contorno con la variable `--color-secundario`, y agregué `outline-offset: 2px` para separarlo del enlace y que se vea con claridad. Así cada categoría del menú lateral se identifica sin ambigüedad al navegar con Tab.

### 3. Por qué es buena práctica decorar con `::before` y `::after`

- **HTML limpio y semántico:** el HTML debe contener solo contenido con significado (títulos, párrafos, enlaces). Si agregara etiquetas como `<span class="insignia"></span>` solo para decorar, mezclaría estructura con presentación.
- **Mantenimiento más fácil:** si quiero cambiar el texto de una insignia, su color o la viñeta del menú, lo hago en un solo lugar del CSS, sin editar cada tarjeta o cada enlace del HTML.
- **Reutilización:** una sola regla como `aside a::before` decora todas las categorías, incluso las que agregue en el futuro.
- **Mejor accesibilidad:** los elementos decorativos no se convierten en nodos vacíos que un lector de pantalla pueda leer sin aportar información.
- **Separación de responsabilidades:** el HTML define la estructura y el CSS define la apariencia, que es el principio que seguimos en todo el proyecto.

### 4. Auditoría con DevTools

Con las herramientas de desarrollador del navegador inspeccioné los enlaces del `<aside>` y los botones de las tarjetas. En el panel *Styles* abrí la opción **:hov** y forcé los estados `:hover`, `:active` y `:focus`, y comprobé que los estilos cambiaban como esperaba. También verifiqué que los bloques `::before` y `::after` aparecen en el árbol del DOM del navegador aunque no existen en el archivo HTML, lo que confirma que son contenido virtual inyectado desde CSS.


# Hallazgos: Arquitectura BEM y Modularidad

### 1. ¿Qué significa BEM y cómo se escribe visualmente?

BEM es la sigla de **Block, Element, Modifier** (Bloque, Elemento, Modificador). Es una metodología para nombrar las clases de CSS de forma ordenada, de manera que con solo leer el nombre de una clase se entienda a qué componente pertenece y qué papel cumple dentro de él.

Visualmente se escribe con la estructura `bloque__elemento--modificador`. El **bloque** es un componente que puede existir por sí solo, como `card`, `btn` o `sidebar`. El **elemento** es una parte interna de ese bloque y se separa con doble guion bajo (`__`), por ejemplo `card__title` o `card__price`. El **modificador** es una variante visual del bloque o del elemento y se separa con doble guion medio (`--`), por ejemplo `card--destacada` o `btn--secundario`.

En el HTML el modificador siempre acompaña a la clase base, porque la base define el estilo general y el modificador solo cambia lo que es distinto:

`<a href="#" class="btn btn--secundario">Comprar</a>`

### 2. ¿Por qué dividir el CSS en varios archivos facilita el trabajo colaborativo?

En este proyecto el CSS dejó de ser un solo archivo grande y se organizó en tres carpetas: `base/` (reset y estilos del body), `layout/` (header, contenedor, main y footer) y `components/` (tarjeta, botones y menú lateral). El archivo `estilos.css` quedó únicamente con los `@import` y las variables de `:root`, que funciona como un índice que conecta todos los módulos.

Esta división facilita el trabajo en equipo porque cada integrante puede encargarse de un componente distinto sin modificar el mismo archivo que sus compañeros. Al usar Git, esto reduce los conflictos al unir ramas, ya que los cambios quedan en archivos separados. También ayuda a encontrar errores con más rapidez: si algo falla en los botones, solo se revisa `botones.css` y no cientos de líneas mezcladas. Además, cada componente queda independiente, por lo que se puede reutilizar en otro proyecto copiando su archivo, y el código resulta más fácil de leer y de mantener.

### 3. ¿Cómo evita BEM la guerra de especificidad (cascada) al mantener un peso plano?

La especificidad es el peso que el navegador le asigna a cada selector para decidir qué regla se aplica cuando dos estilos entran en conflicto. Con selectores anidados como `.producto h2` o `aside a:hover`, el peso aumenta, y para sobrescribir esos estilos después hay que escribir selectores aún más pesados o recurrir a `!important`. Así se genera la "guerra de especificidad", donde cada cambio obliga a escribir una regla más fuerte que la anterior y el código se vuelve difícil de controlar.

BEM resuelve este problema porque cada elemento tiene su propia clase única y se estiliza solo con ella (por ejemplo `.card__title`). Todas las reglas tienen el mismo peso, una sola clase equivale a 10, sin importar en qué parte del HTML se encuentren. Como el peso es plano, el comportamiento es predecible: si dos reglas tienen el mismo peso, gana la que está más abajo en el código. Para crear una variante no hace falta escribir un selector más fuerte, basta con agregar un modificador como `.btn--secundario`. Además, los nombres son únicos (`card__title` nunca se confunde con `sidebar__title`), así que los estilos de un componente no afectan a otro por accidente.

Esto se comprobó en DevTools, en el panel de estilos, donde los selectores como `.card__title` muestran una especificidad de (0,1,0), es decir, peso 10.