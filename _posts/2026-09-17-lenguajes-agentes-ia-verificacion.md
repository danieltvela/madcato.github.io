---
layout:     post
title:      "El eje no es scripting contra compilado: es el coste de verificar"
subtitle:   "Lenguajes para agentes de IA: lo que el debate de agosto de 2026 dejó claro"
date:       2026-09-17 22:20:00
author:     "Daniel Vela"
og_image:   "/img/post-bg-01.jpg"
locale:     es
lang-ref:   lenguajes-agentes-ia-verificacion
---

Toda mi vida he llevado en la cabeza un equilibrio que parecía permanente: los lenguajes de scripting (Ruby, Python, JavaScript) compran velocidad de iteración y pagan con errores que solo aparecen en runtime; los compilados (C, Java, Rust) compran garantías y pagan con un bucle lento — esperas a que compile para saber si algo funciona. Xcode Previews fue el intento más serio de darle al mundo compilado el bucle del scripting: cambias el aspecto de la UI sin compilar ni lanzar el depurador. Cuando funciona es una delicia. Cuando no, vuelves a esperar.

Con la programación agéntica la sospecha es natural: si el agente vive de iterar, los lenguajes de scripting deberían arrasar, porque su bucle de verificación es inmediato. Yo también lo sospechaba. Después de seguir el debate real de 2026, mi conclusión es otra: **la dicotomía scripting/compilado está mal planteada**. El eje correcto es el coste de verificar — y ese eje partió la dicotomía por la mitad.

## El debate se formalizó en agosto de 2026

