# HTML · CSS · Sass

- [HTML · CSS · Sass](#html--css--sass)
  - [Qué configurar](#qué-configurar)
  - [Instalación](#instalación)
  - [.gitignore](#gitignore)
  - [Estructura de carpetas](#estructura-de-carpetas)
  - [Scripts de package.json](#scripts-de-packagejson)
  - [Verificación](#verificación)

Proyecto de sitio estático. Sin framework.

## Qué configurar
[↑ Volver arriba](#html--css--sass)

En este orden:

1. [`.editorconfig`](../02-tools/editorconfig.md)
2. [`.gitattributes`](../02-tools/gitattributes.md)
3. [Prettier](../02-tools/prettier.md)
4. Stylelint — pendiente
5. `.gitignore` — [ver abajo](#gitignore)

## Instalación
[↑ Volver arriba](#html--css--sass)

Inicializa el proyecto:

```bash
pnpm init
```

Instala las dependencias:

```bash
pnpm add -D --save-exact prettier stylelint stylelint-config-standard-scss sass
```

## .gitignore
[↑ Volver arriba](#html--css--sass)

Crea el archivo `.gitignore`, raíz de tu proyecto:

```bash
echo > .gitignore
```

Ahora, abre el archivo y coloca:

```
# Dependencies
node_modules/

# Build outputs
dist/
build/
*.css.map
.sass-cache/

# Logs
*.log

# Environment variables
.env
.env.*
!.env.example

# Enforce pnpm as the only package manager
package-lock.json
yarn.lock

# Claude Code
.claude/
```

`build/` y `dist/` sin `/` al inicio matchean a cualquier profundidad, así que
cubren también `11-cafeteria/build/` y cualquier carpeta que agregues después.

## Estructura de carpetas
[↑ Volver arriba](#html--css--sass)

Arquitectura 7-1 de Sass:

```
scss/
├── abstracts/      variables, mixins, functions
├── base/           reset, tipografía, estilos globales
├── components/     botones, cards, formularios
├── layout/         header, footer, grid, sidebar
├── pages/          estilos específicos de una página
├── themes/         temas de color
├── vendors/        librerías externas (normalize, etc.)
└── main.scss       único archivo que importa todo lo demás
```

Solo `main.scss` se compila. Los demás archivos llevan guion bajo
(`_variables.scss`) para que Sass no genere un `.css` por cada uno.

## Scripts de package.json
[↑ Volver arriba](#html--css--sass)

```json
"scripts": {
  "sass": "sass scss/main.scss css/style.css --watch",
  "format": "prettier --write .",
  "format:check": "prettier --check .",
  "lint:css": "stylelint \"scss/**/*.scss\"",
  "lint:css:fix": "stylelint \"scss/**/*.scss\" --fix"
}
```

## Verificación
[↑ Volver arriba](#html--css--sass)

```bash
pnpm format:check
pnpm lint:css
```

Sin salida y sin error = todo bien.
