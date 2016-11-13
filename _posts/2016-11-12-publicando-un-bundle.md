---
layout: post
title: Publicando un bundle!
published: true
---

Recientemente me he encontrado con que debia crear un bundle especifico en mi trabajo, el problema readico en que el bundle debia ser lo suficientemente generico como para ser capaz de adaptarse a cualquiera de los diferentes proyectos en los que trabaja la compañia. Fue entonces cuando decidi crear mi primer bundle distribuible.

Si en algun momento han tenido la oportunidad de registrar un bunble en [packagist](https://packagist.org/) sabran que la primera vez puede no ser sencillo entender la metodologia que se usa para registrar los bundles o cualquier proyecto.

Es por eso que decido explicar como publicar un proyecto distribuible. Es logico que nuestro proyecto tiene que estar almacenado en internet la comunidad de programadores utliza por defecto [GitHub](https://github.com/), de cierta forma es como el facebook para programadores. Sin embargo tambien puedes hostear tu bundle en cualquier plataforma de gestion de versiones online.

### Buenas Practicas

En la [documentacion oficial de symfony](https://symfony.com/doc/current/bundles/best_practices.html) nos muestra unas recomendaciones para tener en cuenta al momento de crear bundles que pretendemos distribuir, a continuacion mostrare un poco de lo que nos habla symfony que debemos tener en cuenta para estos casos, sin embargo, puedes sentirte libre en ir directamente a la documentacion y ampliar un poco la informacion que aqui explico.

- **Nombre:** para el nombre del bundle no es ningun secreto que debemos terminarlo con el sufijo 'Bundle', ademas de eso debe ser camelCase, alfanumerico y en caso de contener espacios, estos deben ser reemplazados por guiones bajos "_".

- **README.md:** este archivo es obligatorio, y puede contener ya sea la informacion de la instalacion o simplemente un preambulo de que es el bundle.

- **LICENSE:** pese a que la documentacion de symfony obliga a utilizarlo, me he encontrado con proyectos que no tienen este archivo. No obstante yo recomiendo usarlo si planeas convertirte en un "entrenador pokemon", es decir si planeas crear una imagen publica y que diferentes personas utilizen tu bundle, ya que este archivo estamos diciendo que pueden y que no pueden hacer otras personas con el. La licencia mas comun que podemos encontrar es "MIT" que basicamente dice que hemos hecho un proyecto pero que dejamos que otras personas hagan cualquier cosa con el. Podemos encontrar mas informacion acerca de los tipos de licencias en [choosealicense.com](http://choosealicense.com/).

- **Resources/docs/index.rst:** este archivo debe estar en esta ruta y basicamente lo que tiene es la documentacion oficial del bundle, del mimsmo modo cualquier archivo de documentacion deberia ir en este directorio.

### Composer.json

Posiblemente este archivo ha sido el que mas trabajo me ha dado en comprender, y luego de muchos intentos, encontre lo que para mi fue la manera mas sencilla de crear este archivo. Lo primero que debemos hacer es abrir una consola y dentro navegamos hasta nuestro proyecto para poder correr el comando composer init.

```SHEL

cd path/to/your/bundle
composer init

```