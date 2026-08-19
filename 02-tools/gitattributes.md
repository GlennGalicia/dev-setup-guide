# .gitattributes

- [.gitattributes](#gitattributes)
  - [¿Qué es?](#qué-es)
  - [Precedencias](#precedencias)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#gitattributes)

Le dice a Git cómo tratar cada tipo de archivo: cuáles son texto, cuáles son binarios, y con qué final de línea guardarlos.

**¿Para qué sirve en la práctica?**

Evita que un proyecto compartido entre mac y Windows genere diffs donde el 100% de las líneas aparecen modificadas solo por el cambio de CRLF a LF.

## Precedencias
[↑ Volver arriba](#gitattributes)

De mayor a menor prioridad:

1. `.gitattributes` del proyecto
2. `core.autocrlf` (config global de Git)

* `.gitattributes` se commitea, así que la regla viaja con el repo. Quien lo clone la obtiene sin configurar nada.
* `core.autocrlf` solo aplica a la máquina de quien lo configuró. Por eso no basta en equipo.

## Configuración
[↑ Volver arriba](#gitattributes)

No se instala nada, Git ya lo soporta.

Crea el archivo `.gitattributes`, raíz de tu proyecto:

```bash
echo > .gitattributes
```

Ahora, abre el archivo y coloca:

```
# Normalize line endings to LF in the repository
* text=auto eol=lf

# Binary files: never modify
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.webp binary
*.ico binary
*.woff binary
*.woff2 binary
*.ttf binary
*.pdf binary

# SVG is XML, treat it as text for readable diffs
*.svg text
```

## Comandos útiles
[↑ Volver arriba](#gitattributes)

**Proyecto nuevo:**

No hace falta nada más. Las reglas aplican desde el primer commit.

**Proyecto existente:**

Los archivos ya commiteados conservan sus finales de línea viejos. Hay que renormalizarlos:

```bash
git add --renormalize .
git commit -m "chore: normalize line endings"
```

## Verificación
[↑ Volver arriba](#gitattributes)

Consulta qué reglas aplican a un archivo:

```bash
git check-attr -a index.html
```

Debe responder `text: auto` y `eol: lf`.

## Diccionario
[↑ Volver arriba](#gitattributes)

**`* text=auto`**
Aplica a todos los archivos. Git detecta cuáles son texto y normaliza sus finales de línea al guardarlos en el repositorio.

**`eol=lf`**
Fuerza LF también en tu carpeta de trabajo, no solo dentro del repo. Coincide con el `endOfLine: "lf"` que Prettier usa por defecto.

**`binary`**
Marca el archivo como binario: Git no toca su contenido ni intenta mostrar diffs de texto.

**`text`**
Fuerza el tratamiento como texto, aunque Git lo hubiera detectado distinto. Es el caso de `.svg`, que es XML.
