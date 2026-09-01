# Python 3

- [Python 3](#python-3)
  - [Qué configurar](#qué-configurar)
  - [Instalación](#instalación)
  - [.gitignore](#gitignore)
  - [Estructura de carpetas](#estructura-de-carpetas)
  - [Comandos](#comandos)
  - [Verificación](#verificación)

Proyecto de Python 3 con `uv` como gestor de paquetes y entornos.

## Qué configurar
[↑ Volver arriba](#python-3)

En este orden:

1. [`.editorconfig`](../02-tools/editorconfig.md)
2. [`.gitattributes`](../02-tools/gitattributes.md)
3. [Ruff](../02-tools/ruff.md)
4. `.gitignore` — [ver abajo](#gitignore)

Los dos primeros se usan **tal cual**, sin cambios respecto a cualquier otro stack:

* `.editorconfig` ya trae `[*.py] indent_size = 4`, que es lo que pide PEP 8. Un solo archivo cubre todos los lenguajes del repo.
* `.gitattributes` normaliza finales de línea con `* text=auto eol=lf`, que no tiene nada de específico de un lenguaje. Solo añade una línea si tu proyecto versiona un binario que no esté en la lista — `*.xlsx binary` si guardas hojas de cálculo, por ejemplo. Eso es del proyecto, no de Python.

## Instalación
[↑ Volver arriba](#python-3)

Inicializa el proyecto:

```bash
uv init <nombre-del-proyecto>
cd <nombre-del-proyecto>
```

Añade Ruff como dependencia de desarrollo:

```bash
uv add --dev ruff
```

`uv init` deja el proyecto ya inicializado en Git y con un `.gitignore` mínimo. `uv add` crea el entorno virtual en `.venv/` y el `uv.lock`.

Tres archivos que **se commitean** aunque parezcan generados:

| Archivo | Por qué |
|---|---|
| `uv.lock` | Fija las versiones exactas. Mismo papel que `pnpm-lock.yaml` |
| `.python-version` | Fija la versión de Python del proyecto |
| `pyproject.toml` | Dependencias y configuración de las herramientas |

## .gitignore
[↑ Volver arriba](#python-3)

`uv init` ya creó uno, pero cubre solo lo básico. Este lo reemplaza:

```bash
cat > .gitignore << 'EOF'
# Environment & secrets
# Order matters: the negation must come after the wildcard.
# Commit .env.example so collaborators know which keys are required.
.env
.env.*
!.env.example

# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info/

# NOTE: uv.lock and .python-version are intentionally NOT ignored.
# They pin the exact versions that keep every machine reproducible.

# Virtual environment
.venv/

# Tooling caches
.ruff_cache/
.pytest_cache/
.mypy_cache/
.coverage
coverage.xml
htmlcov/

# Jupyter
.ipynb_checkpoints/

# VS Code — project config is committed, personal config is not.
# Must be `.vscode/*` with the asterisk: Git never descends into an
# ignored directory, so `.vscode/` would make the negations below dead.
.vscode/*
!.vscode/settings.json
!.vscode/extensions.json
!.vscode/launch.json
!.vscode/tasks.json
!.vscode/*.code-snippets

# Logs
*.log
EOF
```

> **`>` trunca.** Aquí es intencional: reemplaza el `.gitignore` que dejó `uv init`.

`*.py[oc]` cubre `.pyo` y `.pyc` en un solo patrón. Ninguna regla lleva `/` al inicio, así que aplican a cualquier profundidad.

Si tu proyecto guarda datos locales que no deben viajar al repo, añade el par completo — así la carpeta existe en un clon nuevo aunque esté vacía:

```gitignore
data/*
!data/.gitkeep
```

## Estructura de carpetas
[↑ Volver arriba](#python-3)

`uv init` genera el esqueleto. Cuál te da depende del flag:

**Aplicación** — `uv init <nombre>`, para un script o programa que se ejecuta:

```
proyecto/
├── .python-version     versión de Python fijada
├── .gitignore
├── README.md
├── main.py             punto de entrada
├── pyproject.toml      dependencias y config de Ruff
└── uv.lock             versiones exactas
```

**Librería** — `uv init --lib <nombre>`, para código que otro proyecto importará:

```
proyecto/
├── src/
│   └── nombre_proyecto/
│       ├── __init__.py
│       └── py.typed    declara que el paquete lleva anotaciones de tipo
├── pyproject.toml
└── uv.lock
```

El layout `src/` no es decorativo: obliga a que los tests importen el paquete instalado y no los archivos sueltos del directorio, que es como se detectan los errores de empaquetado antes de publicar.

`.venv/` aparece al primer `uv add` y nunca se commitea.

## Comandos
[↑ Volver arriba](#python-3)

Python no tiene un bloque `scripts` como `package.json`. Los comandos se corren directo, y `uv run` se encarga de usar el entorno del proyecto sin que tengas que activarlo.

**Revisar sin tocar nada:**

```bash
uv run ruff check .
uv run ruff format --diff .
```

**Aplicar:**

```bash
uv run ruff check --fix .
uv run ruff format .
```

Primero `check --fix`, después `format`. El linter reordena imports y el formateador deja el resultado presentable; al revés tendrías que formatear dos veces.

**Ejecutar el proyecto:**

```bash
uv run main.py
```

**Primer formateo de un proyecto existente** — en un commit aparte, para no mezclar formato con lógica:

```bash
git add .
git commit -m "style: apply ruff formatting"
```

## Verificación
[↑ Volver arriba](#python-3)

```bash
uv run ruff check .
```

Exit code `0` y `All checks passed!` = todo limpio.

Si algo falla, Ruff señala el archivo, la línea y el código de la regla. Ese código es lo que buscas en la documentación cuando el mensaje no basta.