El 11 de agosto el equipo de Go publicó [Why Go is an Ideal Language for AI-Assisted Software Engineering](https://developers.googleblog.com/why-go-is-an-ideal-language-for-ai-assisted-software-engineering/). La premisa, que casi nadie discute: cuando un agente genera cientos de líneas en segundos, **el cuello de botella deja de ser escribir código y pasa a ser verificarlo**. Todo el análisis útil de 2026 gira alrededor de ese traslado.

El argumento de Google: `gofmt` da uniformidad total (el código generado se lee igual que el humano), el sistema de tipos estáticos mata las APIs alucinadas en compile time y — el detalle decisivo — Go compila órdenes de magnitud más rápido que Java, C# o Rust, así que el agente hace generate → compile → fix → recompile sin esperar. Su dato (declarado, no medido en público): los agentes producen Go válido al primer intento el 95% de las veces.

Hacker News respondió con cientos de comentarios: sin benchmarks, dijeron unos; y los data races de la auditoría de Uber (46 millones de líneas, más de 2.000 races, 790 parches) descalifican la tesis, dijeron otros. El comentario que para mí ganó el debate apuntaba a un sitio distinto: el compilador quisquilloso es *ideal* para un LLM — martillearlo con tokens es mejor estrategia que deducir dónde fallará en runtime y atraparlo con tests. Los tokens son baratos; las sorpresas en runtime, no.

## La contra-tesis que deshace al scripting puro

Mi hipótesis asumía que la espera era el único coste del compilado. Hay otro, invertido: el compilador es el mejor detector de alucinaciones que existe, y el scripting lo delega todo al runtime.

El benchmark académico que cita todo el ecosistema de guardarrails ([arXiv:2404.00971](https://arxiv.org/abs/2404.00971)) encontró que un 15,1% de las alucinaciones en código son de tipo "API que no existe o nunca se importó" — y que **menos del 10% del código alucinado falla los tests**. Traducción: en un lenguaje de scripting, la mayoría del código alucinado atraviesa la suite de tests y solo explota en la ruta de ejecución que nadie probó. El compilador mata *clases* enteras de error en cada pasada, gratis; el runtime solo mata *instancias*, solo en caminos ejecutados. Para un agente que genera miles de líneas por sesión, esa diferencia de cobertura es enorme.

Además, la señal de error importa tanto como su existencia: un error de compilación es determinista, preciso y gratuito de interpretar para el modelo. Un stack trace de runtime solo informa de lo ejecutado. El bucle de corrección con compilador es más *informativo* por iteración.

## El agua fría empírica

Lo que nadie de los dos bandos quiere oír: los benchmarks recientes dicen que el lenguaje importa menos de lo que ambos creen.

- **MirrorCode** (2026), en tareas de horizonte largo: diferencias pequeñas de tasa de resolución entre Python, C, Rust, Go, OCaml y Ada; solo diferencias modestas de tokens entre runs exitosos. Los modelos actuales **transfieren su capacidad entre lenguajes** mejor de lo que la gente asume. ([resumen del debate](http://hndebrief.com/2026-08-11/whats-the-best-programming-language-for-coding-agents))
- El benchmark de danluu.com (agentes implementando un decodificador Zstandard y un conversor tipo Pandoc en varios lenguajes) fue recibido con escepticismo sano: la comunidad recondujo la conversación hacia tooling, verificación y convenciones, y dejó un consejo que sobrevive al ruido: optimiza tu repo y tus flujos alrededor de los bucles de verificación; trata la eficiencia de tokens por lenguaje como una señal débil.
- La observación que considero la más importante: **la arquitectura importa más que la pureza del lenguaje**. Un sistema componentizado con contratos nítidos hace que un lenguaje "débil" sea más fácil para un agente que un monolito en un lenguaje estricto.

## Mi ledger: tres factores, tres ganadores

Para evaluar un lenguaje como superficie de trabajo agéntica:

    1. Coste de un round-trip de corrección       → scripting (Python/JS) y Go
    2. Clases de error eliminadas por verificación → Rust (y en menor grado TS/Go)
    3. Carga de revisión humana y uniformidad      → Go, TypeScript tipado

La posición "scripting siempre" optimiza el factor 1 y olvida el 2. La posición "Rust es mejor, así de simple" optimiza el 2 y olvida el 1. Ambas están incompletas por la misma razón: el eje es verificación, no el modelo de ejecución.

Y el campeón del argumento de latencia no fue Python: fue Go, un lenguaje compilado. Go es la prueba de que la dicotomía estaba partida por la mitad desde dentro: tipos estáticos + compilación sub-segundo.

## La convergencia que disuelve el debate

La industria está resolviendo la pregunta por hibridación, no por victoria:

- **TypeScript** es hoy el lenguaje agéntico dominante en UI y es exactamente esa hibridación: runtime de scripting + compilador instantáneo (tsc). El caso Xcode Previews ya pasó en la web hace años: el hot reload (Vite/HMR) lleva la verificación al interior del bucle.
- **Python** respondió igual: pyright/mypy + ruff funcionan como "un compilador invocable en milisegundos" sobre un runtime de scripting. Los repos agénticos serios ejecutan el type-check como paso del bucle, igual que ejecutan tests.
- **Armin Ronacher** ([Fast and Hard Code](https://lucumr.pocoo.org/2026/8/22/fast-hard-code/), agosto 2026) — normalmente del bando scripting — predice lo contrario de mi sospecha inicial: *más* software en lenguajes estáticos compilados, porque los agentes absorben la curva de aprendizaje de Rust/Zig que antes frenaba su adopción. Cloudflare construye servicios en Zig puro, Vercel publicó un agente de código en Zig; casi todo, según él, asistido por LLMs.
- Dos anécdotas en direcciones opuestas, ambas reales: OpenAI migró su Codex CLI de Rust a Python en 2026 (de 648K LOC a 41K, una reducción de 15,9×; [arXiv:2604.11518](https://arxiv.org/abs/2604.11518)) — el impuesto de build y el ownership de Rust mataban la iteración. Y en sentido contrario, portar scikit-learn a Rust con agentes resultó viable con modelos actuales (Max Woolf, febrero 2026).

En una frase: **el mercado está convirtiendo cada lenguaje popular en "scripting con verificación instantánea"**. La dicotomía se disuelve en otra pregunta: ¿qué tan barato y qué tan completo es verificar?

## Veredicto práctico

- **Python**: el más versátil para agentes (ecosistema ML, bucle instantáneo). Condición: pyright/ruff/tests como parte del bucle, no como ocurrencia posterior.
- **TypeScript**: el estándar de facto para UI agéntica — el caso de las previews resuelto desde hace años.
- **Go**: el mejor compromiso backend — bucle de autocorrección más barato del mundo compilado, revisión humana uniforme.
- **Rust**: cuando clases enteras de bug deben morir en compile time (concurrencia, seguridad). Los agentes lo manejan bien ahora; el bucle a escala sigue siendo lento.
- **Evitar para trabajo agéntico-first**: builds pesados de JVM/Maven y monorepos C++ gigantes — el impuesto de planta es real.

Y la regla de fondo, la que creo que sobrevivirá a este debate: **elige por coste de verificación × cobertura de clases de error × calidad de la señal de error — no por el modelo de ejecución**. Con modelos que transfieren capacidad entre lenguajes, lo que marca la diferencia no es el lenguaje sino la infraestructura de verificación que pongas alrededor: tipos invocables, tests, linters — el equivalente exacto de Xcode Previews, pero para la lógica.
