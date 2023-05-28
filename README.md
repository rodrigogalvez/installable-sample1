# Aplicación instalable

## Propuesta

Hacer una App instalable utilizando Vite+VueJS+TypeScript y otras herramientas. El objetivo es crear un material de consulta para trabajar en la instalación y actualización de la App como PWA o SPA o como aplicación de escritorio.

Crearemos una App ridículamente simple para no distraernos con la lógica y concentrarnos en el proceso de configuración e instalación. La App solo dirá *Hola Mundo Versión 1.0* y la usaremos en el navegador de escritorio o el teléfono. Una vez instalada como aplicación, cambiaremos algo (por ejemplo que diga *Hola Mundo Version 2.0*) para ver si se detecta el cambio automáticamente.

Este documento se debe leer secuencialmente. Explora los problemas y actividades que realizar en cada momento. Por ejemplo, el primer problema es que Vite no reconoce el alias "@" para referirse a la carpeta *src*.

## Paso 1: Inicialización - crear el proyecto

Creada con Vite y TypeScrpt.

```shell
npm create vite
```

Hacer lo siguiente:

* Nombrar el proyecto (en este caso, *installable-sample1*)
* Seleccionar **Vue**
* Seleccionar **TypeScript**

Instalar dependencias usando y entrar a VS Code:

```shell
cd installable-sample1
npm install
code .
```

## Paso 2: Problema - Vite no reconoce el alias "@"

**Problema**: Vite no reconoce el alias "@". VS Code lo presenta como error y el browser en línea presenta errores. El alias "@" permite referirse a la carpeta *src* del proyecto. Esto no funciona en Vite en este momento. Por ejemplo, en este código fuente siguiente que se genera automáticamente al crear el proyecto, se cambió el punto por una arroba. VS Code lo presentará como un error y Vite entrega un error de compilación:

