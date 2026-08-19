# .editorconfig

- [.editorconfig](#editorconfig)
  - [¿Qué es?](#qué-es)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#editorconfig)

Define indentación, codificación y finales de línea para todo el repositorio. Lo lee el **editor**, no una herramienta que ejecutes.

**¿Para qué sirve en la práctica?**

Que un archivo se vea igual sin importar quién lo abra ni en qué editor. Aplica a todos los archivos del repo, incluidos los que Prettier no toca: `.md`, `.yml`, `.env`, `.py`.

## Precedencias
[↑ Volver arriba](#editorconfig)

De mayor a menor prioridad:

1. `.prettierrc`
2. `.editorconfig`
3. Defaults del editor

* Prettier traduce solo cinco propiedades: `indent_style`, `indent_size`, `max_line_length`, `end_of_line` y `quote_type`. El resto las ignora.
* Si una opción está en los dos archivos, gana `.prettierrc`. No la repitas.
* Si hay varios `.editorconfig` en carpetas anidadas, gana el más cercano al archivo. La búsqueda se detiene en el que tenga `root = true`.

## Instalación
[↑ Volver arriba](#editorconfig)

No se instala nada en el proyecto, pero el editor necesita soportarlo.

* **VS Code** — instala la extensión `EditorConfig for VS Code`. Sin ella el archivo se ignora por completo.
* **WebStorm / PhpStorm** — soporte nativo, no hace falta nada.

## Configuración
[↑ Volver arriba](#editorconfig)

Crea el archivo `.editorconfig`, raíz de tu proyecto:

```bash
echo > .editorconfig
```

Ahora, abre el archivo y coloca:

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[*.py]
indent_size = 4

[Makefile]
indent_style = tab
```

Un solo archivo cubre todos los stacks. Las secciones por extensión hacen el trabajo, no hace falta un `.editorconfig` por lenguaje.

## Comandos útiles
[↑ Volver arriba](#editorconfig)

**Proyecto nuevo:**

Nada más. El editor lo aplica desde el siguiente archivo que abras.

**Proyecto existente:**

El `.editorconfig` solo afecta lo que edites de aquí en adelante; no reformatea lo que ya está. Para emparejar todo el proyecto de una vez:

```bash
pnpm format
```

## Verificación
[↑ Volver arriba](#editorconfig)

Abre cualquier archivo y presiona Tab. Deben insertarse 2 espacios, no un tabulador.

Para revisar todo el proyecto sin instalar nada:

```bash
npx editorconfig-checker
```

## Diccionario
[↑ Volver arriba](#editorconfig)

**`root = true`**
Marca este archivo como el último. Sin él, el editor sigue buscando `.editorconfig` en carpetas superiores y puede heredar reglas ajenas al proyecto.

**`charset = utf-8`**
Codificación de los archivos.

**`end_of_line = lf`**
Final de línea. Coincide con el `eol=lf` del `.gitattributes` y con el default de Prettier.

**`insert_final_newline = true`**
Deja una línea vacía al final del archivo. Sin ella, Git marca la última línea como modificada al agregar contenido.

**`trim_trailing_whitespace = true`**
Borra los espacios sobrantes al final de cada línea.

**`indent_style = space`**
Espacios en vez de tabuladores.

**`indent_size = 2`**
Dos espacios por nivel de indentación.

**`[*.md]`**
En Markdown, dos espacios al final de una línea significan salto de línea. Por eso aquí se desactiva `trim_trailing_whitespace`: si no, el editor los borra y el formato se rompe.

**`[Makefile]`**
Los Makefiles exigen tabuladores. Con espacios simplemente no funcionan.
