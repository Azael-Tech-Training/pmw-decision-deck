---
theme: seriph
background: https://cover.sli.dev
title: PWM-002 — Vite y salida estática
info: |
  Presentación de decisión del proyecto PWM Learning Site:
  por qué elegimos Vite con salida completamente estática.
author: Joel Ramirez Barboza
keywords: PWM, Vite, salida estática, arquitectura, ADR
transition: slide-left
---

# PWM-002: Vite y salida estática

### La decisión de tooling y modelo de renderizado del sitio de aprendizaje PWM

<div class="pt-2 text-sm opacity-60">
Joel Ramirez Barboza · agosto 2026
</div>

<!--
Objetivo de la sesión: explicar y defender la decisión ya aceptada en ADR-0001.
No es una votación: la decisión está tomada; vengo a mostrar el razonamiento y el alcance.
Duración objetivo: 10–15 minutos.
-->

---

# Contexto

Proyecto: sitio de aprendizaje de **PWM en sistemas automotrices**

Hoy en el repositorio:

- **8 lecciones** HTML escritas a mano, con widgets interactivos en JS vanilla
- Un **deck interactivo** ya construido en Slidev
- Sin bundler, sin índice, sin navegación compartida

La meta: publicarlo todo como un **sitio web estático**

Antes de construir había que decidir **con qué herramientas** → eso fue PWM-002

<!--
Estado verificado del repositorio: lecciones 0001–0008, stylesheet compartido,
Teaching Deck con 8 componentes Vue. PWM-002 resolvió bundler + modelo de renderizado.
-->

---

# El problema

Sin bundler no hay código compartido:

- navegación, estilos y widgets se **duplican** entre ~10 páginas

PWM-005 traerá landing page, índice y navegación compartida — justo el dolor que un bundler resuelve

Elegir mal hoy = **migrar a mitad de proyecto**

Dos preguntas distintas:

1. ¿Qué **bundler**?
2. ¿Qué **modelo de renderizado**?

<!--
Enfatizar: son dos preguntas independientes y así se investigaron.
El error caro sería adoptar un framework dinámico por defecto.
-->

---
layout: center
class: text-center
---

# La decisión

<div class="text-4xl font-bold leading-relaxed py-4">

**Vite como bundler.**
**Salida completamente estática.**

</div>

ADR-0001 · aceptada el 2026-08-20, tras investigación con fuentes primarias

Se decide *antes* de PWM-005 a propósito: **construir sobre la herramienta**, no migrar hacia ella

<!--
Momento clave de la presentación. Pausa después de leer la decisión.
Explicar el orden invertido: normalmente se decide arquitectura primero;
aquí se fijó el toolchain antes porque PWM-005 lo necesita como base.
-->

---

# Qué NO es

Cuatro términos que suelen confundirse — y una regla del proyecto:

| Término | Qué significa | ¿Nos aplica? |
| --- | --- | --- |
| **Salida estática** | Todo el HTML se ensambla antes del despliegue; cada visitante recibe los mismos archivos desde el CDN | ✅ **nuestra decisión** |
| SSG | Una herramienta genera el HTML en build desde plantillas o contenido | ❌ reservado para un posible futuro (PWM-005) |
| SPA | Un solo documento reescrito por JavaScript | ❌ rechazado |
| SSR | HTML renderizado por petición en un servidor | ❌ rechazado |

<div class="pt-6">
Regla del proyecto: decimos <b>salida estática</b>, no «SSG».
</div>

<!--
Diapositiva de vocabulario. La palabra SSG queda RESERVADA en este proyecto:
solo será correcta si PWM-005 adopta un generador. Evita malentendidos futuros.
-->

---

# Alternativas evaluadas

| Alternativa | Veredicto | Motivo |
| --- | --- | --- |
| Rspack / Rsbuild | Rechazada | Sus ventajas son compatibilidad webpack y Module Federation; no tenemos legado webpack |
| esbuild solo | Rechazada | Sin live-reload por diseño; reconstruiríamos lo que Vite da gratis |
| Parcel | Rechazada | Capaz, pero sin arrastre de ecosistema frente a Vite |
| Next.js / Nuxt / SvelteKit / Vike | Rechazadas | Maquinaria para apps dinámicas o boilerplate DIY desproporcionado |
| **Astro** | **Diferida** | No rechazada: sigue viva como opción real de PWM-005 |
| SPA como modelo | Rechazado | Complejidad de routing y SEO sin beneficio en contenido estático |

<!--
Si preguntan por Astro: NO fue descartada. Cumple el invariante de la decisión
y vuelve si el formato de lección cae del lado de Markdown (ver diapositiva 8).
-->

