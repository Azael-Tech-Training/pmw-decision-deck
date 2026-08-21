---
theme: default
title: PWM en Sistemas Automotrices
info: |
  ## Presentación sobre Modulación por Ancho de Pulso (PWM)
  Aplicaciones en sistemas automotrices modernos
  
  Autor: Gelacio Azael Fernandez Aldava
class: text-center
drawings:
  persist: false
transition: slide-left
comark: true
duration: 35min
---

<style>
@import './style.css';
</style>

# PWM en Sistemas Automotrices

Modulación por Ancho de Pulso: Del concepto a la práctica

<div class="mt-8 text-lg opacity-80">
Gelacio Azael Fernandez Aldava
</div>

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Presiona Espacio para continuar <carbon:arrow-right />
</div>

---
transition: fade-out
---

# ¿Qué es PWM?

**Modulación por Ancho de Pulso** (Pulse Width Modulation)

<div v-click>

## El problema original

Los sistemas automotrices antiguos usaban **resistencias en serie** para controlar la velocidad de motores y la intensidad de luces.

</div>

<div v-click>

## La solución moderna

**PWM** controla la potencia encendiendo y apagando rápidamente un componente electrónico (MOSFET).

</div>

<div v-click>

## Ventajas principales

- ✅ **Eficiencia energética** - Sin calor desperdiciado
- ✅ **Control preciso** - Ajuste fino de velocidad/intensidad
- ✅ **Menor peso** - Sin resistencias grandes
</div>

---
layout: center
---

# Simulador PWM Interactivo

<PwmSimulator />

---
layout: two-cols
---

# Conceptos Clave de PWM

<div v-click>

## Ciclo de Trabajo (Duty Cycle)

$$D = \frac{T_{on}}{T_{total}} \times 100\%$$

- **0%**: Siempre apagado
- **50%**: Encendido la mitad del tiempo
- **100%**: Siempre encendido (corriente directa)

</div>

<div v-click>

## Frecuencia (Hz)

$$f = \frac{1}{T_{total}}$$

- **Baja frecuencia** (< 100 Hz): Se nota el parpadeo
- **Alta frecuencia** (> 1 kHz): Suave y continuo
- **Automotriz típico**: 100 Hz - 20 kHz

</div>

::right::

<div class="mt-12 ml-4">

<QuizWidget 
  :questionNumber="1"
  question="¿La corriente de la pila (batería) es una PWM?"
  :options="[
    { text: 'Sí, siempre está encendida', correct: false },
    { text: 'No, es corriente directa (DC)', correct: true },
    { text: 'Solo cuando el motor está encendido', correct: false },
    { text: 'Depende del sistema eléctrico', correct: false }
  ]"
  explanation="La batería suministra corriente directa (DC), que es como PWM al 100% de ciclo de trabajo a 0 Hz."
/>

</div>

---

<QuizWidget 
  :questionNumber="2"
  question="Si una batería fuera PWM, ¿cuál sería el ciclo de trabajo?"
  :options="[
    { text: '50% - mitad encendida, mitad apagada', correct: false },
    { text: '100% - siempre encendida', correct: true },
    { text: '0% - siempre apagada', correct: false },
    { text: 'Varía según la carga', correct: false }
  ]"
  explanation="Ciclo de trabajo = 100% significa que el tiempo encendido es igual al tiempo total del ciclo. La batería entrega voltaje constante continuo."
/>

---

<QuizWidget 
  :questionNumber="3"
  question="¿Cuál sería la frecuencia (Hz) de una batería?"
  :options="[
    { text: '60 Hz como la corriente alterna', correct: false },
    { text: '1 kHz como PWM típico', correct: false },
    { text: '0 Hz - no hay ciclo, es voltaje constante', correct: true },
    { text: '100 Hz como PWM automotriz', correct: false }
  ]"
  explanation="Frecuencia = 0 Hz significa que no hay 'período' que medir. El voltaje nunca cambia, es continuo."
/>

---
transition: slide-up
---

# Interruptor Low-Side vs High-Side

