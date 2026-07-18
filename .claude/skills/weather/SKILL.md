---
name: weather
description: Obtiene el clima actual o el pronóstico de cualquier ciudad consultando wttr.in por línea de comandos (sin API key). Úsala siempre que el usuario pregunte por el clima, la temperatura, el pronóstico, si va a llover, si hace frío/calor, o pida "el tiempo" de alguna ciudad o de su ubicación actual.
---

# Weather (clima vía wttr.in)

Esta skill obtiene datos de clima reales usando `curl` contra el servicio gratuito
[wttr.in](https://wttr.in), que no requiere API key ni registro. Todo se resuelve
con una llamada de red simple desde la terminal.

## Paso 1: Determinar la ubicación

wttr.in siempre necesita saber para qué lugar consultar el clima:

- Si el usuario ya mencionó una ciudad, país, o lugar en su mensaje (p. ej. "clima en
  Bogotá", "va a llover en Madrid?"), usa ese lugar directamente — no preguntes de nuevo.
- Si el usuario no especificó ningún lugar (p. ej. "¿cómo está el clima?", "dame el
  pronóstico"), usa **Pereira, Colombia** por defecto sin preguntar.
- Si el usuario pide explícitamente el clima "de donde estoy" o "de mi ubicación
  actual", puedes usar wttr.in sin nombre de ciudad (ver más abajo) — ese modo detecta
  la ubicación aproximada por IP del servidor donde corre `curl`, así que acláraselo
  al usuario si el resultado no parece corresponder a donde él está físicamente.

## Paso 2: Construir la consulta

Ejecuta con Bash. Reemplaza `<ciudad>` por el lugar (espacios como `+`, p. ej.
`Ciudad+de+Mexico`), o usa `Pereira,Colombia` si no se especificó ninguna ciudad
(ver Paso 1). Omite `<ciudad>` solo para autodetección por IP:

**Clima actual, resumen en una línea (rápido, ideal como respuesta directa):**
```bash
curl -s "wttr.in/<ciudad>?format=%l:+%c+%t+(sensación+%f),+humedad+%h,+viento+%w&lang=es"
```

**Reporte completo con pronóstico de 3 días (cuando el usuario pide "el pronóstico" o
más detalle):**
```bash
curl -s "wttr.in/<ciudad>?lang=es&T"
```
`&T` desactiva colores ANSI para que el texto se vea limpio en la respuesta.

**Solo hoy, sin los próximos días:**
```bash
curl -s "wttr.in/<ciudad>?lang=es&T&0"
```

Si el usuario pide unidades imperiales (Fahrenheit, millas), agrega `&u` a la URL;
si pide unidades métricas explícitas (poco común, ya es el default fuera de EE. UU.),
agrega `&m`.

## Paso 3: Presentar el resultado

- Para el formato de una línea, simplemente muéstraselo al usuario o intégralo en tu
  respuesta en prosa.
- Para el reporte completo (ASCII art), pégalo dentro de un bloque de código para que
  no se rompa el alineado.
- Si el usuario solo preguntó algo puntual ("¿hace frío?", "¿necesito paraguas?"),
  responde la pregunta directamente basándote en los datos, no solo vuelques el
  reporte crudo.

## Manejo de errores

- Si `curl` devuelve un error de red o timeout, informa al usuario que no se pudo
  contactar el servicio de clima y sugiere reintentar.
- Si la ciudad no se reconoce (wttr.in suele devolver un mensaje indicando que no
  encontró el lugar), pide al usuario que aclare o revise el nombre de la ciudad.
