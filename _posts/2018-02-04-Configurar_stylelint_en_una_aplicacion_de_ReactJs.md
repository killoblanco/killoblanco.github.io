---
layout: post
title: Cómo configurar Stylelint en una aplicación de ReactJS
published: true
---

Hola, en este post estaremos hablando de como hacer las configuraciones basicas `Stylelint` dentro de una aplicacion de ReactJS para que en caso de tener conflictos dentro de nuestras hojas de estilos la aplicación sea capaz de mostrarnos donde y cual es el conflicto.

Lo primero que debemos hacer es crear una aplicación de ReactJS en caso de no tener alguna y luego la vamos a ejectar para poder asi empezar con las configuraciones necesarias en nuestro proyecto.

```bash
# Creamos la aplicacion de react
~ $ npx create-react-app <app_name>
# Una vez dentro del directorio que nos ha generado vamos a
# ejectar la aplicación, cuando prengunte si realmente
# queremos ejectar la aplicación le daremos 'y'
<app_name> $ yarn eject
```

Muy bien, ahora tenemos una aplicación de ReactJS creada y ejectada procederemos a instalar las dependencias necesarias para que nuestro observador de estilos trabaje adecuadamente.

```bash
# Instalamos stylelint junto con el plugin para webpack
<app_name> $ yarn add --dev stylelint stylelint-webpack-plugin
```
Ahora que tenemos `Stylelint` instalado podemos hacer la configuración basica. Dentro de nuestra estructura de directorios tendremos una carpeta que se llama `config/` la cual dentro contriene nuestro archivo de configuración de webpack.

Con nuestro editor de codigo modificamos el archivo `webpack.config.dev.js` para que se vea como el siguiente:

```js
'use strict';

...
const StyleLintPlugin = require('stylelint-webpack-plugin');

...
module.exports = {
  ...
  plugins: [
    ...
    // Style Linter Plugin
    new StyleLintPlugin({
        configFile: '.stylelintrc',
        context: 'src',
        files: '**/*.css',
        failOnError: false, 
        quiet: false,
    }),
  ],
  ...
}
```

Para saber mas hacerca de las opciones que recive `StyleLintPlugin` puedere revisar su [documentación oficial](https://github.com/JaKXz/stylelint-webpack-plugin#options).

Con estos pasos hechos hasta aqui solo hace falta definir nuestras reglas para nuestro linter, en este caso solo extenderemos las reglas del la configuración estandar para compartir estilos. Así que, instalemos `stylelint-config-standard` y agreguemos lo a nuestro proyecto.

```bash
<app_name> $ yarn add --dev stylelint-config-standard
```

Por ultimo creamos un archivo con el nombre `.stylelintrc` en la raiz de nuestro proyecto y le agregamos las siguientes lineas.

```json
// .stylelintrc
{
  "extends": "stylelint-config-standard"
}
```

Puedes leer mas sobre las reglas de `Stylelint` desde su [web oficial](https://stylelint.io/user-guide/rules/), asi como puedes revisar la documentacion de la configuración estandar [aqui](https://github.com/stylelint/stylelint-config-standard).