```TypeScript
<script setup lang="ts">
import HelloWorld from '@/components/HelloWorld.vue'
</script>
```
**Solución**: [(adaptado desde aquí y otras fuentes que no anoté)](https://dev.to/tikashi/Vite-import-path-alias-only-setting-tsconfigjson-2fjh) Para corregir esto, instalar *Vite-ts-config-paths*:

```shell
npm install --save Vite-tsconfig-paths
```

Corregir el archivo Vite.config.ts agregando las línea que falten:

```TypeScript
import { defineConfig } from 'Vite'
import vue from '@Vitejs/plugin-vue'
import path from 'path'
import tsconfigPaths from 'Vite-tsconfig-paths'


// https://Vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), tsconfigPaths()],
  resolve: {
    alias: [{ find: '@', replacement: path.resolve(__dirname, 'src') }],
  },
})
```

Ageregar en tsconfig.json las siguientes líneas dentro de *compilerOptions*:

```json
{
  "compilerOptions": {
    /*...*/
    "paths": {
      "@": [
        "./src"
      ],
      "@/*": [
        "./src/*"
      ],
      "$lib": [
        "./src/lib"
      ],
      "$lib/*": [
        "./src/lib/*"
      ]
    }
    /*...*/
  }
}
```

## Paso 3: instalar hoja de estilo

Lidiar con una hoja de estilo puede ser una gran inversión de tiempo. Para simplicidad, utilizaremos *PicoCSS*, que es una hoja de estilo muy simple. Más información [aquí](https://picocss.com/).

Para instalar:

```shell
npm install --save @picocss/pico
```

Para habilitar en el proyecto, modificaremos el archivo *main.ts*:

```TypeScript
import { createApp } from 'vue'
import './../node_modules/@picocss/pico/css/pico.min.css'
import App from './App.vue'

createApp(App).mount('#app')
```

## Paso 4: crear la funcionalidad principal

Recordemos que la App será ridículamente simple para evitar distraeronos con la lógica y concentrarnos en la configuración e instalación. Borraremos entonces el archivo HelloWorld.vue y crearemos uno nuevo llamado *Function1.vue* en la carpeta *components*.

Bueno, mentí un poco. Para hacerla un poco menos aburrida, utilizaremos un tag llamado *details* para mostrar el texto del saludo y preguntaremos a quién saludar con un campo de texto.

El archivo *Function1.vue* está a continuación:

```TypeScript
<template>
    <details open>
        <summary role="button" class="primary">Saludar</summary>
        <form v-on:submit.prevent>
            <label>
                Nombre
                <input type="text" v-model="username" />
            </label>
            <label>Saludo
                <p>¡Hola {{ username }}!</p>
                <small>Versión de la App: 1.0</small>
            </label>
        </form>
    </details>
</template>
    
<script lang="ts">

export default {
    data() {
        return {
            username: "mundo",
        };
    }
};
</script>
```

Luego reemplazamos el archivo *App.vue* por este para incorporar la funcionalidad:

```TypeScript
<script setup lang="ts">
import Function1 from '@/components/Function1.vue';
</script>

<template>
  <main class="container">
    <Function1 />
  </main>
</template>
```

## Paso 5: probar antes de continuar

Podemos probar la aplicación en el browser en línea y ver que no existan errores. Al hacer click sobre el botón *Saludar* aparecerá o se ocultará el formulario de la aplicación. El texto con el saludo aparece un poco más abajo. No es muy espectacular, pero servirá para probar.

Ahora viene lo interesante...

## Paso 6: Convertir este proyecto en una PWA

Agregaremos al proyecto el plug-in PWA de Vite. Si seguimos más o menos las instrucciones que están [en este tutorial](https://rubenr.dev/pwa-vite/), tendremos una App instalable y que además funciona sin conexión a Internet. Lo malo del citado tutorial, es que asumen que uno hará un único ejercicio. Pero en este documento explico como hacer los mismos pasos con el proyecto ya iniciado (por ejemplo, si ya hemos creado la App hace tiepmpo y ahora queremos convertirla en PWA en vez de partir desde cero).

Para el navegador considere la App como PWA, debe complir con cuatro requisitos:

* Debe tener un ícono (usaremos el archivo *vite.svg* que ya existe en el proyecto),
* Debe tener una ruta de inicio (dejaremos la raíz),
* Debe tener un archivo de manifiesto (en inglés se llama *manifest*) y
* Debe tener un Service Worker

Lo primero es instalar el plug-in PWA de Vite y una librería llamada Workbox. Esta última que nos ayudará con el manifiesto y lo generará automáticamente. Usaremos este comando:

```shell
npm install --save vite-plugin-pwa workbox-precaching
```

Modificaremos el archivo *vite.config.ts* otra vez para que se active el plug-in. Notar que esta modificación es más sencilla que la que aparece en el tutorial. Debemos agregar a la sección plugins el VitePWA con algunos parámetros (recuerda cambiar el nombre del proyecto al que estás trabajando).

```TypeScript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'
import tsconfigPaths from 'vite-tsconfig-paths'
import { VitePWA } from "vite-plugin-pwa";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), tsconfigPaths(),
    VitePWA({
      mode: "development",
      base: "/",
      srcDir: "src",
      filename: "sw.ts",
      includeAssets: ["/favicon.png"],
      strategies: "injectManifest",
      manifest: {
        name: "Installable-sample1",
        short_name: "Installable-s1",
        theme_color: "#ffffff",
        start_url: "/",
        display: "standalone",
        background_color: "#ffffff",
        icons: [
          {
            src: "vite.svg",
            sizes: "192x192",
            type: "image/png",
          },
        ],
      },
    })
  ],
  resolve: {
    alias: [{ find: '@', replacement: path.resolve(__dirname, 'src') }],
  },
})
```

Notar que el ícono está en la sección *manifest.icons*, dejamos el archivo vite.svg. Lo podemos cambiar, pero en otro momento. Además dejamos la ruta raíz en el parámetro *manifest.start_url*.

Los parámetros del plug-in VitePWA hacen referencia a un archivo llamado *sw.ts*. Lo crearemos con este contenido en la carpeta *src*:

```TypeScript
import { precacheAndRoute } from 'workbox-precaching'
declare let self: ServiceWorkerGlobalScope
self.addEventListener('message', (event) => {
  if (event.data && event.data.type === 'SKIP_WAITING') self.skipWaiting()
})
// self.__WB_MANIFEST is default injection point
precacheAndRoute(self.__WB_MANIFEST)
```

El editor VS Code nos mostrará muchos errores en pantalla. ¡No desesperes!. Lo corregiremos en un momento.

En el archivo *tsconfig.json* haremos las siguientes modificaciones:

* Agregaremos la liberería *WebWorker* a la sección *compilerOptions.lib*,
* Agregaremos una sección de tipos a *compilerOptions* y
* Agregaremos una sección de exclusines.

Eso se resume más o menos así:

```json
{
  "compilerOptions": {
    /*...*/
    "lib": [
      /*...*/
      "WebWorker"
    ],
    "types": [
      "node",
      "vite-plugin-pwa/client"
    ],
  },
  "exclude": [
    "dist",
    "node_modules",
    "test",
    "test.ts",
    "**/*.spec.ts",
    "**/*.worker.ts"
  ],
  /*...*/
}
```

Por último, debemos registrar la aplicación. Para eso crearemos un componente que podemos reutilizar en cualquier momento sin cambios (bueno, tal vez algunos en lo estético). Notar que el componente en el tutorial original solo actualiza la App si hay modificaciones. El componente propuesto aquí además ofrece la instalación de la App si es posible además de la actualización.

La instalación no puede ser automática. El método de instalación requiere que antes haya una interacción humana, es decir, alguien tiene que hacer un click en un botón. El proceso es más o menos como sigue:

* Escuchar el evento *beforeinstallprompt* y capturarlo,
* Disponibilizar un botón en la pantalla para que el usuario pueda presionarlo,
* Invocar el método *prompt* del evento *beforeinstallprompt* capturado, que preguntará efectivamente al usuario si desea instalar la aplicación.
* Invocar al método *userChoice*, que devuelve una promesa especial con una estructura que define y entrega elección del usuario.

El componente lo crearemos en la carpeta *components* y lo llamaremos *ReloadPWA.vue*. Este es el código fuente:

```TypeScript
<template>
    <dialog v-if="offlineReady || needRefresh || couldInstall" open>
        <article>
            <header>
                <h3>Importante</h3>
            </header>
            <p v-if="couldInstall">App lista para instalar</p>
            <p v-if="offlineReady">App lista para funcionar sin Internet</p>
            <p v-else>Nueva versión de la App disponible</p>
            <footer>
                <a href="#" role="button" v-if="couldInstall" @click.prevent="install">
                    Instalar
                </a>
                <a href="#" role="button" v-if="needRefresh" @click.prevent="updateServiceWorker()">
                    Actualizar App
                </a>
                <a href="#" role="button" @click.prevent="close" class="secondary">
                    Ok
                </a>
            </footer>
        </article>
    </dialog>
</template>
<script lang="ts">

import { ref } from "@vue/reactivity";
import { useRegisterSW } from "virtual:pwa-register/vue";

interface BeforeInstallPromptEvent extends Event {
    readonly platforms: Array<string>;
    readonly userChoice: Promise<{
        outcome: 'accepted' | 'dismissed',
        platform: string
    }>;
    prompt(): Promise<void>;
}

export default {
    setup() {
        const { offlineReady, needRefresh, updateServiceWorker } = useRegisterSW();
        const couldInstall = ref(false);
        const close = async () => {
            offlineReady.value = false;
            needRefresh.value = false;
            couldInstall.value = false;
        };
        let theEvent: BeforeInstallPromptEvent | undefined = undefined;

        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            theEvent = (e as BeforeInstallPromptEvent);
            couldInstall.value = true;
            console.log(e);
        });

        const install = async () => {
            theEvent?.prompt();
            theEvent?.userChoice
                .then(choice => {
                    if (choice.outcome === 'accepted') {
                        console.log('App instalada');
                        close();
                    } else {
                        console.log('No se instala la App');
                    }
                })
        }

        return { offlineReady, needRefresh, couldInstall, install, updateServiceWorker, close };
    },

}
</script>
```

Para terminar, agregamos este componente al archivo *App.vue*. El código fuente quedará así:

```TypeScript
<script setup lang="ts">
import Function1 from '@/components/Function1.vue';
import ReloadPWA from '@/components/ReloadPWA.vue';
</script>

<template>
  <main class="container">
    <ReloadPWA />
    <Function1 />
  </main>
</template>
```

Si quieres saber más sobre el plug-in VitePWA, [mira este enlace](https://vite-pwa-org.netlify.app/frameworks/vue.html). Si quieres saber más sobre el evento *beforeinstallprompt*, [mira este enlace](https://stackoverflow.com/questions/58729197/the-prompt-method-must-be-called-with-a-user-gesture-error-in-angular-pwa).

## Paso 7: ¡Probar!

Si has mantenido abierto el navegador de Vite, notarás que nada ha pasado. Esto es porque, de acuerdo a como está escrito el archivo *sw.ts*, no permite la instlacación si estamos este navegador.

Para probar, debemos generar la versión final del proyecto y luego pervisualizar. Antes de comenzar, agreguemos recomiendo que agregues al archivo *.gitignore* la carpeta *public*.

Generaremos la versión final del proyecto utilizando este comando:

```shell
npm run build
```

Si no hay errores, podemos probar utilizando este comando:

```shell
npm run preview
```

Si todo sale bien, deberías ver la aplicación funcionando. Si ya habías ejecutado la aplicación o la volviste generar, una ventana avisará que hay una nueva versión disponible. También ofrecerá instalar la aplicación.

**Nota importante**: cuando ejecutas la previsualización, y ya has construido una App, no ofrecerá instalarla ya que la ruta que quedó registrada es la ruta raíz del computador local, por ejemplo algo así:

```html
http://localhost:4173/
```

El navegador asume que es la misma aplicación. Por lo tanto debes desinstalarla para probar la nueva. Otra alternativa es modificar el archivo *hosts* para definir una ruta local diferente y probar varias App de manera independiente.
