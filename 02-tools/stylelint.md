# Stylelint

- [Stylelint](#stylelint)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#stylelint)

Linter de CSS y SCSS. Revisa **qué escribiste**, no cómo se ve. El formato es tarea del formateador; Stylelint no opina sobre eso.

**¿Para qué sirve en la práctica?**

Atrapa errores que el navegador ignora en silencio. Una propiedad mal escrita, una unidad inexistente o una propiedad declarada dos veces no producen ningún aviso: el navegador simplemente descarta esa línea y tú buscas el bug en otro lado.

```scss
.card {
  margn: 5px;      /* propiedad inexistente */
  width: 10pxx;    /* unidad inexistente */
  color: red;
  color: blue;     /* la primera nunca se aplica */
}
```

Nada de eso rompe la compilación de Sass ni aparece en la consola. Stylelint te lo marca antes del commit.

## Lenguajes que soporta
[↑ Volver arriba](#stylelint)

| Área | Extensiones |
|---|---|
| CSS | `.css` |
| SCSS | `.scss` (requiere el config de SCSS) |
| Otros | `.less`, `.sass`, CSS-in-JS — vía plugins |

**No soporta:**

* JavaScript y TypeScript — eso es ESLint.
* HTML — no revisa el marcado, solo las hojas de estilo.
* Formato — desde la versión 15 no reordena ni reindenta nada.

## Precedencias
[↑ Volver arriba](#stylelint)

De mayor a menor prioridad:

1. El bloque `rules` de tu `.stylelintrc.json`
2. Las reglas del config que extiendas (`stylelint-config-standard-scss`)
3. Nada — Stylelint sin config no aplica ninguna regla

* Una regla que declares en `rules` sobrescribe la del preset. Para apagarla, `null`.
* **No choca con el formateador.** Desde Stylelint 15 se eliminaron las reglas de estilo del core, así que los dos dejaron de competir. Por eso `stylelint-config-prettier` está deprecado: ya no hay nada que desactivar.
* **No lee el `.gitignore`.** A diferencia de otras herramientas, aquí no hay herencia: usa un `.stylelintignore` o acota el glob del comando.

## Instalación
[↑ Volver arriba](#stylelint)

En terminal, dentro del proyecto:

```bash
pnpm add -D --save-exact stylelint stylelint-config-standard-scss
```

`stylelint-config-standard-scss` ya arrastra `stylelint-config-standard` y `postcss-scss`. No hace falta instalarlos aparte.

Verifica que quedó instalado:

```bash
pnpm stylelint --version
```

## Configuración
[↑ Volver arriba](#stylelint)

Crea el archivo `.stylelintrc.json` en la raíz de tu proyecto, ya con su contenido:

```bash
cat > .stylelintrc.json << 'EOF'
{
  "$schema": "https://json.schemastore.org/stylelintrc.json",
  "extends": "stylelint-config-standard-scss",
  "rules": {
    "selector-class-pattern": [
      "^[a-z][a-z0-9]*(-[a-z0-9]+)*(__[a-z0-9]+(-[a-z0-9]+)*)?(--[a-z0-9]+(-[a-z0-9]+)*)?$",
      { "message": "Class names must follow BEM: block, block__element, block--modifier" }
    ]
  }
}
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

> **Aquí el `'EOF'` entrecomillado es obligatorio.** El patrón BEM termina en `$`, que es un ancla de la expresión regular. Sin comillas la shell lo tomaría como variable y te dejaría el patrón cortado.

El archivo solo declara la desviación: todo lo demás viene del preset.

Qué acepta y qué rechaza el patrón:

```
.card                 ✓  block
.card-header          ✓  block con guion
.card__title          ✓  element
.card--dark           ✓  modifier
.card__title--big     ✓  element con modifier
.cardTitle            ✗  camelCase
.card__a__b           ✗  element anidado
```

## Comandos útiles
[↑ Volver arriba](#stylelint)

Agrega los scripts a tu `package.json`:

```json
"scripts": {
  "lint:css": "stylelint \"**/*.scss\"",
  "lint:css:fix": "stylelint \"**/*.scss\" --fix"
}
```

Las comillas del glob son necesarias: sin ellas lo expande la shell antes de que Stylelint lo vea, y solo revisaría el primer nivel.

**Proyecto nuevo:**

```bash
pnpm lint:css
```

**Proyecto existente:**

Primero revisa qué encontraría, sin tocar nada:

```bash
pnpm lint:css
```

Lo que sea corregible de forma automática:

```bash
pnpm lint:css:fix
```

`--fix` no arregla todo. Un nombre de clase fuera de BEM o una propiedad mal escrita requieren decisión tuya: renombrar una clase puede romper el HTML que la usa, así que Stylelint no lo hace por ti.

## Verificación
[↑ Volver arriba](#stylelint)

```bash
pnpm lint:css
```

Exit code `0` y sin salida = todo limpio.

Cuando encuentra algo, lo lista con archivo, línea y el nombre de la regla:

```
scss/base/_globales.scss
  2:3   ×  Unknown property "margn"   property-no-unknown
  7:1   ×  Class names must follow BEM: block, block__element, block--modifier  selector-class-pattern

× 2 problems (2 errors, 0 warnings)
```

El nombre de la regla al final es lo que buscas en la documentación si no entiendes el mensaje.

## Diccionario
[↑ Volver arriba](#stylelint)

Referencia de opciones y reglas. La marca indica de dónde sale cada una:

* **sin marca** — está en tu `.stylelintrc.json`.
* **(del preset)** — viene de `stylelint-config-standard-scss`. No la escribes; se documenta para que sepas qué te está revisando.

**`extends`**
Hereda la configuración de otro paquete. Es la base sobre la que aplicas tus reglas.

**`rules`**
Tus reglas propias. Sobrescriben las heredadas. Para apagar una del preset, dale `null`:

```json
"rules": { "no-descending-specificity": null }
```

**`selector-class-pattern`**
Exige que los nombres de clase sigan un patrón. Aquí impone BEM.
El segundo elemento del array es el mensaje que verás al fallar; sin él, Stylelint muestra la expresión regular cruda, que no le dice nada a nadie.

**`property-no-unknown`** (del preset)
Detecta propiedades que no existen: `margn`, `colour`, `bacground`.

**`unit-no-unknown`** (del preset)
Detecta unidades inválidas: `10pxx`, `2ren`.

**`declaration-block-no-duplicate-properties`** (del preset)
La misma propiedad dos veces en un bloque. La primera queda muerta.

**`block-no-empty`** (del preset)
Bloques vacíos: código que quedó a medias o sobró tras un refactor.

**`no-descending-specificity`** (del preset)
Un selector menos específico después de uno más específico sobre el mismo elemento. Es la causa habitual de "esta regla no se aplica y no entiendo por qué".
