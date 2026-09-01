# Prettier

- [Prettier](#prettier)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [.prettierignore](#prettierignore)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#prettier)

Code formatter, no analiza calidad de código (eso lo hace ESLint). Solo reescribe el formato: comillas, punto y coma, ancho de línea, indentación, trailing commas.

## Lenguajes que soporta
[↑ Volver arriba](#prettier)

De fábrica, sin plugins:

| Área | Extensiones |
|---|---|
| Web | `.html`, `.vue` |
| Estilos | `.css`, `.scss`, `.less` |
| JavaScript | `.js`, `.mjs`, `.cjs`, `.jsx` |
| TypeScript | `.ts`, `.tsx` |
| Datos | `.json`, `.yaml`, `.yml` |
| Docs | `.md`, `.mdx` |
| Otros | `.graphql`, `.hbs` |

**¿Para qué sirve en la práctica?**

Elimina las discusiones de estilo del code review y evita diffs ruidosos en Git donde la mitad de las líneas cambiaron solo porque otro editor indentó distinto.

## Precedencias
[↑ Volver arriba](#prettier)

Hay dos precedencias distintas y conviene no confundirlas.

**1. Qué archivo de configuración gana**

Prettier usa **el primero que encuentra** y se detiene ahí. No combina varios:

```
package.json ("prettier")  →  .prettierrc  →  .prettierrc.json
→  .prettierrc.yml/.yaml   →  .prettierrc.json5
→  .prettierrc.js/.cjs/.mjs
→  prettier.config.js/.cjs/.mjs  →  .prettierrc.toml
```

Si tienes dos archivos de config en el repo, el de menor prioridad se ignora en
silencio. Ese es el origen del clásico "cambié la config y no pasó nada".
Mantén uno solo por proyecto.

**2. Qué valor gana para una opción**

De mayor a menor prioridad:

1. `.prettierrc.json`
2. `.editorconfig` (solo rellena lo que el `.prettierrc.json` no definió)
3. Defaults de Prettier

* Si una opción está en los dos archivos, gana `.prettierrc.json`. No la repitas.
* Prettier no lee el `.editorconfig` entero: solo convierte cinco claves
  (`indent_style`, `indent_size`/`tab_width`, `max_line_length`, `quote_type`,
  `end_of_line`) y descarta el resto.
* `.editorconfig` gobierna todos los archivos del repo en tu editor, incluso los
  que Prettier no formatea. Sobre esos no hay precedencia que discutir: manda
  `.editorconfig` porque Prettier ni se entera.
* ESLint no entra en esta lista: es otro programa. Para que no pelee con
  Prettier, instala `eslint-config-prettier`.

## Instalación
[↑ Volver arriba](#prettier)

En terminal, dentro del proyecto:

```bash
pnpm add -D --save-exact prettier
```

Verifica que quedó instalado:

```bash
pnpm prettier --version
```

## Configuración
[↑ Volver arriba](#prettier)

Crea el archivo `.prettierrc.json` en la raíz de tu proyecto, ya con su
contenido:

```bash
cat > .prettierrc.json << 'EOF'
{
  "$schema": "https://json.schemastore.org/prettierrc",
  "singleQuote": true,
  "printWidth": 100
}
EOF
```

> **Las comillas en `'EOF'` no son opcionales aquí.** El contenido incluye
> `$schema`. Sin comillas, la shell lo trata como variable, la encuentra vacía y
> escribe `"": "https://..."` en el archivo. La config queda rota y el error no
> es evidente al leerla. Con el delimitador entrecomillado, el bloque se copia
> literal.

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo
> reemplaza por completo sin preguntar.

## .prettierignore
[↑ Volver arriba](#prettier)

Archivos que Prettier no debe tocar. Usa la misma sintaxis que `.gitignore`.
`node_modules` ya viene excluido por defecto, no hace falta listarlo.

Crea el archivo `.prettierignore` en la raíz de tu proyecto, ya con su
contenido:

```bash
cat > .prettierignore << 'EOF'
pnpm-lock.yaml

# Vendor code: keep upstream formatting so updates stay diffable.
normalize.css
_normalize.scss

# Compiled assets
*.min.css
*.min.js
EOF
```

## Comandos útiles
[↑ Volver arriba](#prettier)

Agrega los scripts a tu `package.json`:

```json
"scripts": {
  "format": "prettier --write .",
  "format:check": "prettier --check ."
}
```

**Proyecto nuevo:**

Formatea todo:

```bash
pnpm format
```

**Proyecto existente:**

Primero revisa qué archivos cambiarían, sin tocar nada:

```bash
pnpm format:check
```

Si el resultado te convence, formatea:

```bash
pnpm format
```

Haz ese primer formateo en un commit aparte, para no ensuciar el historial:

```bash
git add .
git commit -m "style: apply prettier formatting"
```

## Verificación
[↑ Volver arriba](#prettier)

```bash
pnpm format:check
```

Exit code `0` y este mensaje = todo bien formateado:

```
Checking formatting...
All matched files use Prettier code style!
```

Si algún archivo no cumple, los lista y termina con exit code `1`.

## Diccionario
[↑ Volver arriba](#prettier)

Referencia de opciones. El `.prettierrc.json` solo declara las desviaciones, así
que la mayoría de lo que aparece aquí no está en él. La marca lo indica:

* **sin marca** — está en el `.prettierrc.json`.
* **(default)** — ya viene activa en Prettier 3. Se documenta para que sepas qué
  hace sin que se lo pidas.
* **(no está en la config)** — no se usa en este setup. Queda documentada por si
  algún proyecto la necesita.

**`singleQuote: true`**
Usa comillas simples en vez de dobles.

```js
const name = 'Miller';   // en vez de "Miller"
```

**`printWidth: 100`**
Ancho de línea a partir del cual Prettier intenta partir el código.
El default es `80`.

**`semi: true`** (default)
Pone punto y coma al final de cada instrucción.

```js
const total = 10;
```

**`arrowParens: "always"`** (default)
Pone paréntesis a las arrow functions aunque tengan un solo parámetro.

```js
(item) => item.id      // en vez de item => item.id
```

**`trailingComma: "all"`** (default)
Deja coma después del último elemento en listas de varias líneas.
Hace que los diffs de Git solo marquen la línea nueva.

```js
const colors = [
  'red',
  'blue',
];
```

**`bracketSameLine: false`** (default)
Cuando una etiqueta HTML se parte en varias líneas, el `>` baja a su propia línea.

```html
<button
  class="btn"
  type="submit"
>
```

**`htmlWhitespaceSensitivity: "css"`** (default)
Respeta los espacios que sí importan según el `display` del elemento.
Evita que Prettier rompa el espaciado de un `<span>` o un `<a>`.

**`overrides`** (no está en la config)
Aplica opciones distintas a ciertos archivos. No se usa en este setup —
`singleQuote` aplica a todo el proyecto, incluido CSS y SCSS— pero queda
documentado por si algún proyecto lo necesita.

Si el patrón no lleva barra, Prettier lo matchea contra el nombre del archivo,
así que `*.scss` ya alcanza cualquier profundidad. `**.scss` es equivalente y
solo añade ruido.

```json
"overrides": [
  { "files": "*.scss", "options": { "singleQuote": false } }
]
```
