# Añadiendo estilos con Next,js y CSS

En este ejemplo, vamos a cubrir los aspectos básicos de la configuración de Next.js para incorporar estilos externamente en ficheros CSS

# Paso para incluir archivos CSS

- Tomaremos el ejemplo 06 como punto de partida.

- Como de costumbre, el primer paso sería instalar las dependencias especificadas en el archivo package.json.

```bash
npm install
```

- Para poder utilizar archivos CSS en nuestra aplicación, nos hace falta añadir una nueva dependencia al proyecto. Concretamente, se trata del plugin ```next-css```

```bash
npm install @zeit/next-css --save
```

- Este plugin nos facilita una función ```withCSS``` que podemos invocar desde nuestro archivo de configuración ```next.config.js``` para añadir la funcionalidad necesaria para gestionar archivos CSS en nuestro empaquetado. Para ello, es necesario modificar nuestro fichero ```next.config.js``` tal cual sigue:

_./next.config.js_

```diff
const withTypescript = require('@zeit/next-typescript');
+ const withCSS = require('@zeit/next-css');

- module.exports = withTypescript();
+ module.exports = withTypescript(withCSS());

```

- Con esto ya podemos procesar archivos CSS. El siguiente paso sería crear unos estilos en un fichero externo para probarlo. Creamos pues un fichero ```global-styles.css```:

_./styles/global-styles.css_

```
.blue-box {
  border: 3px solid blue;
}
```

- Y ahora hacemos uso de esta nueva clase en nuestra lista de usuarios. Por ejemplo, podemos aplicar la nueva regla de estilos a la columna 'Avatar' de nuestro componente cabecera.

_./components/users/header.tsx_

```diff
+ import '../../styles/global-styles.css';

export const Header = () => (
  <tr>
-   <th>Avatar</th>
+   <th className="blue-box">Avatar</th>
    <th>Id</th>
    <th>Name</th>
  </tr>
)

```

- Si fueramos a lanzar nuestra aplicación ahora, no veríamos, sin embargo, ningún cambio. Esto se debe a que todavía no le hemos dicho a Next.js como deberíamos enlazar este nuevo recurso con nuestra aplicación web. Por defecto, ```next-css``` compila nuestros archivos CSS en ```.next/static/style.css```, y estos contenidos se sirven desde la siguiente url

```
/_next/static/style.css
```

- Por tanto, necesitamos indicarle a nuestra aplicación que use los estilos desde ese punto de entrada. Para conseguirlo, debemos indicarle a Next.js que utilice un documento personalizado como base, en el cual se indique mediante el uso de elementos ```link``` en la cabecera la fuente de entrada para nuestros estilos CSS. Podemos conseguir esto mediante la creación de un objeto ```_document.js``` (¡ojo, que debe llevar un guión bajo en el nombre!) dentro de nuestra carpeta ```pages```. Dicho archivo contendría el siguiente código:

_./pages/\_document.js_

```javascript
import Document, { Head, Main, NextScript } from 'next/document';

export default class MyDocument extends Document {
  render() {
    return (
      <html>
        <Head>
          <link rel="stylesheet" href="/_next/static/css/styles.chunk.css" />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </html>
    );
  }
}

```

- Ahora sí que podemos lanzar nuestra app con ```npm run dev```, y comprobaremos que, efectivamente, la parte de la cabecera correspondiente a 'Avatar' queda encuadrada dentro de un rectángulo azúl.

# Pasos para añadir CSS utilizando el modelo de CSS-Modules

- Si queremos utilizar módulos CSS en nuestra aplicación, basta con habilitar el correspondiente flag en la configuración base de nuestro archivo ```next.config.js```.

_./next.config.js_

```diff
const withTypescript = require('@zeit/next-typescript');
const withCSS = require('@zeit/next-css');

- module.exports = withTypescript(withCSS());
+ module.exports = withTypescript(
+   withCSS({
+     cssModules: true,
+     cssLoaderOptions: {
+       camelCase: true,
+     },
+   })
+ );

```

- Tras hacer esto, podemos crear ahora un archivo ```header.css``` dentro de la carpeta del componente que renderiza la lista de usuarios, y anexarle a este archivo algunas reglas de estilo. Por ejemplo:

_./components/users/header.css_

```css
.purple-box {
  border: 2px dotted purple;
}
```

- Ahora podemos, por ejemplo, modificar nuestro componente cabecera para aplicar el nuevo estilo al elemento 'Id'. Resaltar que al haber utilizado kebab-case, necesitamos utilizar una propiedad calculada para acceder a la clase CSS en concreto.

_./components/users/header.tsx_

```diff
import '../../styles/global-styles.css';
+ const styles = require('./header.css');

export const Header = () => (
  <tr>
    <th className="blue-box">Avatar</th>
-   <th>Id</th>
+   <th className={styles.purpleBox}>Id</th>
    <th>Name</th>
  </tr>
);

```

