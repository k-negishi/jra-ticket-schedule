<script setup lang="ts">
import Top from './components/Top.vue'
</script>

<template>
    <div class="app">
        <header class="bg-gray-400 flex p-4">
            <div class="logo-container">
                <img alt="logo" class="logo" src="./assets/keiba.png" />
            </div>
            <div class="text-white ml-4">
                <p class="text-lg font-semibold">JRA指定席</p>
                <p class="text-lg font-semibold">ネット予約スケジュール</p>
            </div>
        </header>
        <div class="container">
            <main>
                <Top />
            </main>
        </div>
    </div>
</template>

<style scoped>
/* App style */
.app {
    margin: 0 auto; /* 中央に配置するためのマージン設定 */
    width: 100%; /* 幅100%を設定 */
}

/* ヘッダーのスタイル */
header {
    background-color: rgba(169, 169, 169, 0.7);
    height: 70px;
    align-items: center;

    @media (max-width: 1024px) {
        height: 60px;
    }
}

.logo-container {
    height: 100%;
    display: flex;
    align-items: center;
    mix-blend-mode: multiply;
}

.logo {
    display: block;
    margin: 0;
    height: 150%; /* 高さ150%に設定 */
    max-width: 100%; /* 幅100%を設定 */
}

/* Container style */
.container {
    @apply p-6 text-center bg-gray-50;
    max-width: 100%;
}

/* 2行のテキストをスタイル化 */
header p {
    margin: 0;
}
</style>
