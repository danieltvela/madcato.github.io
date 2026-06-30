---
layout:     post
title:      "Principios SOLID"
subtitle:   "¿Qué son los principios solid?"
date:       2023-11-05 06:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:     es
lang-ref:   solid-principles
---

# SOLID

Los principios SOLID son comúnmente utilizados para orientar decisiones arquitectónicas en el desarrollo de software. Son cinco reglas diseñadas para fomentar un código orientado a objetos que sea limpio y extensible, asegurando comportamientos estables sin necesidad de modificar el código existente.

Los principios SOLID son los siguientes:

1.	**Single Responsibility**: Cada clase o función debe tener una única responsabilidad. Si una clase maneja múltiples tareas, es recomendable dividirla en clases más simples con responsabilidades únicas. Lo mismo aplica para las funciones: si una función tiene demasiados parámetros o realiza numerosas operaciones, debería dividirse en funciones más pequeñas y específicas.
Es crucial nombrar correctamente variables, clases y funciones para reflejar su responsabilidad exclusiva.
2.	**Open/Closed**: Las clases deben estar abiertas para extensión pero cerradas para modificación. En lugar de cambiar el comportamiento de una clase existente, es preferible crear una nueva que la extienda.
Modificar una clase a menudo contraviene el primer principio de SOLID. Diseñar clases que puedan ser extendidas sin ser modificadas es una forma de evitar este problema.
3.	**Liskov Substitution**: Las instancias de clases derivadas deben poder reemplazar a las de sus clases base. Esto significa que las especializaciones deben mantener los comportamientos de la clase base, ajustando los detalles para cumplir con el contrato de la clase base.
4.	**Interface Segregation**: Es preferible tener múltiples interfaces pequeñas en lugar de una grande. Este principio es similar al de Single Responsibility pero aplicado a las interfaces.
5.	**Dependency Inversion**: Se debe depender de abstracciones, no de implementaciones concretas. Esto significa que nuestro código debería interactuar con interfaces, no con implementaciones específicas, a menudo logrado a través de la inyección de dependencias.

## Conclusiones

Aunque estos principios se asocian con la programación orientada a objetos, no son exclusivos de ella. Ayudan a mejorar la cohesión del código y a evitar dependencias excesivas entre clases, permitiendo que cada parte del código conozca solo lo que necesita para funcionar correctamente.

