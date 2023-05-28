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
    