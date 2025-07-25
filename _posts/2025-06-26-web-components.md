---
title: Introducción a la creación de componentes con lit + ts
tags:
    - configuraciones
    - desarrollo
---

# Configuraciones previas
Sobre todo, tener instalado `nodejs`. Instalar el paquete que nos provea del comando `tsc` para compilar el TypeScript. 

En este ejemplo uso `yarn`, pero debería funcionar con los comandos equivalentes en `npm` según tengo entendido

# Creación del proyecto
Creamos un proyecto de nombre `proyecto`. El template a usar es `lit-ts`. (Pueden verse los demás templates disponibles usando `--help`)

```bash
yarn create vite proyecto --template lit-ts 
cd proyecto
```

# Configuración de tailwindcss
La configuración de tailwindcss ha cambiado mucho en la versión 4, en el caso de la integración con Vite, esta parece haber sido simplificada:

```bash
yarn add -D @tailwindcss/vite
```

Necesitamos configurar `vite.config.ts` de la siguiente forma:
```ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    tailwindcss(), // Plugin oficial de Tailwind CSS 4
  ],
  build: {
    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          if (assetInfo.name?.endsWith('.css')) {
            return 'index.css'
          }
          return assetInfo.name || 'asset'
        }
      }
    }
  }
})
```

Cambiamos el contenido del archivo `src/index.css` para poder crear estilos personalizados de forma global. Por ahora, basta con agregar el [preflight](https://tailwindcss.com/docs/preflight) de tailwindcss, que resetea la mayoría de los estilos propios de cada navegador, por lo cual es una buen punto de inicio.

```css
@import "tailwindcss";
```

Para efectos de prueba, usaremos el componente que se hizo por defecto, `my-element`, que se define en `src/my-element.ts`. 
Basta con cambiar el contenido actual para que queda de la siguiente forma:

```ts
import { LitElement, html, css, unsafeCSS } from 'lit'
import { customElement } from 'lit/decorators.js'
// Importar CSS como texto para inyectarlo en Shadow DOM
import tailwindStyles from './index.css?inline'

@customElement('my-element')
export class MyElement extends LitElement {
  // Combinar Tailwind con estilos específicos del componente
  static styles = [
    // Inyectar Tailwind en el Shadow DOM
    unsafeCSS(tailwindStyles),
    // Estilos específicos del componente (scoped)
    css`
      :host {
        display: block;
      }
    `
  ]

  render() {
    return html`
      <div class="p-4 bg-blue-500 text-white rounded-lg shadow-md max-w-sm">
        <h1 class="text-2xl font-bold mb-4 custom-animation">Primer intento</h1>
        <p class="mt-2 text-sm opacity-75">
          CSS scoped + Tailwind global funcionando
        </p>
      </div>
    `
  }
}
```
Ejecutamos desde consola

```bash
yarn dev
El resultado es el siguiente:
```

![Pantalla inicial de mitmproxy]({{ site.url }}{{ site.baseurl }}/assets/images/Captura de pantalla_2025-07-25_11-05-59.png)

El reset que usamos le quito todo el estilo por defecto en el navegador, por eso no hay margenes alrededor de nuestro elemento, que por otra parte, esta muy bonito.

Como un primer acercamiento creo que esta bien. Incluso podría servir para crear elementos sin considerar estilos alrededor
