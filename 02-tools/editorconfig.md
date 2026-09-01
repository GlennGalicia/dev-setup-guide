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

Que un archivo se vea igual sin importar quién lo abra ni en qué editor. Aplica a todos los archivos del repo, incluidos los que ningún formateador cubre: `.env`, `Dockerfile`, `.sql`, `.ini`.

## Precedencias
[↑ Volver arriba](#editorconfig)

De mayor a menor prioridad:

1. La configuración del formateador del proyecto, si lo hay
2. `.editorconfig`
3. Defaults del editor

* Si una opción está definida en los dos, gana el formateador. No la repitas.
* No todos los formateadores leen `.editorconfig`, y los que lo hacen traducen solo algunas propiedades. Cuáles, lo dice el doc de cada herramienta.
* Si hay varios `.editorconfig` en carpetas anidadas, gana el más cercano al archivo. La búsqueda se detiene en el que tenga `root = true`.

## Instalación
[↑ Volver arriba](#editorconfig)

No se instala nada en el proyecto, pero el editor necesita soportarlo.

* **VS Code** — instala la extensión `EditorConfig for VS Code`. Sin ella el archivo se ignora por completo.
* **WebStorm / PhpStorm** — soporte nativo, no hace falta nada.

## Configuración
[↑ Volver arriba](#editorconfig)

Crea el archivo `.editorconfig` en la raíz de tu proyecto, ya con su contenido:

```bash
cat > .editorconfig << 'EOF'
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
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

Un solo archivo cubre todos los stacks. Las secciones por extensión hacen el trabajo, no hace falta un `.editorconfig` por lenguaje.

## Comandos útiles
[↑ Volver arriba](#editorconfig)

**Proyecto nuevo:**

Nada más. El editor lo aplica desde el siguiente archivo que abras.

**Proyecto existente:**

El `.editorconfig` solo afecta lo que edites de aquí en adelante; no reformatea lo que ya está.

Para emparejar un proyecto existente necesitas un formateador, y cuál sea depende del stack: lo indica la checklist en `01-stacks`.

## Verificación
[↑ Volver arriba](#editorconfig)

Abre cualquier archivo y presiona Tab. Deben insertarse 2 espacios, no un tabulador.

En un `.md`, escribe dos espacios al final de una línea y guarda. Deben seguir ahí: es lo que hace la sección `[*.md]`.

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
