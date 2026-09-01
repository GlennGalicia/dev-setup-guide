# <Herramienta>

- [](#)
  - [¿Qué es?](#qué-es)
  - [Lenguajes que soporta](#lenguajes-que-soporta)
  - [Precedencias](#precedencias)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Comandos útiles](#comandos-útiles)
  - [Verificación](#verificación)
  - [Diccionario](#diccionario)

## ¿Qué es?
[↑ Volver arriba](#herramienta)

<Qué hace y qué NO hace. Una o dos líneas.>

**¿Para qué sirve en la práctica?**

<Qué problema real te quita de encima.>

## Lenguajes que soporta
[↑ Volver arriba](#herramienta)

<Tabla o lista de extensiones.>

**No soporta:**

* <lo que hay que resolver con otra herramienta>

## Precedencias
[↑ Volver arriba](#herramienta)

De mayor a menor prioridad:

1. <archivo que manda>
2. <archivo que rellena huecos>
3. <defaults>

* <con qué otra herramienta choca y cómo se resuelve>

## Instalación
[↑ Volver arriba](#herramienta)

En terminal, dentro del proyecto:

```bash
<comando de instalación>
```

Verifica que quedó instalado:

```bash
<comando de versión>
```

## Configuración
[↑ Volver arriba](#herramienta)

Crea el archivo `<archivo>` en la raíz de tu proyecto, ya con su contenido:

```bash
cat > <archivo> << 'EOF'
<contenido>
EOF
```

> **`>` trunca.** Si el archivo ya existía con contenido, este comando lo reemplaza por completo sin preguntar.

<Si el contenido incluye `$`, el delimitador entrecomillado (`'EOF'`) es lo que
evita que la shell lo expanda. Mantenlo siempre.>

## Comandos útiles
[↑ Volver arriba](#herramienta)

Agrega los scripts a tu `package.json`:

```json
"scripts": {}
```

**Proyecto nuevo:**

```bash
<comando>
```

**Proyecto existente:**

Primero revisa sin tocar nada:

```bash
<comando de revisión>
```

Si el resultado te convence:

```bash
<comando que aplica cambios>
```

## Verificación
[↑ Volver arriba](#herramienta)

```bash
<comando de check>
```

<Cómo se ve un resultado correcto.>

## Diccionario
[↑ Volver arriba](#herramienta)

**`<opción>`**
<Qué hace, en una línea.>

```<lenguaje>
<ejemplo mínimo>
```
