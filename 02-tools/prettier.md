# Prettier

- [Prettier](#prettier)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos Útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#prettier)

Code formatter, no analiza calidad de código(eso lo hace ESLint). Solo reescribe el formato: comillas, punto y coma, ancho de línea, indentación, trailing commas.

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

De mayor a menor prioridad:
1. `.prettierrc`
2. `.editorconfig` (solo rellena lo que el `.prettierrc` no definió)
3. Defaults de Prettier

* Si una opción está en los dos archivos, gana `.prettierrc`. No la repitas.
* `.editorconfig` aplica a todo el repo (`.md`, `.yml`, `.env`), Prettier no.
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

Crea el archivo .prettierrc, raíz de tu proyecto:
```bash
echo {} > .prettierrc
```
Ahora, abre el archivo y coloca:
```json
{
  "singleQuote": true,
  "semi": true,
  "arrowParens": "always",
  "trailingComma": "all",
  "bracketSameLine": false,
  "htmlWhitespaceSensitivity": "css",
  "overrides": [
    { "files": "*.scss", "options": { "singleQuote": false } }
  ]
}
```

Crea el archivo .prettierignore, raíz de proyecto:
```bash
echo > .prettierignore
```
Ahora, abre el archivo y coloca:
```
pnpm-lock.yaml

# Vendor code: keep upstream formatting so updates stay diffable.
normalize.css
_normalize.scss

# Compiled assets
*.min.css
*.min.js
```

## Comandos Útiles
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

Sin salida y sin error = todo bien formateado.

## Diccionario
[↑ Volver arriba](#prettier)

**`singleQuote: true`**
Usa comillas simples en vez de dobles.

```js
const name = 'Miller';   // en vez de "Miller"
```

**`semi: true`**
Pone punto y coma al final de cada instrucción.

```js
const total = 10;
```

**`arrowParens: "always"`**
Pone paréntesis a las arrow functions aunque tengan un solo parámetro.

```js
(item) => item.id      // en vez de item => item.id
```

**`trailingComma: "all"`**
Deja coma después del último elemento en listas de varias líneas.
Hace que los diffs de Git solo marquen la línea nueva.

```js
const colors = [
  'red',
  'blue',
];
```

**`bracketSameLine: false`**
Cuando una etiqueta HTML se parte en varias líneas, el `>` baja a su propia línea.

```html
<button
  class="btn"
  type="submit"
>
```

**`htmlWhitespaceSensitivity: "css"`**
Respeta los espacios que sí importan según el `display` del elemento.
Evita que Prettier rompa el espaciado de un `<span>` o un `<a>`.

**`overrides`**
Aplica reglas distintas a ciertos archivos.
Aquí: en `.scss` usa comillas dobles, aunque el resto del proyecto use simples.

```json
"overrides": [
  { "files": "*.scss", "options": { "singleQuote": false } }
]
```
