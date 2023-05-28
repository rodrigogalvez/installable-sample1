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