<div class="grid grid-cols-2 gap-8 mt-6">

<div v-click>

## Low-Side (Lado Bajo)

- MOSFET entre **carga y tierra**
- Más común en automotrices
- Controla el lado de tierra del circuito
- **N-channel MOSFET**

</div>

<div v-click>

## High-Side (Lado Alto)

- MOSFET entre **fuente y carga**
- Menos común pero importante
- Controla el lado de voltaje del circuito
- **P-channel MOSFET**

</div>

</div>

<div v-click class="mt-6 p-4 bg-amber-900/30 rounded-lg border border-amber-500/30">

**¿Por qué importa?** La ubicación del interruptor afecta qué pasa con un **corto a tierra**.

</div>

---
layout: center
---

# Simulador de Circuitos

<CircuitSimulator variant="low-high" />

---
layout: center
---

<QuizWidget 
  :questionNumber="4"
  question="En un circuito low-side, ¿qué pasa si hay un corto a tierra antes del MOSFET?"
  :options="[
    { text: 'El fusible se quema inmediatamente', correct: false },
    { text: 'La carga se enciende permanentemente', correct: true },
    { text: 'El circuito se apaga completamente', correct: false },
    { text: 'No pasa nada, el corto no afecta', correct: false }
  ]"
  explanation="La carga se enciende permanentemente porque la corriente fluye directamente a tierra sin pasar por el interruptor. No se quema el fusible porque la carga limita la corriente."
/>

---
transition: slide-up
---

# Diagnósticos: Multímetro vs Osciloscopio

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## Multímetro (DMM)

- Mide **voltaje promedio**
- Útil para voltaje DC
- **Trampa**: No muestra la forma de onda
- Modos: V/Hz/Duty

</div>

<div v-click>

## Osciloscopio

- Muestra **forma de onda completa**
- Visualiza PWM en tiempo real
- Ajusta: timebase, volts/div, trigger
- **Esencial para diagnóstico PWM**

</div>

</div>

<div v-click class="mt-6 p-4 bg-red-900/30 rounded-lg border border-red-500/30">

**¡Cuidado!** Un multímetro en modo voltaje puede mostrar 6V en un PWM al 50% de 12V. ¡No significa que haya un problema!

</div>

---

<QuizWidget 
  :questionNumber="5"
  question="Si mides 6V con un multímetro en un circuito de 12V con PWM, ¿el circuito está funcionando correctamente?"
  :options="[
    { text: 'Sí, siempre está bien', correct: false },
    { text: 'No, hay un problema', correct: false },
    { text: 'Depende del ciclo de trabajo', correct: true },
    { text: 'Solo si la frecuencia es correcta', correct: false }
  ]"
  explanation="Si el PWM está al 50% de ciclo de trabajo, 6V es correcto (12V × 0.5 = 6V promedio). Necesitas un osciloscopio para confirmar la forma de onda correcta."
/>

---
transition: slide-up
---

# Cortos a Tierra y Fusibles

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## Ley de Ohm en Fallas

$$I = \frac{V}{R}$$

- Sin carga: **R ≈ 0Ω**
- Corriente se **dispara** (240A+)
- El fusible protege el circuito

</div>

<div v-click>

## Calentamiento Joule

$$P = I^2 \times R$$

- 240A × 0.02Ω = **1,152W**
- Genera calor extremo
- **Fusible se funde** intencionalmente

</div>

</div>

<div v-click class="mt-6 p-4 bg-green-900/30 rounded-lg border border-green-500/30">

**Dato clave**: Un fusible de 20A puede manejar 20A continuamente, pero se funde con 240A en milisegundos.

</div>

---
layout: center
---

# Simulador de Cortocircuitos

<CircuitSimulator variant="short-circuit" />

---
layout: center
---

