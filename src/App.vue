<script setup lang="ts">
import Top from './components/Top.vue'
</script>

<template>
    <div class="container">
        <header class="flex justify-center">
            <img alt="logo" class="logo" src="./assets/keiba_jockey4_blue.png" width="125" />
        </header>
        <main>
            <Top />
        </main>
    </div>
</template>

<style scoped>
header {
    line-height: 1.5;
}

.logo {
    display: block;
    margin: 0 auto 2rem;
}

/* Container style */
.container {
    @apply p-6 text-center mx-auto bg-green-100;
    /* 背景色をVue.jsの緑色に設定 */
}
</style>
