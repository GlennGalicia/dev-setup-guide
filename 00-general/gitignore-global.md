# .gitignore global

- [.gitignore global](#gitignore-global)
  - [¿Qué es?](#qué-es)
  - [Precedencias](#precedencias)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#gitignore-global)

Un `.gitignore` que aplica a **todos** los repositorios de tu máquina. No se commitea y nadie más lo hereda.

**¿Para qué sirve en la práctica?**

Separar tu basura de la del proyecto. Si el archivo lo genera tu sistema operativo o tu editor, va aquí. Si lo genera el proyecto, va en el `.gitignore` del repo — así el repositorio se explica solo y no obliga a nadie a configurar su máquina.

## Precedencias
[↑ Volver arriba](#gitignore-global)

De mayor a menor prioridad:

1. `.gitignore` del proyecto (gana el más cercano al archivo)
2. `.git/info/exclude` (local, no se commitea)
3. `.gitignore` global

* El global es el de **menor** prioridad. Un `!patron` en el `.gitignore` del proyecto puede revertir lo que el global ignora.
* Ignorar un archivo no lo destrackea. Si ya estaba commiteado, sigue ahí.

## Configuración
[↑ Volver arriba](#gitignore-global)

Crea el archivo:

```bash
touch ~/dotfiles/.gitignore_global
```

Regístralo en Git:

```bash
git config --global core.excludesFile ~/dotfiles/.gitignore_global
```

En PowerShell:

```powershell
git config --global core.excludesFile "$env:USERPROFILE\dotfiles\.gitignore_global"
```

Ahora, abre el archivo y coloca:

```ignore
# ===============================================================
# Global gitignore — machine and editor noise only
# ===============================================================
# Scope rule: if it comes from YOUR machine or YOUR editor, it
# belongs here. If the PROJECT generates it, it belongs in that
# project's .gitignore so the repo stays self-describing.

# ---------------------------------------------------------------
# macOS
# ---------------------------------------------------------------
.DS_Store
.AppleDouble
.LSOverride
._*

# ---------------------------------------------------------------
# Windows
# ---------------------------------------------------------------
[Tt]humbs.db
[Ee]hthumbs.db
[Dd]esktop.ini
$RECYCLE.BIN/
*.lnk

# ---------------------------------------------------------------
# Editors & IDEs
# ---------------------------------------------------------------
# NOTE: .vscode/ is deliberately NOT here. Project-level VSCode
# config (settings.json, extensions.json, launch.json) is a team
# convention and must be committable.
*.swp
*.swo
*~
.idea/
*.iml
.azuredatastudio/

# ---------------------------------------------------------------
# Temp & merge artifacts
# ---------------------------------------------------------------
*.tmp
*.temp
*.bak
*.orig

# ---------------------------------------------------------------
# Office lock files
# ---------------------------------------------------------------
~$*.docx
~$*.xlsx
~$*.pptx

# ---------------------------------------------------------------
# Per-user tool state
# ---------------------------------------------------------------
**/.claude/settings.local.json
```

## Comandos útiles
[↑ Volver arriba](#gitignore-global)

**Máquina nueva:**

Los dos comandos de arriba y listo. Aplica a todos los repos, incluidos los ya clonados.

**Archivo ya commiteado por error:**

Agregarlo al `.gitignore` no lo saca del repositorio. Hay que destrackearlo:

```bash
git rm --cached .DS_Store
git commit -m "chore: untrack .DS_Store"
```

Para una carpeta completa:

```bash
git rm -r --cached .idea/
```

`--cached` lo quita del repositorio pero lo conserva en tu disco. Sin esa bandera, lo borra.

## Verificación
[↑ Volver arriba](#gitignore-global)

Confirma que Git lo está leyendo:

```bash
git config --global core.excludesFile
```

Averigua qué regla está ignorando un archivo:

```bash
git check-ignore -v .DS_Store
```

Responde con el archivo, la línea y el patrón que hizo match. Si no responde nada, ese archivo no se está ignorando.

## Diccionario
[↑ Volver arriba](#gitignore-global)

**`*.tmp`**
El asterisco sustituye cualquier texto. Matchea a cualquier profundidad, porque el patrón no lleva barra.

**`.idea/`**
La barra final limita el match a carpetas. Sin ella, también atraparía un archivo llamado `.idea`.

**`**/.claude/settings.local.json`**
Un patrón que contiene barra se ancla a la raíz del repositorio. El `**/` lo libera para que matchee a cualquier profundidad.

**`[Tt]humbs.db`**
Clase de caracteres: matchea `Thumbs.db` y `thumbs.db`. Necesario porque los patrones distinguen mayúsculas en mac y Linux.

**`!patron`**
Negación. Excluye algo de una regla anterior, como `!.env.example` después de ignorar `.env*`.

**`$RECYCLE.BIN/`**
La papelera de Windows. Aparece en la raíz de unidades externas, y termina dentro de tu repo si trabajas desde una USB.
