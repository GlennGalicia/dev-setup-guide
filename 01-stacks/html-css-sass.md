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
4. [Stylelint](../02-tools/stylelint.md)
5. [Husky](../02-tools/husky.md) — requiere Prettier y Stylelint ya configurados
6. `.gitignore` — [ver abajo](#gitignore)

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

Crea el archivo `.gitignore` en la raíz de tu proyecto, ya con su contenido:

```bash
cat > .gitignore << 'EOF'
# Dependencies
node_modules/

# Build outputs
dist/
build/
css/
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

# VS Code — project config is committed, personal config is not.
# Must be `.vscode/*` with the asterisk: Git never descends into an
# ignored directory, so `.vscode/` would make the negations below dead.
.vscode/*
!.vscode/settings.json
!.vscode/extensions.json
!.vscode/launch.json
!.vscode/tasks.json
!.vscode/*.code-snippets

# Claude Code
.claude/
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

`build/`, `dist/` y `css/` sin `/` al inicio matchean a cualquier profundidad, así que
cubren también `11-cafeteria/build/` y cualquier carpeta que agregues después.

`css/` es la salida de Sass, no código fuente. Prettier lee este archivo, así que
al ignorarlo aquí tampoco lo formatea: no hay que repetirlo en el `.prettierignore`.

## Estructura de carpetas
[↑ Volver arriba](#html--css--sass)

Arquitectura 7-1 de Sass:

TODO añadir estructura de mis proyectos

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

TODO revisar los scripts a configurar

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

`format:check` da exit code `0` y este mensaje cuando todo está formateado:

```
Checking formatting...
All matched files use Prettier code style!
```

`lint:css` da exit code `0` y sin salida cuando no encuentra problemas. Requiere el `.stylelintrc.json` que crea la guía de [Stylelint](../02-tools/stylelint.md); sin ese archivo el comando aborta.
TODO revisar como queda lo de .vcode
TODO compartir los warnings que me esta dando el otro proyecto
TODO correr comandos y verificar errores
