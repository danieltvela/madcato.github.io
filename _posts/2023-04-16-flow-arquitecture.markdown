---
layout:     post
title:      "Arquitecturas de flujo unidireccional para iOS"
subtitle:   "Qué son las arquitecturas de flujo y ejemplos."
date:       2023-04-21 06:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:     es
lang-ref:   flow-arquitecture
---

## ¿Qué son las arquitecturas de flujo unidireccionaal?
¡Las arquitecturas de flujo son una forma interesante de pensar en el diseño de software, y han ganado popularidad en los últimos años debido a su capacidad para simplificar el proceso de desarrollo y mantener aplicaciones escalables. En este post, exploraremos algunos ejemplos de arquitecturas de flujo, sus ventajas y cómo se aplican en el mundo real.

Es importante entender la idea detrás de las arquitecturas de flujo. En lugar de pensar en el software como una colección de componentes, **las arquitecturas de flujo lo consideran como un flujo continuo de datos**. Esto significa que el software se organiza en torno a los flujos de datos y cómo se mueven a través de la aplicación.

El flujo de datos puede ser controlado por un "controlador" que asegura que los datos lleguen a donde deben ir. En lugar de tener múltiples componentes que se comunican entre sí, todas las partes de la aplicación se conectan al controlador, lo que simplifica el diseño.

Uno de los ejemplos más populares de una arquitectura de flujo es la arquitectura por eventos. Esta arquitectura es muy popular en la programación de sistemas y en el desarrollo de aplicaciones móviles. La idea detrás de esta arquitectura es simple: **se utilizan eventos para comunicar información entre diferentes partes de la aplicación**.

En este modelo, la aplicación se divide en pequeñas partes llamadas "eventos". Estos eventos son enviados a través de un bus de eventos central que los distribuye a los diferentes componentes de la aplicación. Los componentes pueden ser servicios, bases de datos, servicios de mensajería, **y las propias vistas que inician los eventos; realizando un trayecto completo en una única dirección** . Cada componente recibe los eventos que le interesan y los procesa de acuerdo a sus necesidades.

Esta animación del proyecto Suas muestra cómo funciona la arquitectura de flujo de datos unidireccional.

![Arquitectura de flujo de datos unidireccional](https://camo.githubusercontent.com/20f9063b591f63cf779535868fdd7ca01a09dc2ac38c43d369cfc51dd5125364/687474703a2f2f692e696d6775722e636f6d2f453743783274662e676966)

## Ejemplos de arquitecturas de flujo aplicadas a la programación móvil
Estos son solo unas pocas librerías que utilizan arquitectura de flujo.

- [Suas](https://github.com/danieltvela/Suas-iOS) es una implementación de la arquitectura Fluxflujo de datos unidireccional para iOS. Tiene una buena animación explicativo de que son este tipo de arquitecturas. Este proyecto parece descontinuado, en origen pertenecía a ZenDesk. Es una pena porque disponía de un sistema de logging excelente.

- [The Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture). Esta librería permite construir aplicaciones de manera consistente y entendible, mediante composición, testeos y facilidad de uso. Puede ser usada con SwiftUI o UIKit y en cualquier plataforma que soporte Swift.

- [Reswift: Unidirectional Data Flow in Swift - Inspired by Redux](https://github.com/ReSwift/ReSwift). ReSwift es como una implementación de Redux de la arquitectura de flujo de datos uniireccional para Swift.

- [Awesome iOS architecture](https://github.com/onmyway133/awesome-ios-architecture#swiftui). Este es un excelente recurso que contiene un listado con muchas otras librerías que utilizan arquitecturas para iOS, así cómo artículos sobre arquitectura.