<script setup lang="ts">
import Top from './components/Top.vue'
</script>

<template>
    <div class="container">
        <header>
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

@media (min-width: 1024px) {
    header {
        display: flex;
        place-items: center;
        padding-right: calc(var(--section-gap) / 2);
    }

    .logo {
        margin: 0 2rem 0 0;
    }
}

/* Container style */
.container {
    @apply mx-auto p-6 max-w-full text-center bg-green-100;
    /* 背景色をVue.jsの緑色に設定 */
}
</style>