- Si lanzamos nuestra aplicación, veremos que en efecto el elemento Id queda envuelto en un cuadro púrpura punteado. Además, el rectángulo azúl del elemento Avatar ha desaparecido. Esto es así debido a que ahora todos los CSS se compilan usando el patrón de módulo CSS. Por tanto, no podemos utilizar clases sólo con el nombre literal de la regla CSS en cuestión. De hecho, podemos comprobar que las dos clases creadas siguen estando en nuestra aplicación compilada. Basta con ir a la consola del navegador y mirar las fuentes para el archivo _next/static/style.css, que debe contener tanto la regla para el rectángulo azúl como el púrpura.

- Vamos a refactorizar un poco el código para dejar todos los estilos acordes al patrón de Módulos CSS. Borramos la carpeta ```styles``` anteriormente creada y en su lugar añadimos la regla del cuadro azúl a nuestro fichero ```header.css```. Además, le quitamos el guión de enmedio para no tener que utilizar una propiedad computada para referirlo en el componente.

_./styles/global-styles.css_

```diff
- .blue-box {
-   border: 3px solid blue;
- }

```

_./components/users/header.css_

```diff
.purple-box {
  border: 2px dotted purple;
}

+.blue-box {
+  border: 3px solid blue;
+}
```

- Por último, refactorizamos el componente cabecera de forma acorde.

_./components/users/header.tsx_

```diff
- import '../../styles/global-styles.css';
const styles = require('./header.css');

export const Header = () => (
  <tr>
-   <th className="blue-box">Avatar</th>
+   <th className={styles.blueBox}>Avatar</th>
    <th className={styles.purpleBox}>Id</th>
    <th>Name</th>
  </tr>
);

```

- Ahora deberíamos de ser capaces de ver los dos estilos aplicados al unísono.

- Para finalizar, la función ```withCSS``` básicamente nos ofrece azucarillo sintáctico en forma de composición funcional para simplificar la configuración de webpack que subyace. De hecho, internamente utiliza un ```css-loader``` para gestionar los ficheros CSS. Podemos indicar opciones de configuración propias de dicho módulo (ver [webpack's css loader options](https://github.com/webpack-contrib/css-loader#options) para más detalles), como, por ejemplo, añadir ids de ámbito local a nuestras clases CSS. Si modificamos el fichero ```next.config.js``` conforme a lo siguiente:

_./next.config.js_

```diff
const withTypescript = require('@zeit/next-typescript');
const withCSS = require('@zeit/next-css');

module.exports = withTypescript(
  withCSS({
    cssModules: true,
    cssLoaderOptions: {
      camelCase: true,
+     importLoaders: 1,
+     localIdentName: '[local]___[hash:base64:5]',
    },
  })
);

```

- Y comprobamos ahora las clases que se muestran en la consola del navegador para la fuente  ```_next/static/styles.css```, podemos corroborar que en lugar de la cadena hash ostentosamente más larga que había antes, ahora las clases tienen nombres que se ajustan al patrón indicado en ```cssLoaderOptions.localIdentName```.

- Como estamos usando `css-modules`, no necesitamos la pagina `_document.js` para cargar los ficheros css. Asi que podemos eliminarla.

## Apendice

- Para dar finalmente por terminado el ejemplo, vamos a refactorizar nuestro código para aplicar un estilado más elegante a nuestra tabla de usuarios:

_./components/users/header.css_

```diff
- .purple-box {
-   border: 2px dotted purple;
- }

- .blue-box {
-   border: 3px solid blue;
- }

+ .header {
+   background-color: #ddefef;
+   border: solid 1px #ddeeee;
+   color: #336b6b;
+   padding: 10px;
+   text-align: left;
+   text-shadow: 1px 1px 1px #fff;
+ }

```

- Actualizamos `row` y `table`:

_./components/users/row.css_

```css
.row {
  border: solid 1px #DDEEEE;
  color: #333;
  padding: 10px;
  text-shadow: 1px 1px 1px #fff;
}

```

_./components/users/table.css_

```css
.table {
  border: solid 1px #ddeeee;
  border-collapse: collapse;
  border-spacing: 0;
  font: normal 13px Arial, sans-serif;
}

```

- Y por último, modificamos los componentes para importar las nuevas reglas de estilo:

_./components/users/header.tsx_

```diff
const styles = require('./header.css');

export const Header = () => (
- <tr>
+ <tr className={header}>
-   <th className={styles.blueBox}>Avatar</th>
+   <th>Avatar</th>
-   <th className={styles.purpleBox}>Id</th>
+   <th>Id</th>
    <th>Name</th>
  </tr>
);

```

_./components/users/row.tsx_

```diff
import * as Next from 'next';
import Link from 'next/link';
import { User } from '../../model/user';
+ const styles = require('./row.css');
...

export const Row: Next.NextStatelessComponent<Props> = (props) => (
- <tr>
+ <tr className={styles.row}>
    <td>
...

```

_./components/users/table.tsx_

```diff
import * as Next from 'next';
import { User } from '../../model/user';
import { Header } from './header';
import { Row } from './row';
+ const styles = require('./table.css');

...

export const Table: Next.NextStatelessComponent<Props> = (props) => (
- <table>
+ <table className={styles.table}>
...

```
