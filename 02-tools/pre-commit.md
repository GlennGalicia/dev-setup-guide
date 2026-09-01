# pre-commit (Python)

> **En este setup:** gestor de hooks de los proyectos Python. En JS ese papel lo cumple Husky. No los mezcles en un mismo repo: Husky apunta `core.hooksPath` a `.husky/` y `pre-commit` escribe en `.git/hooks/`, así que uno de los dos deja de ejecutarse sin avisar.
>
> **Complementario a [Ruff](./ruff.md):** `pre-commit` no revisa nada por sí mismo, solo decide *cuándo* corren las herramientas. Configura Ruff primero o no tendrá nada que ejecutar.

- [pre-commit (Python)](#pre-commit-python)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#pre-commit-python)

Un administrador de hooks de Git. El nombre está sobrecargado y conviene separarlo desde el principio:

| `pre-commit` | Qué es |
|---|---|
| El **hook** | Un archivo ejecutable en `.git/hooks/pre-commit`. Nativo de Git, existe sin instalar nada |
| La **herramienta** | Un paquete que genera y administra ese archivo desde un `.pre-commit-config.yaml` |

La herramienta escribe el hook. Tú editas el YAML, nunca el archivo en `.git/hooks/`.

**¿Para qué sirve en la práctica?**

Mueve la revisión del momento en que te acuerdas al momento en que commiteas. Sin hook, `ruff check` se corre cuando alguien se acuerda; con hook, no hay commit sin pasar por ahí.

Y como solo mira los archivos en el *staging area*, revisa lo que estás por commitear, no el proyecto entero.

## Lenguajes que soporta
[↑ Volver arriba](#pre-commit-python)

La herramienta está escrita en Python pero **no es exclusiva de Python**: gestiona hooks de JS, Go, Rust, YAML, shell y más. Aquí se usa en el lado Python porque en JS ya tienes Husky, no por una limitación suya.

**No hace:**

* Revisar código. Eso lo hacen las herramientas que tú declares.
* Instalarse solo en un clon nuevo — a diferencia de Husky, que se engancha al `prepare` de `package.json`. Ver [Instalación](#instalación).

## Precedencias
[↑ Volver arriba](#pre-commit-python)

De mayor a menor prioridad:

1. `git commit --no-verify` — salta todos los hooks
2. `.pre-commit-config.yaml`
3. Nada: sin ese archivo, `pre-commit` no hace nada

* **Solo ve archivos en staging.** Un archivo modificado pero sin `git add` no se revisa, y esa es la causa habitual de "pasó el hook y el CI falló".
* **Choca con Husky.** Los dos quieren gobernar el mismo hook. Uno por repo.
* `--no-verify` existe para emergencias. Si lo usas seguido, el problema es la configuración, no el hook.

## Instalación
[↑ Volver arriba](#pre-commit-python)

Como dependencia de desarrollo:

```bash
uv add --dev pre-commit
```

Y después, para que escriba el hook en `.git/hooks/`:

```bash
uv run pre-commit install
```

> **Ese segundo comando hay que correrlo una vez por clon.** `pre-commit` no tiene equivalente al `prepare` de `package.json`: quien clone el repo tendrá el YAML pero no el hook, y commiteará sin validar sin enterarse. Déjalo escrito en el README del proyecto.

Verifica que quedó instalado:

```bash
uv run pre-commit --version
ls .git/hooks/pre-commit
```

## Configuración
[↑ Volver arriba](#pre-commit-python)

Crea el archivo `.pre-commit-config.yaml` en la raíz de tu proyecto, ya con su contenido:

```bash
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: local
    hooks:
      - id: ruff-check
        name: ruff check
        entry: uv run ruff check --fix
        language: system
        types_or: [python, pyi]
        require_serial: true

      - id: ruff-format
        name: ruff format
        entry: uv run ruff format
        language: system
        types_or: [python, pyi]
        require_serial: true
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

**Por qué `repo: local` y no el repositorio oficial de Ruff.**

La forma estándar es apuntar a `astral-sh/ruff-pre-commit` con un `rev` fijado, y entonces `pre-commit` descarga su propia copia aislada de Ruff. Eso te deja la versión en dos sitios:

```
uv.lock                    →  la versión que usas con uv run
.pre-commit-config.yaml    →  la versión que corre al commitear
```

El día que difieran, el hook bloquea un commit por algo que `uv run ruff check .` no reporta. Con `repo: local` hay un solo Ruff, el del `uv.lock`, y no hay nada que sincronizar.

El precio: quien clone necesita `uv` instalado. En este setup ya lo necesita para todo lo demás.

El orden de los hooks importa: `check --fix` primero, `format` después. El linter reordena imports y el formateador deja el resultado presentable.

## Comandos útiles
[↑ Volver arriba](#pre-commit-python)

Los hooks corren solos al commitear. Estos comandos son para lo demás:

**Correr los hooks sobre todo el proyecto**, no solo lo que está en staging:

```bash
uv run pre-commit run --all-files
```

Es lo que quieres al configurarlo por primera vez sobre código existente.

**Saltar los hooks en un commit puntual:**

```bash
git commit --no-verify -m "wip: guardando avance"
```

**Desinstalar el hook:**

```bash
uv run pre-commit uninstall
```

## Verificación
[↑ Volver arriba](#pre-commit-python)

Haz un commit con código que sabes que falla. Debe verse así:

```
ruff check...............................................................Failed
- hook id: ruff-check
- files were modified by this hook

Found 1 error (1 fixed, 0 remaining).

ruff format..............................................................Failed
- hook id: ruff-format
- files were modified by this hook

1 file reformatted, 1 file left unchanged
```

**El commit no se creó, y eso es correcto aunque desconcierte.** Los hooks *arreglaron* los archivos, pero lo que estaba en staging ya no coincide con lo corregido. Añade los cambios y repite:

```bash
git add .
git commit -m "feat: add report parser"
```

La segunda vez pasa:

```
ruff check...............................................................Passed
ruff format..............................................................Passed
```

Ese ciclo de dos pasos es el comportamiento normal la primera vez que un archivo necesita correcciones. No es un error de configuración.

## Diccionario
[↑ Volver arriba](#pre-commit-python)

**`repos`**
Lista de fuentes de hooks. Cada una es un repositorio remoto o `local`.

**`repo: local`**
Los hooks se definen aquí mismo y usan lo que ya está instalado, en vez de descargar una copia propia. Es lo que evita duplicar la versión de Ruff.

**`id`** y **`name`**
`id` identifica el hook para `pre-commit run <id>`. `name` es lo que ves en la salida.

**`entry`**
El comando que se ejecuta. `pre-commit` le añade al final los archivos en staging que pasaron el filtro.

**`language: system`**
Usa el comando tal cual está en el sistema, sin crear un entorno aislado. Es la contraparte de `repo: local`.

**`types_or: [python, pyi]`**
Filtra qué archivos recibe el hook. Sin esto, `entry` recibiría también los `.md` o `.yaml` que estés commiteando.

**`require_serial: true`**
Ejecuta el hook una sola vez con todos los archivos, en lugar de repartirlos en procesos paralelos. Ruff ya paraleliza internamente, así que dividirlo solo añade arranques de proceso.

**`--no-verify`**
Flag de `git commit`, no de `pre-commit`. Salta todos los hooks. Para emergencias.