---

# Por qué Vite

1. La salida requerida son **archivos estáticos puros** — `vite build` produce exactamente eso, hosteable donde sea
2. Nuestra interactividad **no necesita servidor** — widgets autocontenidos con estado local
3. **Un solo modelo mental** — Slidev ya corre sobre Vite: deck y sitio comparten toolchain
4. **Accesible para el equipo** — una config mínima; las lecciones siguen siendo HTML/CSS/JS
5. **Bajo riesgo de obsolescencia** — Vite 8 unificó Rolldown (motor en Rust) y el ecosistema consolidó sobre él: Slidev, Astro, Nuxt y SvelteKit

<!--
Razón 3 es la más estratégica para nosotros: ya mantenemos un proyecto Slidev.
Razón 5: la migración a Rolldown preservó la compatibilidad de plugins Rollup.
-->

---

# Por qué aún no SSG

- Ninguna herramienta genera nuestro HTML **hoy**: las lecciones ya están escritas a mano
- Generar HTML en build solo cobra sentido si cambia el **formato de lección** (HTML crudo vs Markdown) — pregunta abierta de PWM-005
- **Astro sigue siendo candidata real** si el formato cae del lado de Markdown
- Decidir SSG ahora sería especular sobre una decisión que aún no tomamos

<!--
Aquí se defiende la disciplina de alcance: PWM-002 fija toolchain y rendering,
no arquitectura. Mantener SSG fuera evita comprometer PWM-005 prematuramente.
-->

---
layout: two-cols
---

# Consecuencias y trade-offs

::left::

### Ganamos

- **Neutralidad de framework**: Vite es la intersección de todos los desenlaces de PWM-003 — React vía plugin, web components nativos, Vue/Slidev igual
- **Despliegue simple**: cualquier servidor de archivos estáticos sirve; sin infraestructura de routing

::right::

### Aceptamos

- Motor **Rolldown nuevo** (2026-03): exposición mínima por construcción, versión fijada exacta, bajar a Vite 7 es un cambio de una línea
- **No resuelve** los problemas actuales del README (alineación de diagramas, índice): eso es PWM-005/PWM-006

<!--
Honestidad sobre trade-offs da credibilidad. El riesgo Rolldown está acotado:
MPA simple, JS/CSS vanilla, sin plugins custom; los fallos aparecen en build, nunca en silencio.
-->

---

# Lo diferido

| Ticket | Qué decide | Estado |
| --- | --- | --- |
| PWM-003 | Componentes compartidos sitio ↔ deck (¿React?) | abierto |
| PWM-004 | Hosting y despliegue | abierto |
| PWM-005 | Arquitectura del sitio: MPA pura vs Astro · formato de lección · i18n | abierto |

Nota: las configs Netlify/Vercel existentes son artefactos SPA del Teaching Deck — **no se reúsan para el Website**

<!--
Dejar claro que nada se perdió: todo lo no decidido tiene ticket propio.
La nota de configs evita que alguien copie el vercel.json del deck al sitio.
-->

---

# Evidencia: el experimento OpenCode

El **Teaching Deck** ya demuestra el flujo de trabajo:

- Presentación interactiva completa: simulador PWM, quiz, contadores — **8 componentes Vue**
- Construida con OpenCode y skills de ingeniería, sobre Slidev → que corre sobre **Vite**
- La misma herramienta que hoy estandarizamos

*(demo en vivo si el tiempo alcanza)*

<!--
Este deck también cuenta como evidencia: se creó con OpenCode siguiendo
grilling → domain-modeling → implement. Si hay tiempo, abrir el Teaching Deck
y mostrar el simulador PWM en vivo (~2 min máximo).
-->

---
layout: center
---

# Siguientes pasos

**Decidido y registrado**: Vite + salida completamente estática — `docs/adr/0001-vite-static-output.md`

El mapa continúa:

- **PWM-003** — componentes compartidos sitio ↔ deck
- **PWM-004** — hosting y despliegue
- **PWM-005** — arquitectura del sitio (MPA vs Astro · formato de lección · i18n)

Único detonante de revisión: que PWM-003 identifique un requisito que **Vite no soporte**

### Preguntas

*Gracias — la investigación completa vive en `docs/RESEARCH-TOOLING-RENDERING.md`*

<!--
Cerrar agradeciendo. La decisión no se reabre salvo por el detonante de revisión del ADR.
Enlaces útiles si preguntan: doc de investigación (docs/RESEARCH-TOOLING-RENDERING.md)
y ADR-0001 en el repositorio del proyecto.
-->
