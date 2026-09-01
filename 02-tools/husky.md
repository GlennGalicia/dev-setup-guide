# Husky (JS)

> **En este setup:** gestor de hooks de los proyectos JS. En Python ese papel lo cumple [pre-commit](./pre-commit.md). No los mezcles en un mismo repo: Husky apunta `core.hooksPath` a `.husky/` y `pre-commit` escribe en `.git/hooks/`, así que uno de los dos deja de ejecutarse sin avisar.
>
> **Complementario a [Prettier](./prettier.md) y [Stylelint](./stylelint.md):** Husky no revisa nada por sí mismo, solo decide *cuándo* corren las herramientas. Configúralas primero o no tendrá nada que ejecutar.

- [Husky (JS)](#husky-js)
  - [¿Qué es?](#qué-es)
  - [Hooks disponibles](#hooks-disponibles)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#husky-js)

Un administrador de hooks de Git para proyectos con `package.json`. Guarda los hooks en `.husky/`, que sí se commitea, en vez de `.git/hooks/`, que no viaja con el repo.

Se usa junto a **lint-staged**, que es quien decide qué comando corre sobre qué archivos. Los dos se documentan aquí porque por separado no hacen nada útil:

| Herramienta | Responsabilidad |
|---|---|
| Husky | *Cuándo* — engancha el hook al evento de Git |
| lint-staged | *Sobre qué* — filtra los archivos en staging y lanza el comando |
| Prettier / Stylelint | *Qué* — el trabajo real |

**¿Para qué sirve en la práctica?**

Sin hook, `pnpm format` se corre cuando alguien se acuerda. Con hook, no hay commit sin pasar por ahí. Y como lint-staged solo mira el *staging area*, revisa lo que estás por commitear y no el proyecto entero.

## Hooks disponibles
[↑ Volver arriba](#husky-js)

| Hook | Cuándo se dispara | Para qué |
|---|---|---|
| `pre-commit` | antes de crear el commit | validar y formatear código |
| `commit-msg` | después de escribir el mensaje | validar el mensaje |
| `pre-push` | antes de `git push` | correr los tests |

Cada uno es un archivo dentro de `.husky/` con ese nombre exacto.

Aquí se configura solo `pre-commit`. `commit-msg` necesita **commitlint** por detrás para validar Conventional Commits — el hook por sí solo no valida nada. `pre-push` solo tiene sentido cuando el proyecto tiene tests.

## Precedencias
[↑ Volver arriba](#husky-js)

De mayor a menor prioridad:

1. `git commit --no-verify` — salta todos los hooks
2. Los archivos dentro de `.husky/`
3. Nada: sin `.husky/pre-commit`, no ocurre nada al commitear

* **lint-staged solo ve archivos en staging.** Un archivo modificado pero sin `git add` no se revisa. Es la causa habitual de "pasó el hook y el CI falló".
* **Choca con `pre-commit`** (la herramienta de Python). Uno por repo.
* `--no-verify` existe para emergencias. Si lo usas seguido, el problema es la configuración.

## Instalación
[↑ Volver arriba](#husky-js)

```bash
pnpm add -D --save-exact husky lint-staged
```

Inicializa Husky:

```bash
pnpm exec husky init
```

Ese comando hace tres cosas: crea `.husky/`, escribe un `pre-commit` de ejemplo con `pnpm test` dentro, y añade el script `prepare` a tu `package.json`:

```json
"scripts": { "prepare": "husky" }
```

> **No borres ese `prepare`.** Es lo que reinstala los hooks tras un `pnpm install` en un clon nuevo. Sin él, quien clone tendrá los archivos de `.husky/` pero los hooks no estarán enganchados, y commiteará sin validar sin enterarse.

## Configuración
[↑ Volver arriba](#husky-js)

**1. El hook.** Reemplaza el `pnpm test` de ejemplo:

```bash
echo 'pnpm exec lint-staged' > .husky/pre-commit
```

**2. Qué corre sobre qué.** Crea el `.lintstagedrc.json` en la raíz:

```bash
cat > .lintstagedrc.json << 'EOF'
{
  "*.{html,css,js,json,md}": "prettier --write",
  "*.scss": ["stylelint --fix", "prettier --write"]
}
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

**Fíjate en que `scss` no está en el primer patrón.** Si estuviera en los dos, lint-staged lanzaría las tareas de ambos patrones **en paralelo** y dos procesos escribirían el mismo archivo a la vez. Cada archivo debe caer en un solo patrón.

Dentro de un patrón, el array **sí garantiza el orden**: primero `stylelint --fix`, después `prettier --write`. El linter corrige y el formateador deja el resultado presentable; al revés tendrías que formatear dos veces.

**3. `.gitignore`.** Nada que añadir: Husky pone su propio `.gitignore` dentro de `.husky/_/`, que es la única parte generada. Tus hooks quedan como archivos rastreados, que es lo que quieres — **no ignores `.husky/`** o dejarán de viajar con el repo.

## Comandos útiles
[↑ Volver arriba](#husky-js)

Los hooks corren solos al commitear. Estos son para lo demás:

**Probar lint-staged sin commitear:**

```bash
pnpm exec lint-staged
```

**Saltar los hooks en un commit puntual:**

```bash
git commit --no-verify -m "wip: guardando avance"
```

**Desinstalar los hooks** sin borrar `.husky/`:

```bash
git config --unset core.hooksPath
```

## Verificación
[↑ Volver arriba](#husky-js)

Commitea un archivo mal formateado. Debe verse así:

```
⋯ Running tasks for staged files…
    *.{html,css,js,json,md} — 1 file
      ⋯ prettier --write
    *.scss — 1 file
      ⋯ stylelint --fix
      ⋯ prettier --write

✔ prettier --write
✔ stylelint --fix
✔ prettier --write

✔ Done running tasks for staged files!
⋯ Staging changes from tasks…
✔ Done staging changes from tasks!
```

**El commit se crea, ya con los archivos corregidos.** lint-staged vuelve a añadirlos al staging por ti — no tienes que repetir el `git add` ni el commit.

Compruébalo mirando lo que quedó guardado, no tu copia de trabajo:

```bash
git show HEAD:src/app.scss
```

## Diccionario
[↑ Volver arriba](#husky-js)

**`.husky/pre-commit`**
El hook. Un archivo con los comandos a ejecutar, uno por línea. Se commitea.

**`.husky/_/`**
Los internos que genera Husky. Trae su propio `.gitignore` con `*`, así que se ignora solo. No lo edites.

**`"prepare": "husky"`**
Script de `package.json`. `pnpm install` lo ejecuta automáticamente, y eso reengancha los hooks en cada clon.

**`core.hooksPath`**
Config de Git que Husky apunta a `.husky/`. Es la razón por la que sus hooks funcionan sin tocar `.git/hooks/`, y también por la que choca con otros gestores.

**`.lintstagedrc.json`**
Mapa de patrón → comando. La clave es un glob; el valor, un comando o un array de comandos en orden.

**Array vs string en lint-staged**
Un string es un comando. Un array son varios **en secuencia**. Entre patrones distintos no hay orden garantizado: corren en paralelo.

**`--no-verify`**
Flag de `git commit`, no de Husky. Salta todos los hooks. Para emergencias.
