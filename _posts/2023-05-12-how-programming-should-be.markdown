---
layout:     post
title:      "Cómo me gustaría que fuera la programación"
subtitle:   "DSL descriptivos"
date:       2023-05-08 06:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:     es
lang-ref:   how-programming-should-be
---

Llevo muchos años programando, demasiados, más de los que mis compañeros de trabajo han vivido en total. Cualquiera podría concluir que estoy en la cumbre, después de todo lo que he pasado. Pero no me siento así.

En realidad, la experiencia como programador no se mide en años, sino en problemas resueltos. Y en ese sentido, no he resuelto tantos problemas como me gustaría. He resuelto muchos problemas, y no me refiero a problemas de programación, sino a problemas de negocio, de la vida real, lo que realmente importa.

Pero en lo que se refiere a mi forma de programar, me siento como un principiante, incapaz de hacer todo lo bien que me gustaría. Y no me refiero a que no sepa programar siguiendo los estándares de Clean Code, arquitecturas, XP, y demás. Me refiero a la estructura base del código.

El código ejecutable, por regla general, tiene una estructura secuencial: primero hago una cosa, luego compruebo otra, llamo aquí, llamo allá y devuelvo un dato. Y así, una y otra vez. Y no me gusta.

## ¿Cómo me gustaría programar?

Lo que me gustaría es un tipo de programación descriptiva. Es decir, que el código describa lo que quiero hacer y no cómo hacerlo, y que el lenguaje de programación me permita hacerlo. Por poner un ejemplo, tenemos a los modelos de ActiveRecord de cualquier proyecto **Ruby on Rails**:

```ruby
class User < ActiveRecord::Base
  belongs_to :merchant
  acts_as_token_authenticatable
  before_save :skip_confirmation_for_users
  after_create :send_instruction_email
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable, :lockable, :omniauthable, omniauth_providers: [:google_oauth2]
  validates_acceptance_of :tos_agreement, allow_nil: false, on: :create

  USER_ROLES = [:root, :admin, :user ]
  enum role: USER_ROLES
```

Es un estilo de programación donde describes lo que quieres; no describes la secuencia para resolver el problema, solo cuál es el problema. Imagínate programar diciéndole al compilador: "Quiero un login de usuarios pidiendo correo y password, pregunta al usuario si quiere mantener la sesión abierta por un mes, guarda el último correo siempre y preséntalo al usuario cada vez que acceda al login, pregunta al usuario si quiere guardar de manera segura el password. Autentica con Google, si no funciona presenta un mensaje de error; si funciona, invoca la funcionalidad X003".

Esto es lo que quiero. No es exactamente lo que propone Rails, pero es lo que más se le parece.

Lo curioso de esto es que quizás estemos llegando a este paradigma gracias a las actuales herramientas de generación de código por inteligencia artificial.

## DSL

Otra opción es la creación de **DSL** _(Domain Specific Languages)_ o lenguajes específico de dominio. Estos proyectos pretenden simplificar el uso de ciertas utilidades o funcionalidades que ofrecen los lenguajes de programación o plataformas de desarrollo.

Un ejemplo de esto sería el fichero de configuración del proyecto Fastlane. Para quien no lo conozca, Fastlane es un gestor de publicaciones, test y construcción de proyectos móviles. Ejecutas `$ fastlane ios release` y te sube versión al App Store Connect.

_(Aunque no lo parezca, esto es lenguaje Ruby)_

```ruby
default_platform(:ios)

platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
    build_app(scheme: "ListaCompra2")
    upload_to_app_store
  end
  
  desc "Launch the project tests"
  lane :tests do
    run_tests(scheme: "ListaCompra2")
  end
  
  desc "Increment build number in current dir"
  lane :increment do
    increment_build_number(xcodeproj: "ListaCompra2.xcodeproj")
  end
end
```

El problema de estos **DSL** es que suelen estar siempre limitados por su propio alcance, ofreciendo pocas capacidades de extensión y nunca ofrecen el total de la funcionalidad del entorno que pretende simplicar. Y peor aún es la cantidad de tiempo necesaria para mantener estos **DSL**, ya que suelen ser muy sensibles a los cambios de versión de las herramientas que los utilizan.