<QuizWidget 
  :questionNumber="6"
  question="¿Por qué un corto a tierra en el lado low-side NO quema el fusible, pero un corto en el lado high-side SÍ lo quema?"
  :options="[
    { text: 'Porque low-side tiene más resistencia', correct: false },
    { text: 'Porque en low-side la carga limita la corriente', correct: true },
    { text: 'Porque high-side tiene un fusible más grande', correct: false },
    { text: 'Porque la batería protege el low-side', correct: false }
  ]"
  explanation="En low-side, el corto está después de la carga. La carga limita la corriente. En high-side, el corto está antes de la carga, bypassing la resistencia."
/>

---
transition: slide-up
---

# Lógica de Control del ECU

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## ECU = Computadora del Motor

- Recibe datos de **sensores**
- Calcula el **ciclo de trabajo** apropiado
- Envía señal PWM al actuador
- **Mantiene frecuencia constante**

</div>

<div v-click>

## Retroalimentación (Closed-Loop)

- Sensor mide condición actual
- ECU compara con **valor objetivo**
- Ajusta PWM para corregir diferencia
- **Ejemplo**: Temperatura del refrigerante

</div>

</div>

<div v-click class="mt-6 p-4 bg-purple-900/30 rounded-lg border border-purple-500/30">

**Concepto clave**: El ECU **nunca cambia la frecuencia**, solo ajusta el ciclo de trabajo para controlar la potencia.

</div>

---
layout: center
---

# Simulador ECU en Loops Cerrados

<EcuSimulator />

---

<QuizWidget 
  :questionNumber="7"
  question="¿Por qué el ECU mantiene la frecuencia constante y solo cambia el ciclo de trabajo?"
  :options="[
    { text: 'Porque es más fácil de programar', correct: false },
    { text: 'Porque la frecuencia es óptima para el componente', correct: true },
    { text: 'Porque cambiar frecuencia es imposible', correct: false },
    { text: 'Porque el consumidor lo exige', correct: false }
  ]"
  explanation="La frecuencia está diseñada para ser óptima para el componente (evitar ruido, vibración). Solo necesitamos variar la potencia, que se controla con el ciclo de trabajo."
/>

---
transition: slide-up
---

# Analogía de la Manguera de Jardín

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## Sistema de Agua

- **Llave de paso** = MOSFET
- **Manguera** = Cable eléctrico
- **Aspersor** = Carga (ventilador)
- **Drenaje** = Tierra

</div>

<div v-click>

## Circuito Eléctrico

- **Batería** = Fuente de agua
- **MOSFET** = Válvula de control
- **Ventilador** = Carga
- **Tierra** = Regreso al tanque

</div>

</div>

<div v-click class="mt-6 p-4 bg-cyan-900/30 rounded-lg border border-cyan-500/30">

**Regla del Circuito en Serie**: La misma corriente fluye por **todos** los componentes en serie. Abrir la válvula en cualquier punto detiene todo el flujo.

</div>

---
layout: center
---

# Simulador Manguera vs Circuito

<FlowComparison />

---
transition: slide-up
---

# Cómo el ECU Cambia el Ciclo de Trabajo

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## El MOSFET es Binario

- **100% ON** o **100% OFF**
- No hay estado "medio"
- El control es por **tiempo**

</div>

<div v-click>

## Modulación de Tiempo

- **Tiempo encendido** vs **Tiempo apagado**
- ECU controla duración de cada pulso
- Motor ventilador **promedia** por inercia

</div>

</div>

<div v-click class="mt-6 p-4 bg-orange-900/30 rounded-lg border border-orange-500/30">

**Analogía**: Es como encender y apagar rápidamente una luz. Si lo haces lo suficientemente rápido, parece que la luz está "medio" encendida.

</div>

---
layout: center
---

# Simulador de Modulación de Tiempo

<TimeModulation />

---

<QuizWidget 
  :questionNumber="8"
  question="Si un MOSFET solo puede estar 100% encendido o 100% apagado, ¿cómo logra el ECU 'reducir' la velocidad del ventilador?"
  :options="[
    { text: 'Usando resistencias internas', correct: false },
    { text: 'Reduciendo el voltaje de la batería', correct: false },
    { text: 'El ventilador promedia por inercia mecánica', correct: true },
    { text: 'Cambiando la frecuencia del PWM', correct: false }
  ]"
  explanation="El ECU reduce el tiempo encendido por ciclo. El ventilador promedia físicamente debido a su inercia mecánica - no puede acelerar/frenar tan rápido como el MOSFET conmuta."
