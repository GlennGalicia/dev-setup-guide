# Ruff

- [Ruff](#ruff)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#ruff)

Linter **y** formateador de Python, en un solo binario. Es la excepción al reparto habitual: hace las dos cosas que en otros stacks se dividen entre dos herramientas.

**¿Para qué sirve en la práctica?**

Sustituye cuatro herramientas que antes se instalaban por separado y había que coordinar entre sí:

| Antes | Ahora |
|---|---|
| Flake8 — errores y estilo | `ruff check` |
| isort — orden de imports | regla `I` |
| pyupgrade — sintaxis moderna | reglas `UP` |
| Black — formato | `ruff format` |

Menos dependencias, un solo archivo de config, y sin el clásico conflicto de isort y Black discutiendo por la misma línea.

## Lenguajes que soporta
[↑ Volver arriba](#ruff)

| Área | Extensiones |
|---|---|
| Python | `.py`, `.pyi` |
| Notebooks | `.ipynb` |

**No soporta:**

* Cualquier lenguaje que no sea Python.
* Tipado estático — para eso es mypy o pyright. Ruff no comprueba tipos, solo los lee.

## Precedencias
[↑ Volver arriba](#ruff)

De mayor a menor prioridad:

1. Flags de la línea de comandos (`--line-length`, `--select`)
2. `.ruff.toml` o `ruff.toml`, si existen
3. `[tool.ruff]` dentro de `pyproject.toml`
4. Defaults de Ruff

* Un `ruff.toml` en la raíz **gana sobre `pyproject.toml`** y este último se ignora por completo. No los mezcles: elige uno.
* **Ruff no lee `.editorconfig`.** A diferencia de Prettier, no traduce ninguna clave. `indent_size` y `max_line_length` ahí solo afectan a tu editor; el ancho real lo decide `line-length`.
* **Sí lee el `.gitignore`.** Los archivos ignorados por Git quedan fuera sin configurar nada.
* No conviene correrlo junto a Black, isort o Flake8: hacen lo mismo y acabarían peleando. Ruff los reemplaza, no los complementa.

## Instalación
[↑ Volver arriba](#ruff)

Como dependencia de desarrollo del proyecto:

```bash
uv add --dev ruff
```

Verifica que quedó instalado:

```bash
uv run ruff --version
```

Queda anotado en `pyproject.toml` bajo `dependency-groups` y fijado en `uv.lock`, así que en otra máquina se instala en la versión exacta.

## Configuración
[↑ Volver arriba](#ruff)

`uv init` ya creó el `pyproject.toml`, así que aquí **se añade al final, no se reemplaza**:

```bash
cat >> pyproject.toml << 'EOF'

[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]
EOF
```

> **Fíjate en el doble `>>`.** Con uno solo borrarías el `pyproject.toml` entero, incluidas tus dependencias. Es el único archivo del setup que se amplía en vez de crearse.

`line-length` gobierna las dos mitades: el formateador parte ahí, y el linter reporta `E501` a partir de ahí. Un solo número, sin desincronizar nada.

## Comandos útiles
[↑ Volver arriba](#ruff)

Python no tiene un `scripts` como `package.json`; los comandos se corren directo.

**Revisar sin tocar nada:**

```bash
uv run ruff check .          # errores de lint
uv run ruff format --diff .  # qué cambiaría el formateador
```

**Aplicar:**

```bash
uv run ruff check --fix .
uv run ruff format .
```

**Orden importa:** primero `check --fix`, después `format`. El linter puede reescribir código —reordenar imports, por ejemplo— y el formateador deja el resultado presentable. Al revés tendrías que formatear dos veces.

Igual que con cualquier formateador, el primer paso sobre un proyecto existente va en un commit aparte:

```bash
git add .
git commit -m "style: apply ruff formatting"
```

## Verificación
[↑ Volver arriba](#ruff)

```bash
uv run ruff check .
```

Exit code `0` y `All checks passed!` = limpio.

Cuando encuentra algo, señala la línea y nombra la regla:

```
F401 [*] `os` imported but unused
 --> src/demo/main.py:2:8
  |
2 | import os
  |        ^^
  |
help: Remove unused import: `os`

Found 1 error.
[*] 1 fixable with the `--fix` option.
```

El `[*]` marca lo que `--fix` puede resolver solo. Lo que no lo lleva necesita decisión tuya.

El código de la izquierda (`F401`) es lo que buscas en la documentación cuando el mensaje no basta.

## Diccionario
[↑ Volver arriba](#ruff)

La marca indica de dónde sale cada cosa:

* **sin marca** — está en tu `pyproject.toml`.
* **(del default)** — comportamiento de fábrica. Se documenta para que sepas qué hace sin que se lo pidas.

**`line-length = 88`**
Ancho máximo de línea. Es el default de Ruff y de Black, así que es lo que asume todo el ecosistema. Afecta al formateador y a la regla `E501` a la vez.

**`select`**
Qué familias de reglas se activan. Cada prefijo es un grupo:

| Prefijo | Origen | Qué detecta |
|---|---|---|
| `E` | pycodestyle | Errores de estilo PEP 8, incluido el ancho de línea |
| `F` | pyflakes | Imports sin usar, variables muertas, nombres inexistentes |
| `I` | isort | Imports desordenados o mal agrupados |
| `UP` | pyupgrade | Sintaxis vieja que ya tiene reemplazo (`List[str]` → `list[str]`) |
| `B` | flake8-bugbear | Bugs que el intérprete no detecta, como un default mutable |

**`ruff check`** vs **`ruff format`**
Dos comandos distintos y no intercambiables. `check` busca problemas de correctitud; `format` reescribe el aspecto. Uno puede pasar y el otro fallar.

**`--fix`** (del default: desactivado)
Aplica solo las correcciones que Ruff considera seguras. Las que podrían cambiar el comportamiento quedan fuera salvo que pidas `--unsafe-fixes`.

**`.ruff_cache/`** (del default)
Carpeta que Ruff crea para no reanalizar lo que no cambió. Va al `.gitignore`.