/>

---
transition: slide-up
---

# Aplicación Práctica: Bomba de Combustible

<div class="grid grid-cols-2 gap-6 mt-6">

<div v-click>

## Sistema de Retorno

- **Bomba constante** a full velocidad
- Combustible excesivo regresa al tanque
- Mayor consumo energético
- Más desgaste de la bomba

</div>

<div v-click>

## Sistema Sin Retorno (PWM)

- Bomba **modulada** según demanda
- Solo envía combustible necesario
- **Menor consumo** de energía
- **Menor desgaste** de la bomba

</div>

</div>

<div v-click class="mt-6 p-4 bg-emerald-900/30 rounded-lg border border-emerald-500/30">

**Beneficios**: Reducción de emisiones EVAP, menor drenaje de batería, vida útil延长ada de la bomba.

</div>

---
layout: center
---

# Simulador de Bomba de Combustible

<FuelPumpSimulator />

---

<QuizWidget 
  :questionNumber="9"
  question="En un sistema de bomba de combustible sin retorno, ¿qué sensores usa el ECU para determinar el ciclo de trabajo correcto?"
  :options="[
    { text: 'Solo el sensor de temperatura', correct: false },
    { text: 'Sensor de presión del riel (FRP) principalmente', correct: true },
    { text: 'Solo la posición del acelerador', correct: false },
    { text: 'No usa sensores, es fijo', correct: false }
  ]"
  explanation="Principalmente el sensor de presión del riel de combustible (FRP). También puede usar posición del acelerador (TPS), temperatura del motor (ECT), y carga del motor (MAF/MAP)."
/>

---

<QuizWidget 
  :questionNumber="10"
  question="¿Por qué el sistema usa dos frecuencias diferentes (80 Hz y 20 kHz) en la bomba de combustible?"
  :options="[
    { text: 'Es un error de diseño', correct: false },
    { text: 'Para ahorrar energía', correct: false },
    { text: 'El PCM al FPDM usa 80 Hz, el FPDM a la bomba usa 20 kHz', correct: true },
    { text: 'Ambas frecuencias son iguales', correct: false }
  ]"
  explanation="El PCM al FPDM usa 80 Hz (señal de control lenta, robusta). El FPDM a la bomba usa 20 kHz (drive de alta frecuencia para operación suave y eficiente del motor DC)."
/>

---
transition: slide-up
---

# Resumen de Conceptos Clave

<div class="grid grid-cols-2 gap-4 mt-6 text-sm">

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **PWM** = Encendido/apagado rápido para controlar potencia promedio

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Ciclo de trabajo** = % del tiempo encendido por ciclo

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Frecuencia** = Cuántas veces por segundo se repite el ciclo

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Low-side** = MOSFET entre carga y tierra (más común)

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **High-side** = MOSFET entre fuente y carga

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Osciloscopio** = Herramienta esencial para diagnóstico PWM

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Closed-loop** = ECU ajusta PWM basado en retroalimentación de sensores

</div>

<div v-click class="p-3 bg-gray-800/50 rounded">

✅ **Frecuencia constante** = Solo el ciclo de trabajo cambia

</div>

</div>

---
layout: end
---

# ¡Gracias!

<div class="text-xl mt-8 opacity-80">

Gelacio Azael Fernandez Aldava

</div>

<div class="mt-8">

## Recursos para Aprender Más

- **PicoScope**: Pruebas guiadas de diagnóstico automotriz
- **ScannerDanner**: Libro y canal de YouTube sobre diagnósticos
- **Halderman**: Textbook de electricidad automotriz
- **Foros**: r/diy电子s, r/mechanicadvice, Speeduino

</div>

<div class="mt-8 text-sm opacity-60">

Presentación creada con Slidev • PWM en Sistemas Automotrices

</div>
